# 一、NIO

## 1.1 传统BIO编程

* 网络编程的基本模型是Client/Server模型，也就是两个进程之间的通信。其中服务端提供位置信息（IP和端口），客户端通过连接操作向服务端监听的地址发出请求。
* 在基于传统同步阻塞模型开发中，ServerSocket负责绑定IP地址，启动监听端口；Socket负责发起连接操作。
* 连接成功后，双方通过输入和输出流进行同步阻塞式通信。

![](Netty\传统线程模型.png)

服务端代码

``` java
public class BIOTimeServer {
    public static void main(String[] args) {
        ServerSocket serverSocket = null;
        Socket socket = null;
        try {
            serverSocket = new ServerSocket(8888);
            System.out.println("服务器已经启动在8888端口");

            while (true) {
                socket = serverSocket.accept();
                new Thread(new TimeServerHandler(socket)).start();
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                serverSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    private static class TimeServerHandler implements Runnable {
        Socket socket;

        public TimeServerHandler(Socket socket) {
            this.socket = socket;
        }

        public void run() {
            BufferedReader in = null;
            PrintWriter out = null;
            try {
                in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                out = new PrintWriter(this.socket.getOutputStream(),true);
                String currentTime = null;
                String body = null;
                while (true) {
                    body = in.readLine();
                    if (body == null)
                        break;
                    System.out.println(Thread.currentThread().getName() + "服务端接收到了请求：" + body);
                    currentTime = new Date(System.currentTimeMillis()).toString();
                    out.println(currentTime);
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                out.close();
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

客户端代码

``` java
public class BIOTimeClient {
    public static void main(String[] args) {
        Socket socket = null;
        BufferedReader in = null;
        PrintWriter out = null;
        try {
            socket = new Socket("127.0.0.1", 8888);
            in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            out = new PrintWriter(socket.getOutputStream(), true);
            out.println("客户端查询请求");
            String res = in.readLine();
            System.out.println("结果为：" + res);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                in.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            out.close();
            socket = null;
        }
    }
}
```

运行结果

![](Netty\Netty-BIO程序.png)

* BIO小结：
    * BIO的主要问题在于每当有一个新得客户端请求接入时，服务端必须创建一个新的线程处理新接入的客户端链路，一个线程只能处理一个客户端连接。在高性能服务器应用领域，往往需要面向成千上万个客户端的并发连接，这种模型无法满足高性能、高并发的接入场景。



## 1.2 伪异步模型

* 采用线程池和任务队列可以实现一种伪异步的I/O通信框架。
* 当有新的客户端接入时，将客户端的Socket封装成一个Task投递到后端的线程池中进行处理。

服务端代码

``` java
public class TimeServer {
    public static void main(String[] args) {
        ServerSocket server = null;
        try {
            server = new ServerSocket(8888);
            System.out.println("服务器在端口8888启动");
            Socket socket = null;
            // 初始化一个线程池来接收任务
            int cpu = Runtime.getRuntime().availableProcessors();
            ExecutorService pool = new ThreadPoolExecutor(
                    2,
                    cpu + 1,
                    2L,
                    TimeUnit.SECONDS,
                    new LinkedBlockingDeque<Runnable>(5),
                    Executors.defaultThreadFactory(),
                    new ThreadPoolExecutor.AbortPolicy()
            );
            while (true) {
                socket = server.accept();
                pool.execute(new TimeServerHandler(socket));
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                server.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private static class TimeServerHandler implements Runnable {
        Socket socket;

        public TimeServerHandler(Socket socket) {
            this.socket = socket;
        }

        public void run() {
            BufferedReader in = null;
            PrintWriter out = null;
            try {
                in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                out = new PrintWriter(this.socket.getOutputStream(),true);
                String currentTime = null;
                String body = null;
                while (true) {
                    body = in.readLine();
                    if (body == null)
                        break;
                    System.out.println(Thread.currentThread().getName() + "服务端接收到了请求：" + body);
                    currentTime = new Date(System.currentTimeMillis()).toString();
                    out.println(currentTime);
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                out.close();
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

}
```

* 伪异步I/O小结：
    * 当接收到新的客户端连接的时候，将请求Socket封装成一个Task，然后调用线程池的excute方法执行，从而避免了每个请求接入都创建一个新的线程
    * 由于线程池和消息队列都是有界的，因此，无论客户端并发连接数有多大，都不会导致线程个数膨胀或者内存溢出
    * 但是尽管如此，它底层通信依然采用同步阻塞模型，因此无法从根本上解决问题
    * 当客户端发送请求或者服务端应答消息缓慢，或者网络传输比较慢时，读取输入流一方的通信将被长时间阻塞。

下面介绍用NIO解决上述问题



## 1.3 NIO类库

NIO是在JDK1.4中引入的，NIO弥补了原来同步阻塞I/O的不足，它在标准Java代码中提供了高速的、面向块的I/O，以及通过以块的形式处理这些数据。



### 缓冲区Buffer

* Buffer是一个对象它包含一些要写入或者要读出的数据。
* 在NIO中，所有数据都是用缓冲区处理的。在读取数据时，直接读取到缓冲区中；在写入数据时，直接写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作
* 缓冲区实质上是一个数组，通常是一个字节数组，也可以是其他类型的数组。

（更多关于Buffer的介绍，看NIO专栏笔记）



### 通道Channel

* 网络数据通过Channel读取和写入。通道与流的不同之处在于通道是双向的，流只在一个方向上移动，而通道可以用于读、写或者同时进行



### 多路复用器Selector

* 多路复用器提供选择已经就绪的任务的能力。Selector会不断轮询注册在其上的Channel，如果某个Channel上面发生读或者写事件，这个Channel就会处于就绪状态，会被Selector轮询出来，然后通过SelectionKey获取就绪Channel的集合，进行后续的I/O操作。
* 一个多路复用器可以同时轮询多个Channel，由于JDK使用了epoll()代替传统select实现，所以没有最大句柄限制。这意味着一个线程负责Selector的轮询，就可以接入成千上万的客户端



### NIO创建的服务端示例



服务端

``` java
public class MultiplexerTimeServer implements Runnable {
    private Selector selector;
    private ServerSocketChannel serverSocketChannel;

    public MultiplexerTimeServer() throws IOException {
        // 获取一个多路复用器
        selector = Selector.open();
        // 获取一个监听TCP连接的通道
        serverSocketChannel = ServerSocketChannel.open();
        // 设置通道为非阻塞
        serverSocketChannel.configureBlocking(false);
        // 绑定端口
        serverSocketChannel.socket().bind(new InetSocketAddress(8888));
        // 注册到多路复用器上，轮询连接事件
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("服务端在8888端口启动");
    }

    @Override
    public void run() {
        while (true) {
            try {
                // 每隔1秒轮询一次
                selector.select(1000);
                // 通过迭代器遍历所有被轮询到的事件
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                SelectionKey selectionKey = null;
                while (iterator.hasNext()) {
                    selectionKey = iterator.next();
                    iterator.remove();
                    try {
                        handler(selectionKey);
                    } catch (Exception e) {
                        if (selectionKey != null) {
                            selectionKey.cancel();
                            if (selectionKey.channel() != null)
                                selectionKey.channel().close();
                        }
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

    }

    public void handler(SelectionKey key) throws Exception {
        if (key.isValid()) {
            // 如果有客户端发送TCP请求，把客户端的通道注册到多路复用器上
            if (key.isAcceptable()) {
                ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
                SocketChannel sc = ssc.accept();
                sc.configureBlocking(false);
                sc.register(selector, SelectionKey.OP_READ);
            }
            if (key.isReadable()) {
                SocketChannel sc = (SocketChannel) key.channel();
                ByteBuffer buf = ByteBuffer.allocate(1024);
                int len = sc.read(buf);
                if (len > 0) {
                    buf.flip();
                    byte[] bytes = new byte[buf.remaining()];
                    buf.get(bytes);
                    System.out.println("服务端" + new String(bytes, "UTF-8"));
                    buf.clear();
                    String date = new Date(System.currentTimeMillis()).toString();
                    buf.put(date.getBytes());
                    buf.flip();
                    sc.write(buf);
                } else if (len < 0) {
                    key.cancel();
                    sc.close();
                } else {

                }
            }
        }
    }
}
```

客户端

``` java
public class NIOTimeClient implements Runnable {

    private Selector selector;
    private SocketChannel socketChannel;
    private boolean stop;

    public static void main(String[] args) {
        NIOTimeClient client = new NIOTimeClient();
        new Thread(client).start();
    }

    public NIOTimeClient() {
        try {
            selector = Selector.open();
            socketChannel = SocketChannel.open();
            socketChannel.configureBlocking(false);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        try {
            doConnecnt();
            while (!stop) {
                selector.select(2000);
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                SelectionKey key = null;
                while (iterator.hasNext()) {
                    key = iterator.next();
                    iterator.remove();
                    handler(key);
                }
            }

        } catch (Exception e) {

        } finally {

        }

    }

    public void doConnecnt() throws Exception {
        if (socketChannel.connect(new InetSocketAddress(8888))) {
            socketChannel.register(selector, SelectionKey.OP_READ);
            response();
        } else {
            socketChannel.register(selector, SelectionKey.OP_CONNECT);
        }
    }

    public void response() throws IOException {
        ByteBuffer buf = ByteBuffer.allocate(1024);
        buf.put("客户端的回应".getBytes());
        socketChannel.write(buf);
    }

    public void handler(SelectionKey key) throws IOException {
        if (key.isValid()) {
            SocketChannel sc = (SocketChannel) key.channel();
            if (sc.finishConnect()) {
                sc.register(selector, SelectionKey.OP_READ);
                response();
            } else {
                System.exit(1);
            }

            if (key.isReadable()) {
                ByteBuffer buf = ByteBuffer.allocate(1024);
                int len = sc.read(buf);
                if (len > 0) {
                    System.out.println("服务端给的响应：" + new String(buf.array(), 0, len));
                    stop = true;
                } else if (len < 0) {
                    key.cancel();
                    sc.close();
                } else {

                }
            }
        }
    }


}
```



* NIO服务端应用小结：
    * NIO编程的难度确实比同步阻塞BIO大很多，此程序并没有考虑到“半包读写”。
    * 客户端发起的连接操作都是异步的，可以通过多路复用器注册，不需要像之前的客户端被同步阻塞
    * SocketChannel的读写操作都是异步的，如果没有可读写的数据它不会同步等待
    * 线程模型的优化，由于JDK的Selector在Linux等主流操作系统上通过epoll实现，没有连接句柄数的限制



# 二、Netty

用NIO开发出一个高质量的程序不是一件简单的事情

1. NIO类库和API繁杂，使用麻烦，需要熟练掌握Selector、ServerSocketChannel、SocketChannel、ByteBuffer
2. 需要其他的额外技能做铺垫
3. 可靠性能力补齐

Netty是最流行的NIO框架之一，它的健壮性、功能、性能、可定制性和可扩展性在同类框架中都是首屈一指的，许多商业项目也使用Netty，例如：Hadoop的RPC框架Avro



## 2.1 Netty入门

服务端

``` java
public class TimeServer {
    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap server = new ServerBootstrap();
            server.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new TimeServerHandler());
                        }
                    });
            ChannelFuture f = server.bind(8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {

        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

服务端处理器

``` java
public class TimeServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("客户端发来的消息：" + buf.toString(CharsetUtil.UTF_8));
        String date = new Date(System.currentTimeMillis()).toString();
        ctx.write(Unpooled.copiedBuffer(date, CharsetUtil.UTF_8));
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

客户端

``` java
public class TimeClient {
    public static void main(String[] args) {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new TimeClientHandler());
                        }
                    });
            ChannelFuture f = bootstrap.connect("127.0.0.1", 8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {

        } finally {
            group.shutdownGracefully();
        }
    }

}
```

客户端处理器

``` java
public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.copiedBuffer("客户端已经准备好了", CharsetUtil.UTF_8));
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        System.out.println(buf.toString(CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```



## 2.2 TCP粘包/拆包

TCP是个“流”协议，所谓流，就是没有界限的一串数据。TCP底层并不了解上层业务数据的具体含义，它会根据TCP缓冲区的实际情况进行包的划分，所以在义务上认为，一个完整的包可能会被TCP拆分成多个包进行发送，也有可能把多个小包封装成一个大的数据包进行发送，这就是所谓的TCP粘包和拆包



### 2.2.1 TCP/IP粘包/拆包问题

假设客户端分别发送了两个数据包D1和D2给服务端，由于服务端一次读取到的字节数是不确定的，故可能存在以下4种情况

1. 服务端分两次读取到了两个独立的数据包，分别是D1和D2，没有粘包和拆包
2. 服务端一次接收到了两个数据包，D1和D2粘合在一起，被称为TCP粘包
3. 服务端分两次读取到了两个数据包，第一次读取到了完整的D1包和D2包的部分内容，第二次读取到了D2包的剩余内容，这被称为TCP拆包
4. 服务端分两次读取到了两个数据包，第一次读取到了D1的部分内容，第二次读取到了D1的剩余内容和D2完整的包

如果此时服务端TCP接收滑窗非常小，而数据包D1和D2非常大，很有可能发生第5种可能，即服务端分多次才能将D1和D2包接收完全，期间发生多次拆包。



### 2.2.2 TCP粘包/拆包发生的原因

问题产生的原因有三个：

1. 应用程序write写入的字节大小大于套接口发送缓冲区大小
2. 进行MSS大小的TCP分段
3. 以太网帧的payload大于MTU进行IP分片



### 2.2.3 粘包问题的解决策略

由于底层的TCP无法理解上层的业务数据，所以在底层是无法保证数据包不被拆分和重组的，这个问题只能通过上层的应用协议栈设计来解决，根据业界的主流协议的解决方案，可以归纳如下：

1. 消息定长，例如每个报文的大小为固定长度200字节，如果不够，空格补位
2. 在包尾增加回车换行符进行分割，例如FTP协议
3. 将消息分为消息头和消息体，消息头中包含表示消息总长度（或者消息体长度）的字段，通常设计思路为消息头的第一个字段用int32来表示消息的总长度
4. 更复杂的应用层协议



## 2.3 未考虑TCP粘包导致功能异常

服务端代码

``` java
public class TimeServer {
    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap server = new ServerBootstrap();
            server.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new TimeServerHandler());
                        }
                    });
            ChannelFuture f = server.bind(8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {

        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

服务端处理器

``` java
public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    private int counter;

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        System.out.println(buf.toString(CharsetUtil.UTF_8) + "请求的次数：" + ++counter);
        String date = new Date(System.currentTimeMillis()).toString() + System.getProperty("line.separator");
        ctx.write(Unpooled.copiedBuffer(date, CharsetUtil.UTF_8));
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```



客户端代码

``` java
public class TimeClient {
    public static void main(String[] args) {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new TimeClientHandler());
                        }
                    });
            ChannelFuture f = bootstrap.connect("127.0.0.1", 8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {

        } finally {
            group.shutdownGracefully();
        }
    }

}
```

客户端处理器

``` java
public class TimeClientHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
//        ctx.writeAndFlush(Unpooled.copiedBuffer("客户端的查询请求", CharsetUtil.UTF_8));
        for (int i = 0; i <= 50; i++) {
            ByteBuf buf = Unpooled.copiedBuffer("客户端的请求" + System.getProperty("line.separator"), CharsetUtil.UTF_8);
            ctx.writeAndFlush(buf);
        }
    }


    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        System.out.println(buf.toString(CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

运行结果

```
服务端：
客户端的请求
客户端的请求
...
客户端的请求
客户端的请求
请求的次数：1

客户端：
Wed Dec 02 22:39:25 GMT+08:00 2020
```

* 小结:
    * 按照设计初衷，客户端应该收到50条当前系统时间的消息，但实际上只收到了一条。因为服务端只收到了一条请求，所以服务器只应该发送一条应答，



## 2.4 利用LineBasedFrameDecoder解决TCP粘包问题

* 为了解决TCP粘包/拆包导致的半包读写问题，Netty默认提供了多种编码器用于处理半包，这也是NIO框架和原生JDK的NIO API无法实现的



### 2.4.1 支持TCP粘包的TimeServer

服务端代码

``` java
public class TimeServer2 {
    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap server = new ServerBootstrap();
            server.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 1024)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
                            ch.pipeline().addLast(new StringDecoder());
                            ch.pipeline().addLast(new TimeServerHandler());
                        }
                    });
            System.out.println("服务器准备好了");
            ChannelFuture f = server.bind(8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

服务端处理器

``` java
public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    private int counter;

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String body = (String) msg;
        System.out.println("服务端收到了客户端的请求：" + body + " " + ++counter);
        String time = new Date(System.currentTimeMillis()).toString() + System.getProperty("line.separator");
        ByteBuf buf = Unpooled.copiedBuffer(time.getBytes());
        ctx.writeAndFlush(buf);
    }

//    @Override
//    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
//        ctx.flush();
//    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

客户端代码

``` java
public class TimeClient2 {
    public static void main(String[] args) {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
                            ch.pipeline().addLast(new StringDecoder());
                            ch.pipeline().addLast(new TimeClientHandler());
                        }
                    });
            ChannelFuture f = bootstrap.connect("127.0.0.1", 8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }
    }
}
```

客户端处理器

``` java
public class TimeClientHandler extends ChannelInboundHandlerAdapter {

    private int counter;

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
//        ctx.writeAndFlush(Unpooled.copiedBuffer("客户端的查询请求", CharsetUtil.UTF_8));
        String req = "客户端的请求" + System.getProperty("line.separator");
        for (int i = 0; i < 50; i++) {
            ByteBuf buf = Unpooled.copiedBuffer(req.getBytes());
            ctx.writeAndFlush(buf);
        }
    }


    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String body = (String) msg;
        System.out.println(body + " " + ++counter);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

服务端运行结果

```
服务器准备好了
服务端收到了客户端的请求：客户端的请求 1
服务端收到了客户端的请求：客户端的请求 2
...
服务端收到了客户端的请求：客户端的请求 47
服务端收到了客户端的请求：客户端的请求 48
服务端收到了客户端的请求：客户端的请求 49
服务端收到了客户端的请求：客户端的请求 50
```

客户端运行结果

```
Wed Dec 02 23:28:36 GMT+08:00 2020 1
Wed Dec 02 23:28:36 GMT+08:00 2020 2
...
Wed Dec 02 23:28:36 GMT+08:00 2020 47
Wed Dec 02 23:28:36 GMT+08:00 2020 48
Wed Dec 02 23:28:36 GMT+08:00 2020 49
Wed Dec 02 23:28:36 GMT+08:00 2020 50
```

* 小结：
    * 程序的运行结果完全符号预期，说明通过使用LineBasedFrameDecoder和StringDecoder成功解决了TCP粘包导致的半包读取问题。



### 2.4.2 LineBasedFrameDecoder和StringDecoder的原理分析

* LineBasedFrameDecoder的工作原理是它依次遍历ByteBuf中的可读字节，判断看是否有"\n"或者"\r\n"，如果有，就以此位置为结束位置，从可读索引到技术位置区间的字节就组成了一行。它是以回车换行符为结束标识的解码器，支持携带结束符或者不携带结束符的两种解码方式，同时支持配置单行的最大长度。如果连续读取到最大长度后仍然没有发现换行符，就会抛出异常，同时忽略掉之前读到的异常码流
* StringDecoder的功能非常简单，就是将接收到的对象转换成字符串，然后继续调用后面的Handler。
* LineBasedFrameDecoder+StringDecoder组合就是按行切换的文本解码器





# 三、分隔符和定长解码器的应用

TCP以流的方式进行传输，上层应用协议为了对消息进行区分，往往采用以下四种方式：

1. 消息长度固定
2. 将回车换行符作为消息结束符
3. 将特殊的分割符作为消息的结束标识
4. 通过在消息头中定义长度字段来标识消息的总长度

## 3.1 DelimiterBasedFrameDecoder

以分隔符作为码流结束标识的消息的解码

服务端

``` java
public class TimeServer2 {
    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap server = new ServerBootstrap();
            server.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 1024)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ByteBuf buf = Unpooled.copiedBuffer("$".getBytes());
                            ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, buf));
                            ch.pipeline().addLast(new StringDecoder());
                            ch.pipeline().addLast(new TimeServerHandler());
                        }
                    });
            System.out.println("服务器准备好了");
            ChannelFuture f = server.bind(8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

服务端处理器

``` java
public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    private int counter;

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String body = (String) msg;
        System.out.println("服务端收到了客户端的请求：" + body + " " + ++counter);
        String time = new Date(System.currentTimeMillis()).toString() + "$";
        ByteBuf buf = Unpooled.copiedBuffer(time.getBytes());
        ctx.writeAndFlush(buf);
    }

//    @Override
//    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
//        ctx.flush();
//    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

客户端

``` java
public class TimeClient2 {
    public static void main(String[] args) {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ByteBuf buf = Unpooled.copiedBuffer("$".getBytes());
                            ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, buf));
                            ch.pipeline().addLast(new StringDecoder());
                            ch.pipeline().addLast(new TimeClientHandler());
                        }
                    });
            ChannelFuture f = bootstrap.connect("127.0.0.1", 8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }
    }
}
```

客户端处理器

``` java
public class TimeClientHandler extends ChannelInboundHandlerAdapter {

    private int counter;

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
//        ctx.writeAndFlush(Unpooled.copiedBuffer("客户端的查询请求", CharsetUtil.UTF_8));
        String req = "客户端的请求" + "$";
        for (int i = 0; i < 50; i++) {
            ByteBuf buf = Unpooled.copiedBuffer(req.getBytes());
            ctx.writeAndFlush(buf);
        }
    }


    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String body = (String) msg;
        System.out.println(body + " " + ++counter);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

## 3.2 FixedLengthFrameDecoder

FixedLengthFrameDecoder是固定长度解码器，它能够按照指定的长度对消息进行自动编码，开发者不需要考虑TCP的粘包/拆包问题





# 四、Netty编解码开发



## 4.1 编解码技术

* 基于Java提供的对象输入/输出流ObjectInputStream和ObjectOutputStream，可以直接把Java对象作为可存储的字节数组写入文件，也可以传输到网络上。基于JDK默认的序列化机制可以避免操作底层的字节数组，从而提升开发效率。
* Java序列化的目的主要有两个：
    * 网络传输
    * 对象持久化
* Java序列化仅仅是Java编解码技术的一种，由于它的种种缺线，衍生出了许多编解码技术和框架



### 4.1.2 Java序列化的缺点

* Java序列化从JDK1.1版本就已经提供，它不需要添加额外的类库，只需实现Serializable并生成序列ID即可，但是在远程服务调用（RPC）中，很少使用Java序列化进行消息的编码节码和传输

Java序列化的缺点:

1. 无法跨语言

    * 无法跨语言是Java序列化最致命的问题，对于跨进程的服务调用，服务提供者可能会使用c++或者其他语言开发，当我们需要和异构语言进程交互时，Java序列化就难以胜任
    * 由于Java序列化技术是Java语言内部的私有协议，其他语言并不支持，对于用户来说它完全就是黑盒。对于Java序列化 后的字节数组，别的语言无法进行反序列化，这就严重阻碍了它的应用
    * 目前几乎所流行的RPC通信框架都没有使用Java序列化作为编解码框架，原因就是在于它无法跨语言，而这些RPC框架往往需要支持跨语言使用

2. 序列化后的码流太大

    ``` java
    public class User  implements Serializable {
        private static final long serialVersionUID = 1L;
    
        private String UserName;
        private int UserID;
    
        public String getUserName() {
            return UserName;
        }
    
        public void setUserName(String userName) {
            UserName = userName;
        }
    
        public int getUserID() {
            return UserID;
        }
    
        public void setUserID(int userID) {
            UserID = userID;
        }
    
        public byte[] codeC() {
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            byte[] value = this.UserName.getBytes();
            buffer.put(value);
            buffer.putInt(this.UserID);
            buffer.flip();
            byte[] result = new byte[buffer.remaining()];
            buffer.get(result);
            return result;
        }
    }
    ```

    ``` java
    @Test
    public void test1() throws Exception {
        User user = new User();
        user.setUserName("qiancijun");
        user.setUserID(1);
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream os = new ObjectOutputStream(bos);
        os.writeObject(user);
        os.flush();
        os.close();
        byte[] b = bos.toByteArray();
        System.out.println("JDK:" + b.length);
        bos.close();
        System.out.println("ByteBuffer:" + user.codeC().length);
    }
    ```

    ``` 
    输出结果：
    JDK:109
    ByteBuffer:13
    ```

3. 序列化性能太低

    * 与测试大小同样的方法，只需要在前后记录下时间即可，此处不在举例



## 4.2 主流的编解码框架



### 4.2.1 Google的Protobuf

* Protobuf全称Google Protocol Buffers，它由谷歌开源而来，在谷歌内部久经考验。它将数据结构以.proto文件进行描述，通过代码生成工具生成对应数据结构的POJO对象和Protobuf相关的方法和属性
* 其特点：
    1. 结构化数据存储格式（XML，JSON等）
    2. 高效的编解码性能
    3. 语言无关、平台无关、扩展性好
    4. 官方支持C++、Java和Python三种语言
* 为什么不使用XML？尽管XML的可读性和可扩展性非常好，也非常适合描述数据结构，但是XML解析的时间开销和XML为了可读性而牺牲的空间开销都非常大，因此不适合做高性能的通信协议。Protobuf使用二进制编码，在空间和性能上具有更大的优势
* Protobuf另一个比较吸引人的地方就是它的数据描述文件和代码生成机制，利用数据描述文件对数据结构进行说明的优点如下:
    1. 文本化的数据结构描述语言，可以实现语言和平台无关，特别适合异构系统间的集成
    2. 通过标识字段的顺序，可以实现协议的前向兼容
    3. 自动代码生成，不需要手工编写同样数据结构的C++和Java版本
    4. 方便后续的管理和维护，相比与代码，结构化的文档更容易管理和维护



### 4.2.2 Facebook的Thrift



### 4.2.3 JBoss Marshalling

* JBoss Marshalling是一个Java对象的序列化API包，修正了JDK自带的序列化包很多问题，但又保持跟java.io.Serializable接口的兼容。同时，增加了一些可调的参数和附加的特性，并且这些参数和特性可通过工厂类进行配置
* 相比于传统的Java序列化机制吗，它的优点如下：
    1. 可插拔的类解析器，提供更加便捷的类加载定制策略，通过一个接口即可实现定制
    2. 可插拔的对象替换技术，不需要通过继承的方式
    3. 可插拔的预定义类缓存表，可以减小序列化的字节数组长度，提升常用类型的对象序列化性能
    4. 无需实现Serializable接口，即可实现Java序列化
    5. 通过缓存技术提升对象的序列化性能



# 五、MessagePack编解码

MessagePack是一个高效的二进制序列化框架，它像JSON一样支持不同语言间的数据交换，但是它的性能更快，序列化之后的码流更小



## 5.1 MessagePack介绍

MessagePack特点：

1. 编解码高效，性能高
2. 序列化之后的码流小
3. 支持跨语言



### 5.1.1 多语言支持

衡量序列化框架通用性的一个重要指标就是对多语言的支持，因为数据交换的双方很难保证一定采用相同的开发语言，如果序列化框架和某种语言绑定，它就很难跨语言，例如Java的序列化机制。

MessagePack提供了多种语言的支持，官方支持的语言如下：Java、Python、Ruby、Haskell、C#、Lua、Go、C、C++等。





### 5.1.2 MessagePack Java API

如果使用Maven工程，在pom.xml中引入坐标

``` xml
<!-- https://mvnrepository.com/artifact/org.msgpack/msgpack -->
<dependency>
    <groupId>org.msgpack</groupId>
    <artifactId>msgpack</artifactId>
    <version>0.6.12</version>
</dependency>
```

``` java
@Test
public void test2() throws Exception {
    List<String> src = new ArrayList<>();
    src.add("aaa");
    src.add("bbb");
    src.add("ccc");
    MessagePack messagePack = new MessagePack();
    byte[] raw = messagePack.write(src);
    List<String> dest = messagePack.read(raw, Templates.tList(Templates.TString));
    System.out.println(dest.get(0));
    System.out.println(dest.get(1));
    System.out.println(dest.get(2));
}
```





## 5.2 MessagePack编码器和解码器开发

利用Netty的编解码框架可以非常方便的集成第三方序列化框架，Netty预集成了几种常用的编解码框架，用户可以根据项目的实际情况继承其他编解码框架，或者自定义



### 5.2.1 MessagePack编码器开发



msgpack编码器MsgPackEncoder

``` java
public class MsgPackEncoder extends MessageToByteEncoder<Object> {
    @Override
    protected void encode(ChannelHandlerContext ctx, Object msg, ByteBuf out) throws Exception {
        MessagePack msgpack = new MessagePack();
        byte[] raw = msgpack.write(msg);
        out.writeBytes(raw);
    }
}
```

MsgPackEncoder继承MessageToByteEncoder，它负责将Object类型的POJO对象编码为byte数组，然后写入到ByteBuf中



### 5.2.2 解码器开发

msgpack解码器MsgPackDecoder

``` java
public class MsgPackDecoder extends MessageToMessageDecoder<ByteBuf> {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf msg, List<Object> out) throws Exception {
        final byte[] array;
        final int length = msg.readableBytes();
        array = new byte[length];
        msg.getBytes(msg.readerIndex(), array, 0, length);
        MessagePack msgpack = new MessagePack();
        out.add(msgpack.read(array));
    }
}
```

首先从数据包msg中获取需要节码的byte数组，然后调用MessagePack的read方法将其反序列化为Object对象，将解码后的对象加入到解码列表out中，这样就完成了MessagePack的解码操作



### 5.2.3 功能测试

消息实体类

``` java
// 0.6.x版本需要Message注解，并且提供无参构造

@Message
public class User  implements Serializable {
    private static final long serialVersionUID = 1L;

    private String UserName;
    private int UserID;

    public User() {
    }

    public String getUserName() {
        return UserName;
    }

    public void setUserName(String userName) {
        UserName = userName;
    }

    public int getUserID() {
        return UserID;
    }

    public void setUserID(int userID) {
        UserID = userID;
    }

    public byte[] codeC() {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        byte[] value = this.UserName.getBytes();
        buffer.put(value);
        buffer.putInt(this.UserID);
        buffer.flip();
        byte[] result = new byte[buffer.remaining()];
        buffer.get(result);
        return result;
    }
}
```

服务端代码

``` java
public class MsgServer {
    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workderGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap server = new ServerBootstrap();
            server.group(bossGroup, workderGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new MsgPackDecoder());
                            ch.pipeline().addLast(new MsgPackEncoder());
                            ch.pipeline().addLast(new MsgServerHandler());
                        }
                    });
            ChannelFuture f = server.bind(8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workderGroup.shutdownGracefully();
        }
    }
}
```

服务端处理器

``` java
public class MsgServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println(msg);
        ctx.write("收到请求");
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

客户端代码

``` java
public class MsgClient {
    public static void main(String[] args) {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new MsgPackDecoder());
                            ch.pipeline().addLast(new MsgPackEncoder());
                            ch.pipeline().addLast(new MsgClientHandler(10));
                        }
                    });
            ChannelFuture f = bootstrap.connect("127.0.0.1", 8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }
    }
}
```

客户端处理器

``` java
public class MsgClientHandler extends ChannelInboundHandlerAdapter {

    private int number;

    public MsgClientHandler(int number) {
        this.number = number;
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        User user = new User();
        user.setUserID(1);
        user.setUserName("a");
        ctx.write(user);
        ctx.flush();
//        User[] us = getUsers();
//        for (User user : us) {
//            ctx.writeAndFlush(user);
//        }
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("客户端收到的消息：" + msg);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }

    private User[] getUsers() {
        User[] us  = new User[number];
        User user = null;
        for (int i = 0; i < number; i++) {
            user = new User();
            user.setUserID(i);
            user.setUserName("A" + i);
            us[i] = user;
        }
        return us;
    }
}
```

* 小结:
    * 测试代码中并没有考虑到TCP粘包和半包的处理。



## 5.3 粘包/半包支持

改动如下

``` java
public class MsgServer {
    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workderGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap server = new ServerBootstrap();
            server.group(bossGroup, workderGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new LengthFieldBasedFrameDecoder(1024, 0, 2, 0, 2));
                            ch.pipeline().addLast(new MsgPackDecoder());
                            ch.pipeline().addLast(new LengthFieldPrepender(2));
                            ch.pipeline().addLast(new MsgPackEncoder());
                            ch.pipeline().addLast(new MsgServerHandler());
                        }
                    });
            ChannelFuture f = server.bind(8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workderGroup.shutdownGracefully();
        }
    }
}
```

客户端也一样的改动

运行结果

```
服务端：
["A0",0]
["A1",1]
...
["A97",97]
["A98",98]
["A99",99]

客户端：
客户端收到的消息："收到请求"
客户端收到的消息："收到请求"
...
客户端收到的消息："收到请求"
客户端收到的消息："收到请求"
客户端收到的消息："收到请求"
```

利用Netty的半包编码和解码器LengthFieldPrepender和LengthFieldBasedFrameDecoder可以解决TCP粘包和半包问题





# 六、Google Protobuf编解码

Google的Protobuf在业界非常流行，很多商业项目选择Protobuf作为编解码框架。



## 6.1 Protobuf的入门

Protobuf是一个灵活、高效、结构化的数据序列化框架，相比于XML等传统的序列化工具，更小，更简单。Protobuf支持数据结构化一次可以到处使用，甚至跨语言使用，通过代码生成工具可以自动生成不同语言版本的源代码，甚至可以在不同版本的数据结构进程间进行数据传递，实现数据结构的前向兼容。



## 6.2 Protobuf开发环境搭建

[Protobuf Github](https://github.com/protocolbuffers/protobuf)



maven坐标

``` xml
<dependency>
  <groupId>com.google.protobuf</groupId>
  <artifactId>protobuf-java</artifactId>
  <version>3.11.0</version>
</dependency>
```



Proto文件

``` protobuf
syntax = "proto3"; // 版本

option java_outer_classname = "MyDataInfo"; // 生成的外部类名，也是文件名
option java_package = "com.qiancijun.netty.protobuf";
option optimize_for = SPEED;

message MyMessage {
  enum DataType {
    Person = 0; // 在proto3里，要从0开始编号
    Student = 1;
  }
  // 用data_type来标识传递的是哪一个枚举类型
  DataType data_type = 1;

  // 表示每次枚举类型最多只能出现其中的一个，节省空间
  oneof dataBody {
    Person person = 2;
    Student student = 3;
  }
}

message Person {
  int32 id = 1; // 表示id是第一个属性，不是默认值
  string name = 2;
}

message Student {
  int32 id = 1;
  int32 age = 2;
}
```

输入`protoc.exe --java_out=. 文件名`进行代码生成





## 6.3 客户端和服务端代码

服务端

``` java
public class ProtoBufServer {
    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap server = new ServerBootstrap();
            server.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new ProtobufDecoder(MyDataInfo.MyMessage.getDefaultInstance()));
                            ch.pipeline().addLast(new ProtoBufServerHandler());
                        }
                    });
            ChannelFuture f = server.bind(8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

服务端处理器

``` java
public class ProtoBufServerHandler extends SimpleChannelInboundHandler<MyDataInfo.MyMessage> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MyDataInfo.MyMessage msg) throws Exception {
        MyDataInfo.MyMessage.DataType dataType = msg.getDataType();
        if (dataType == MyDataInfo.MyMessage.DataType.Person) {
            MyDataInfo.Person person = msg.getPerson();
            System.out.println(person.getId() + " " + person.getName());
        } else if (dataType == MyDataInfo.MyMessage.DataType.Student) {
            MyDataInfo.Student student = msg.getStudent();
            System.out.println(student.getId() + " " + student.getAge());
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

客户端

``` java
public class ProtoBufClient {
    public static void main(String[] args) {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new ProtobufEncoder());
                            ch.pipeline().addLast(new ProtoBufClientHandler());
                        }
                    });
            ChannelFuture f = bootstrap.connect("127.0.0.1", 8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }
    }
}
```

客户端处理器

``` java
public class ProtoBufClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        int random = new Random().nextInt(3);
        MyDataInfo.MyMessage myMessage = null;
        if (0 == random) {
            myMessage = MyDataInfo.MyMessage.newBuilder().setDataType(MyDataInfo.MyMessage.DataType.Person).
                    setPerson(MyDataInfo.Person.newBuilder()
                            .setId(1)
                            .setName("Qiancijun")
                            .build())
                    .build();
        } else {
            myMessage = MyDataInfo.MyMessage.newBuilder().setDataType(MyDataInfo.MyMessage.DataType.Student).
                    setStudent(MyDataInfo.Student.newBuilder()
                            .setId(1)
                            .setAge(10)
                            .build())
                    .build();
        }
        ctx.writeAndFlush(myMessage);
    }
}
```



## 6.4 注意事项

ProtobufDecoder仅仅负责解码，它不支持读半包。因此，在ProtobufDecoder前面，一定要有能够处理读半包的解码器，有以下三种方式可以选择

1. 使用Netty提供的ProtobufVarint32FrameDecoder，它可以处理半包消息
2. 继承Netty提供的通用半包解码器LengthFieldBasedFrameDecoder
3. 继承ByteToMessageDecoder类，自己处理半包消息

如果只使用ProtobufDecoder解码器而忽略对半包消息的处理，程序是不能正常工作的。

服务端

``` java
public class ProtoBufServer {
    public static void main(String[] args) {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap server = new ServerBootstrap();
            server.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new ProtobufVarint32FrameDecoder());
                            ch.pipeline().addLast(new ProtobufDecoder(MyDataInfo.MyMessage.getDefaultInstance()));
                            ch.pipeline().addLast(new ProtobufVarint32LengthFieldPrepender());
                            ch.pipeline().addLast(new ProtoBufServerHandler());
                        }
                    });
            ChannelFuture f = server.bind(8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

客户端

``` java
public class ProtoBufClient {
    public static void main(String[] args) {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new ProtobufVarint32LengthFieldPrepender());
                            ch.pipeline().addLast(new ProtobufEncoder());
                            ch.pipeline().addLast(new ProtoBufClientHandler());
                        }
                    });
            ChannelFuture f = bootstrap.connect("127.0.0.1", 8888).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }
    }
}
```





<!-- # 七、HTTP协议开发应用

* HTTP（超文本传输协议）是建立在TCP传输协议之上的应用层协议，它的发展是万维网协会和Internet工作小组IETF合作的结果。HTTP是一个属于应用层的面向对象的协议，由于其简捷、快速的方式，适用于分布式超媒体信息系统
* 由于HTTP协议是目前Web开发的主流协议，基于HTTP的应用非常广泛，因此，掌握HTTP的开发非常重要。



## 7.1 HTTP协议

HTTP协议的主要特点：

1. 支持Client/Server模式
2. 简单——客户向服务器请求服务时，只需指定服务URL，携带必要的请求参数或者消息体
3. 灵活——HTTP允许传输任意类型的数据对象，传输的内容类型由HTTP消息头中的Content-Type加以标记
4. 无状态——HTTP协议是无状态协议，无状态是指协议对于事物处理没有记以能力，缺少状态意味着如果后续处理需要之前的信息，则它必须重传，这样可能导致每次连接传送的数据增大。另一方面，在服务器不需要先前信息时它的应答较快，负载较轻

### 7.1.1 HTTP协议的URL

HTTP URL（URL是一种特殊类型的URI，包含了用于查找某个资源的足够的信息）格式如下：
```
http://[":"port][abs_path]
```
其中，http表示要通过HTTP协议来定位网络资源；host表示合法的Internet主机域名或者IP地址；port指定一个端口号，为空则使用默认80端口；abs_path指定请求资源的URI，如果URL中没有给出abs_path，那么当它作为请求URI时，必须以"/"的形式给出

### 7.1.2 HTTP请求消息（HttpRequest）
HTTP请求由三部分组成

1. HTTP请求行
2. HTTP消息头
3. HTTP请求正文

* 请求行以一个方法符开头，以空格分开，后面跟着请求的URI和协议的版本，格式为：Method Request-URI HTTP-Version CRLF
* Method表示请求方法，Request-URI是一个统一资源标识符，HTTP-Version表示请求的HTTP版本协议，CRLF表示回车和换行（除了作为结尾的CRLF外，不允许出现单独的CR或LF字符）

## 7.2 Netty HTTP开发 -->

<!-- # 七、Netty源码分析

## 7.1 ByteBuf和相关辅助类

### 7.1.1 ByteBuf功能说明
* 当我们进行数据传输的时候，往往需要使用到缓冲区，常用的缓冲区就是JDK NIO类库提供的java.nio.Buffer。
* 实际上7中基础类型（Boolean除外）都有自己的缓冲区实现。对于NIO编程而言，主要使用的是ByteBuffer。从功能角度而言，ByteBuffer完全可以满足NIO编程的需要，但是由于NIO编程的复杂性，ByteBuffer也有其局限性，它的主要缺点如下：
    1. ByteBuffer长度固定，一旦分配完成，它的容量不能动态扩展和收缩，当需要编码的POJO对象大于ByteBuffer的容量时，会发生索引越界异常
    2. ByteBuffer只有一个标识位置的指针position，读写的时候需要手动调用`flip()`和`rewind()`，使用者必须小心谨慎的处理这些API
    3. ByteBuffer的API功能有限，一些高级和实用的特性它不支持，需要使用者自己手动实现

为了弥补这些不足，Netty提供了自己ByteBuffer实现——ByteBuf

### 7.1.2 ByteBuf的工作原理
* ByteBuf依然是一个Byte数组的缓冲区，它的基本功能应该和JDK的ByteBuffer一致，提供以下几类的基本功能：
    1. 7中Java基础类型、byte数组、ByteBuffer等的读写
    2. 缓冲区自身的copy和slice等
    3. 设置网络字节序
    4. 构造缓冲区实例
    5. 操作位置指针等方法

* ByteBuf通过两个位置指针来协调缓冲区的读写操作，读操作使用readerIndex，写操作使用writerIndex。readerIndex和writerIndex的取值一开始都是0，随着数据的写入wreterIndex会增加，读取数据会使readerIndex增加，但是它不会超过writerIndex。在读取之后0到readerIndex就被视为discard的，调用discardReadBytes方法，可以释放这部分空间，它的作用类似ByteBuffer的compact方法。readerIndex和writerIndex之间的数据是可读取的，等价于ByteBuffer position和limit之间的数据。writerIndex和capacity之间的空间是可写的，等价于ByteBuffer limit和capacity之间的可用空间。
* 由于写操作不修改readerIndex指针，读操作不修改writerIndex指针，因此读写之间不再需要调整位置指针，这极大简化了缓冲区的读写操作，避免了由于遗漏或者不熟悉`flip()`操作导致的功能异常
* 通常情况下，对ByteBuffer进行`put`操作的时候，如果缓冲区剩余可写空间不够，就会发生`BufferOverflowException`异常。为了避免发生这个问题，通常在`put`操作的时候，会对剩余可用空间进行校验，如果剩余空间不足，需要重新创建一个新的ByteBuffer，并将之前的ByteBuffer复制到新的ByteBuffer中，最后释放老的ByteBuffer


## 7.2 Channel和Unsafe

## 7.3 ChannelPipeline和ChannelHandler

## 7.4 EventLoop和EventLoopGroup

## 7.5 Future和Promise -->
