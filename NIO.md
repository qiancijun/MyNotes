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

# 缓冲区

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