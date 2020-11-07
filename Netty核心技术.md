# Netty

## 简介

1. Netty是由JBOSS提供的一个Java开源框架，现为Github上的独立项目
2. Netty是一个$\color{red}{异步}$的、$\color{red}{基于事件驱动}$的网络应用框架，用以快速开发高性能、高可靠的网络IO程序
3. Netty主要针对在TCP协议下，面向Clients端的高并发应用，或者P2P场景下的大量数据持续传输的应用
4. Netty本质是一个NIO框架，适用于服务器通讯相关的多种应用场景

Netty的底层仍然是基于TCP/IP协议，在TCP/IP协议之上包装了一层原生的JDK原生的IO，在JDK上包装了一层NIO开发，在NIO之上才是Netty框架

## 应用场景

1. 互联网：
在分布式系统中，各个节点之间需要远程服务调用，高性能的RPC框架必不可少，Netty作为异步高性能的通信框架，往往作为基础通信组件被这些RPC框架调用。典型应用：分布式服务框架RPC框架：Dubbo，Dubbo协议默认使用Netty作为基础通信组件，用于实现各进程节点之间的内部通信。

2. 大数据领域：
Hadoop的高性能通信和序列化组件（AVRO）的RPC框架，默认采用NEtty进行跨界点通信。它的Netty Service 基于 Netty框架二次封装实现

# IO

## I/O模型

1. IO模型：用什么样的通道进行数据的接收和发送，很大程度上决定了程序通信的性能
2. Java共支持3种网络编程模型I/O模式：BIO，NIO，AIO
3. Java BIO：同步并阻塞（传统阻塞型），服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情就会造成不必要的线程开销
4. Java NIO：同步非阻塞，服务器实现模式为一个线程处理多个请求（连接），即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求就进行处理
5. Java AIO：异步非阻塞，AIO引入异步通道的概念，采用了Proactor模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作系统完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连续时间长的应用

* 各模型的适用场景
    1. BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序简单易理解
    2. NIO方式适用于连接数目多且连接比较短的架构，比如聊天服务器，弹幕系统，服务器间通讯等，编程比较复杂，JDK1.4开始支持
    3. AIO方式适用于连接数目多且连接比较长的架构，比如相册服务器，充分调用OS参与并发操作，并发比较复杂，JDK7开始支持

## BIO
* Java BIO就是传统的java io编程，其相关的类和接口在`java.io`
* BIO(blocking I/O)：同步阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要开启一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，可以通过线程池机制改善（实现多客户连接服务器）
* BIO编程简单流程
  1. 服务器端启动一个`ServerSocket`
  2. 客户端启动一个`Socket`对服务器进行通信，默认情况下服务器端需要对每个客户建立一个线程与之通信
  3. 客户端发出请求后先咨询服务器是否有线程相应，如果没有则会等待，或者拒绝
  4. 如果有相应，客户端线程会等待请求结束后，才继续执行

* 应用实例:使用BIO模型编写一个服务器端，监听6666端口，当有客户端连接时，就启动一个线程与之通讯。要求使用线程池机制改善，可以连接多个客户端。服务器端可以接受客户端发送的数据（telnet 方式）。

``` java
public class BIOServer {
    public static void main(String[] args) throws Exception {
        // 创建线程池
        // 线程池不允许使用Executors去创建。
        // 而是通过ThreadPoolExecutor的方式，这样的处理方式更加明确线程池的运行规则，规避资源耗尽的风险。
        // 1.获取CPU核数
        int cpu = Runtime.getRuntime().availableProcessors();
        // 2.自定义线程池
        ExecutorService pool = new ThreadPoolExecutor(
                2,
                cpu + 1,
                2L,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<Runnable>(5),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy()
        );
        // 如果有客户端连接，就创建一个线程与之通讯

        // 1.创建一个ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);
        System.out.println("服务器已经启动");

        while (true) {
            // 监听
            final Socket s = serverSocket.accept();
            System.out.println("有客户端连接到服务器");
            // 创建线程与之通讯
            pool.execute(new Runnable() {
                @Override
                public void run() {
                    // 可以和客户端通讯
                    handler(s);
                }
            });
        }

    }

    public static void handler(Socket socket) {
        try {
            byte[] bytes = new byte[1024];
            InputStream in = socket.getInputStream();
            System.out.println(Thread.currentThread().getId());
            // 循环读取数据
            int len = 0;
            while ((len = in.read(bytes)) != -1) {
                System.out.println(new String(bytes, 0, len));
            }
        } catch (Exception e) {
            System.err.println(e);
        } finally {
            System.out.println("关闭连接");
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

* 存在的问题：
  1. 每个请求都需要创建独立的线程，与对应的客户端进行数据读取，业务处理，数据返回
  2. 当并发数较大时，需要创建大量线程来处理连接，系统资源占用较大
  3. 连接建立后，如果当前线程暂时没有数据可读，则线程就阻塞在read操作上，造成线程资源浪费

## NIO
1. Java NIO全称`java non-blocking IO`，是指JDK提供的新API。从JDK1.4开始，Java提供了一系列改进的输入/输出的新特性，被统称为NIO，是$\color{red}{同步非阻塞}$的
2. NIO相关类都被放在`java.nio`包及子包下，并且对原`java.io`包中的很多类进行重写
3. NIO有三大核心部分：Channel（通道），Buffer（缓冲区），Selector（选择器）
4. NIO是面向$\color{red}{缓冲区，或者面向块}$编程的。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞式的高伸缩性网络
5. Java NIO的非阻塞模式：使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直到数据变得可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。
6. NIO是可以做到用一个线程来处理多个操作的。假设有10000个请求过来，根据实际情况，可以分配50个或者100个线程来处理。不像之前的阻塞IO一样，要分匹配10000个。
7. HTTP2.0使用了多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比HTTP1.1大了好几个数量级。

* 举例说明Buffer的使用
``` java
public class BasicBuffer {
    public static void main(String[] args) {
        // 创建一个Buffer，大小为5，即可以存放5个int
        IntBuffer intBuffer = IntBuffer.allocate(5);
        // 向Buffer中存放数据
        for (int i = 1; i <= intBuffer.capacity(); i++) {
            intBuffer.put(i);
        }
        // 从Buffer读取数据
        // 将buffer读写切换：intBuffer.flip();
        intBuffer.flip();
        while (intBuffer.hasRemaining()) {
            System.out.println(intBuffer.get());
        }
    }
}
```

### NIO和BIO的比较
1. BIO以流的方式处理数据，而NIO以块的方式处理数据，块 I/O的效率比 流 I/O高很多
2. BIO是阻塞的，NIO是非阻塞的
3. BIO基于字节流和字符流进行操作，而NIO基于Channel（通道）和Buffer（缓冲区）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中，Selector（选择器）用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用单个线程就可以监听多个客户端通道

### NIO三大核心组件

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/Netty/Netty-NIO%E4%B8%89%E5%A4%A7%E7%BB%84%E4%BB%B6%E5%85%B3%E7%B3%BB%E5%9B%BE.png)

* 关系图的说明：
  1. 每一个Channel都对应一个Buffer
  2. Selector会对应一个线程，一个线程对应多个Channel
  3. 该图反应了有三个Channel注册到了Selector
  4. 程序切换到哪个Channel是由事件决定的，Event是一个重要的概念
  5. Selector会根据不同的事件，在各个通道上切换
  6. Buffer就是一个内存块，底层是一个数组
  7. 数据的读取写入是通过Buffer，这一点和BIO有本质的区别，BIO是输入流或者是输出流，不能双向，但是NIO的Buffer既可以读也可以写，需要使用`filp()`方法进行切换
  8. Channel是双向的，可以返回底层操作系统的情况，比如Linux，底层的操作系统通道就是双向的

### Buffer
* 缓冲区（Buffer）：缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个$\color{red}{容器对象（含数组）}$，该对象提供了$\color{red}{一组方法}$，可以更轻松的使用内存块，缓冲区对象内置了一些机制，能够追踪和记录缓冲区的状态变化情况。Channel提供从文件、网络读取数据的渠道，但是读取或写入数据都必须经由Buffer
* Buffer类及其子类
  1. 在NIO中，Buffer是一个顶级父类，它是一个抽象类，类的层级关系图
  2. Buffer类定义了所有的缓冲区都具有的四个属性来提供关于其所包含的数据元素的信息：
     1. Capacity：容量，即可以容纳的最大数据量；在缓冲区创建时被设定并且不能改变
     2. Limit：表示缓冲区的当前终点，不能对缓冲区超过极限的位置进行读写操作。且极限是可以修改的
     3. Position：位置，下一个要被读或写的元素的索引，每次读写缓冲区数据时都会改变值，为下次读写做准备
     4. Mark：标记


![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/Netty/Netty-NIO%20Buffer%E7%B1%BB%E5%B1%82%E7%BA%A7%E5%85%B3%E7%B3%BB%E5%9B%BE.png)
