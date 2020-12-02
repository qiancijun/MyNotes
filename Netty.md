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