#  NIO

Java NIO（New IO）是从Java 1.4版本开始引入的一个新的IO API，可以替代标准的Java IO API。NIO与原来的IO有同样的作用和目的，但是使用的方式完全不同，NIO支持面向缓冲区的、基于通道的IO操作。NIO将以更加高效的方式进行文件的读写操作。

* NIO与BIO的区别

|IO|NIO|
|:-|:-|
|面向流(Stream Oriented)|面向缓冲区(Buffer Oriented)|
|阻塞IO(Blocking IO)|非阻塞IO(Non Blocking IO)|

## 通道
Java NIO系统的核心在于：通道(Channel)和缓冲区(Buffer)。通道表示打开到 IO 设备(例如：文件、套接字)的连接。若需要使用 NIO 系统，需要获取用于连接 IO 设备的通道以及用于容纳数据的缓冲区。然后操作缓冲区，对数据进行处理。

>简而言之，Channel 负责传输， Buffer 负责存储

## 缓冲区

* 缓冲区：一个用于特定基本数据类型的容器。由 java.nio 包定义的，所有缓冲区都是 Buffer 抽象类的子类。Java NIO 中的 Buffer 主要用于与 NIO 通道进行交互，数据是从通道读入缓冲区，从缓冲区写入通道中的。

* Buffer 就像一个数组，可以保存多个相同类型的数据。根据数据类型不同(boolean 除外) ，有以下 Buffer 常用子类：
    * ByteBuffer
    * CharBuffer
    * ShortBuffer
    * IntBuffer
    * LongBuffer
    * FloatBuffer
    * DoubleBuffer
* 上述 Buffer 类 他们都采用相似的方法进行管理数据，只是各自管理的数据类型不同而已。都是通过如下方法获取一个 Buffer对象：`static XxxBuffer allocate(int capacity)` : 创建一个容量为 capacity 的 XxxBuffer 对象


# 缓冲区

* Buffer 中的重要概念：
    * 容量 (capacity) ：表示 Buffer 最大数据容量，缓冲区容量不能为负，并且创建后不能更改。
    * 限制 (limit)：第一个不应该读取或写入的数据的索引，即位于 limit 后的数据不可读写。缓冲区的限制不能为负，并且不能大于其容量。
    * 位置 (position)：下一个要读取或写入的数据的索引。缓冲区的位置不能为负，并且不能大于其限制
    * 标记 (mark)与重置 (reset)：标记是一个索引，通过 Buffer 中的 mark() 方法指定 Buffer 中一个特定的 position，之后可以通过调用 reset() 方法恢复到这个 position.

* Buffer 所有子类提供了两个用于数据操作的方法：get()与 put() 方法

* 获取 Buffer 中的数据
    * get() ：读取单个字节
    * get(byte[] dst)：批量读取多个字节到 dst 中
    * get(int index)：读取指定索引位置的字节(不会移动 position)

* 放入数据到 Buffer 中
    * put(byte b)：将给定单个字节写入缓冲区的当前位置
    * put(byte[] src)：将 src 中的字节写入缓冲区的当前位置
    * put(int index, byte b)：将指定字节写入缓冲区的索引位置(不会移动 position)


## 直接缓冲区与非直接缓冲区
* 非直接缓冲区：通过`allocate()`方法分配缓冲区，将缓冲区建立在JVM的内存中
* 直接缓冲区：通过`allocateDirect()`方法分配直接缓冲区，将缓冲区建立在操作系统的物理内存中。可以提高效率


# 通道
用于源节点与目标节点的连接。在Java NIO中负责缓冲区中数据的传输。Channel本身不存储数据，因此需要配合缓冲区进行数据传输

* 通道的主要实现类(java.nio.channels.Channel)
  1. FileChannel
  2. SocketChannel
  3. ServerSocketChannel
  4. DatagramChannel

* 获取通道
  1. Java针对支持通道的类提供了`getchannel()`方法
    * FileInputStream/FileOutputStream（本地IO）
    * RandomAccessFile（本地IO）
    * Socket（网络IO）
    * ServerSocket（网络IO）
    * DatagramSocket（网络IO）
  2. 在JDK7中的NIO2中，针对各个通道提供了静态方法`open()`
  3. 在JDK7中的NIO2中的Files工具类的`newByteChannel()`

* 利用通道完成文件的复制（非直接缓冲区）
``` java
public class TestChannel {
    public static void main(String[] args) throws Exception {
        FileInputStream fis = new FileInputStream("C:\\WorkSpace\\Learn\\NIO\\src\\main\\resources\\test1.txt");
        FileOutputStream fos = new FileOutputStream("C:\\WorkSpace\\Learn\\NIO\\src\\main\\resources\\test2.txt");
        // 获取通道
        FileChannel inChannel = fis.getChannel();
        FileChannel outChannel = fos.getChannel();

        // 分配一个指定大小的额缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);

        // 将通道中的数据存入缓冲区中
        while (inChannel.read(buf) != -1) {
            buf.flip();
            // 将缓冲区中的数据写入通道
            outChannel.write(buf);
            buf.clear(); // 清空缓冲区
        }
        inChannel.close();
        outChannel.close();
        fos.close();
        fis.close();
    }
}
```

* 使用直接缓冲区完成文件的复制（内存映射文件）
``` java
public class TestChannel2 {
    public static void main(String[] args) throws Exception {
        FileChannel inChannel = FileChannel.open(Paths.get(
                "C:\\WorkSpace\\Learn\\NIO\\src\\main\\resources\\test1.txt"),
                StandardOpenOption.READ);
        FileChannel outChannel = FileChannel.open(Paths.get(
                "C:\\WorkSpace\\Learn\\NIO\\src\\main\\resources\\test2.txt"),
                StandardOpenOption.CREATE_NEW,
                StandardOpenOption.READ,
                StandardOpenOption.WRITE);
        // 内存映射文件
        MappedByteBuffer inMapBuf = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, inChannel.size());
        MappedByteBuffer outMapBuf = outChannel.map(FileChannel.MapMode.READ_WRITE, 0, inChannel.size());
        // 直接对缓冲区进行数据的读写操作
        byte[] bytes = new byte[inMapBuf.limit()];
        inMapBuf.get(bytes);
        outMapBuf.put(bytes);
        inChannel.close();
        outChannel.close();
    }
}
```

* 通道之间的传输（直接缓冲区）
``` java
public class TestChannel3 {
    public static void main(String[] args) throws Exception {
        FileChannel inChannel = FileChannel.open(Paths.get(
                "C:\\WorkSpace\\Learn\\NIO\\src\\main\\resources\\test1.txt"),
                StandardOpenOption.READ);
        FileChannel outChannel = FileChannel.open(Paths.get(
                "C:\\WorkSpace\\Learn\\NIO\\src\\main\\resources\\test2.txt"),
                StandardOpenOption.CREATE_NEW,
                StandardOpenOption.READ,
                StandardOpenOption.WRITE);
        inChannel.transferTo(0, inChannel.size(), outChannel);
        // outChannel.transferFrom(inChannel, 0, inChannel.size());
        inChannel.close();
        outChannel.close();
    }
}
```

## 分散读取和聚焦写入
* 分散读取（Scattering Reads）：将通道中的数据分散到多个缓冲区中
> 按照缓冲区的顺序，从Channel中读取的数据依次将Buffer填满

``` java
public void test1() throws Exception {
    RandomAccessFile raf1 = new RandomAccessFile("C:\\WorkSpace\\Learn\\NIO\\src\\main\\resources\\test1.txt", "rw");
    // 1. 获取通道
    FileChannel fileChannel = raf1.getChannel();

    // 2. 分配指定大小的缓冲区
    ByteBuffer buf1 = ByteBuffer.allocate(100);
    ByteBuffer buf2 = ByteBuffer.allocate(1024);
    ByteBuffer buf3 = ByteBuffer.allocate(100);
    // 3. 分散读取
    ByteBuffer[] bufs = {buf1, buf2, buf3};
    fileChannel.read(bufs);
    // 4.
    for (ByteBuffer bf : bufs) {
        bf.flip();
    }
    System.out.println(new String(bufs[0].array(), 0, bufs[0].limit()));
    System.out.println(new String(bufs[1].array(), 0, bufs[1].limit()));
    System.out.println(new String(bufs[2].array(), 0, bufs[2].limit()));
}
```

* 聚集写入（Gathering Writes）：将多个缓冲区中的数据聚集到通道中
> 按照缓冲区的顺序，写入position和limit之间的数据到Channel

``` java
RandomAccessFile raf2 = new RandomAccessFile("C:\\WorkSpace\\Learn\\NIO\\src\\main\\resources\\test2.txt", "rw");
FileChannel raf2Channel = raf2.getChannel();
raf2Channel.write(bufs);
```

## 字符集
* 编码：字符串 -> 字节数组
* 解码：字节数组 -> 字符串

查看所有的支持的字符集
``` java
@Test
public void test2() {
    SortedMap<String, Charset> map = Charset.availableCharsets();
    Set<Map.Entry<String, Charset>> set = map.entrySet();
    for (Map.Entry<String, Charset> s : set) {
        System.out.println(s.getKey() + "=" + s.getValue());
    }
}
```

编码与解码：
``` java
@Test
public void test3() throws Exception {
    Charset charset = Charset.forName("GBK");
    // 获取编码器与解码器
    CharsetEncoder encoder = charset.newEncoder();
    CharsetDecoder decoder = charset.newDecoder();

    CharBuffer charBuffer = CharBuffer.allocate(1024);
    charBuffer.put("千慈菌");
    charBuffer.flip();

    // 编码
    ByteBuffer buffer = encoder.encode(charBuffer);
    for (int i = 0; i < 6; i++) {
        System.out.println(buffer.get());
    }

    // 解码
    buffer.flip();
    CharBuffer ans = decoder.decode(buffer);
    System.out.println(ans.toString());
}
```
> Charset提供了decoder方法，可以不用获取解码器与编码器

# 阻塞与非阻塞
* 传统的 IO 流都是阻塞式的。也就是说，当一个线程调用 read() 或 write()时，该线程被阻塞，直到有一些数据被读取或写入，该线程在此期间不能执行其他任务。因此，在完成网络通信进行 IO 操作时，由于线程会阻塞，所以服务器端必须为每个客户端都提供一个独立的线程进行处理，当服务器端需要处理大量客户端时，性能急剧下降。
* Java NIO 是非阻塞模式的。当线程从某通道进行读写数据时，若没有数据可用时，该线程可以进行其他任务。线程通常将非阻塞 IO 的空闲时间用于在其他通道上执行 IO 操作，所以单独的线程可以管理多个输入和输出通道。因此，NIO 可以让服务器端使用一个或有限几个线程来同时处理连接到服务器端的所有客户端。

## 选择器
* 选择器（Selector） 是 SelectableChannle 对象的多路复用器，Selector 可以同时监控多个 SelectableChannel 的 IO 状况，也就是说，利用 Selector可使一个单独的线程管理多个 Channel。Selector 是非阻塞 IO 的核心。

## NIO网络通信
* 使用 NIO 完成网络通信的三个核心：
    1. 通道（Channel）：负责连接
    2. 缓冲区（Buffer）：负责数据的存取
    3. 选择器（Selector）：是SelectableChannel的多路复用器，用于监控SelectableChannel的IO状况