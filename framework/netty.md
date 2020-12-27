## Java I/O模型

**`BIO`：**同步阻塞。Java传统的`IO`编程方式。服务器实现为一个连接一个线程，通过线程池改善资源浪费的问题，适用于连接数目较少的架构

![image-20201119201151385](https://gitee.com/p8t/picbed/raw/master/imgs/20201119201153.png)

`BIO`服务端

```java
// 通过telnet ip port可以直接访问
public class BIOServer {
    public static void main(String[] args) throws IOException {
        ExecutorService threadPool = Executors.newFixedThreadPool(5);
        ServerSocket serverSocket = new ServerSocket(9999);
        while (true) {
            Socket socket = serverSocket.accept();
            threadPool.execute(() -> {
                try {
                    InputStream is = socket.getInputStream();
                    byte[] buf = new byte[1024];
                    int len;
                    // read()方法会阻塞
                    while ((len = is.read(buf)) != -1) {
                        System.out.println(new String(buf, 0, len));
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    try {
                        socket.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
    }
}
```

**`NIO`：**非阻塞，多路复用。一个线程管理多个客户端，能这样管理的理由是一个线程并不总是在工作，它会有等待磁盘`IO`的时间或等待客户端连接的时间，利用这个时间可以执行其它客户端的任务

![image-20201119201212943](https://gitee.com/p8t/picbed/raw/master/imgs/20201119201214.png)

**`AIO`：**异步非阻塞。还没广泛应用

## NIO

三大核心组件

- `Channel`
- `Buffer`
- `Selector`

![image-20201119211429977](https://gitee.com/p8t/picbed/raw/master/imgs/20201119211431.png)

1、每个`channel`都会对应一个`buffer`，一个线程对应一个`selector`，一个`selector`对应多个`channel`，图中有3个channel注册到`selector`。

2、线程执行哪个`channel`是由事件（`event`）决定的，selector会根据不同的事件，在各个通道上切换

3、buffer就是一个内存块，底层是一个数组

4、数据的读取和写入都要通过`buffer`。`BIO`中流是单向的，而`NIO`是双向的，可读可写

5、`channel`是读写非阻塞的

### Channel

`FileChannel`：用于文件

`ServerSocketChannel / SocketChannel`：用于`TCP`

`DatagramChannel`：用于`UDP`

#### 拷贝文件

![image-20201119221736182](https://gitee.com/p8t/picbed/raw/master/imgs/20201119221737.png)

```java
public void copy() throws IOException {
    FileChannel inChannel = new FileInputStream("1.jpg").getChannel();
    FileChannel outChannel = new FileOutputStream("2.jpg").getChannel();
    ByteBuffer buf = ByteBuffer.allocate(1024);
    while (inChannel.read(buf) != -1) {
        buf.flip(); // 切换读模式
        outChannel.write(buf);
        buf.clear(); // 重置标志位
    }
    inChannel.close();
    outChannel.close();
}

public void copy() throws IOException {
    FileChannel inChannel = new FileInputStream("1.jpg").getChannel();
    FileChannel outChannel = new FileOutputStream("2.jpg").getChannel();
    outChannel.transferFrom(inChannel, 0, inChannel.size());
    inChannel.close();
    outChannel.close();
}
```

### Scatter / Gather

![image-20201119225011653](https://gitee.com/p8t/picbed/raw/master/imgs/20201119225012.png)

### Selector

传统网络IO是客户端与线程一对一的关系，一个客户端阻塞，那么服务它的线程也会阻塞，造成资源浪费。

`NIO`是事件驱动的，关键实现就是`Selector`。线程不再独立服务客户端，而是每个客户端会有一个注册在`Selector`上的`channel`，`Selector`线程会轮询`channel`，找出有事件发生的`channel`进行对应处理。

事件有以下4种

- `OP_READ`
- `OP_WRITE`
- `OP_CONNECT`
- `OP_ACCEPT`

## Reactor模型

很多并发请求服务都应用了`Reactor`模型，比如`netty，redis，nginx`

### 单Reactor单线程

![image-20201120180602597](https://gitee.com/p8t/picbed/raw/master/imgs/20201120180604.png)

### 单Reactor多线程

一般来说处理请求是最耗时的一步，因此在单`Reactor`多线程中，Reactor线程只负责建立连接不处理请求，**把请求直接交给后面的线程池处理**

![image-20201120180640473](https://gitee.com/p8t/picbed/raw/master/imgs/20201120180641.png)

### 主从Reactor多线程

![image-20201120180650843](https://gitee.com/p8t/picbed/raw/master/imgs/20201120180652.png)

## Netty模型

`Reactor`模型属于理论模型，下面是`Netty`落地实现

1、`Netty`抽象出两组线程池，分别为`BossGroup`和`WorkerGroup`。`BossGroup`专门负责建立网络连接，`WorkerGroup`专门负责网络读写

2、`BossGroup`和`WorkerGroup`类型都是`NioEventLoopGroup`

3、`NioEventLoopGroup`相当于一个事件循环组，这个组中含有多个事件循环，每一个事件循环是一个`NioEventLoop`

4、`NioEventLoop`表示一个不断循环的执行任务的线程，每个`NioEventLoop`都有一个`Selector`，用于监听绑定在它上面的网络通讯

5、`BossGroup`的`NioEventLoop`执行步骤为：

- 轮询`accept`事件
- 处理`accept`事件，与`client`建立连接，生成`NioSocketChannel`，并将其注册到`WorkerGroup`的某个`NioEventLoop`的`Selector`上
- 处理任务队列的其它任务，`runAllTasks`

6、`WorkerGroup`的`NioEventLoop`执行步骤为：

- 轮询`read / write`事件
- 处理`IO`事件，在对应的`NioSocketChannel`中处理
- 处理任务队列的其它任务，`runAllTasks`

7、`WorkerGroup`的`NioEventLoop`处理业务时，会使用`pipeline`。`pipeline`中包含了`channel`，可以通过`pipeline`获取对应的`channel`

## 第一个Netty程序

```java
// 服务端
public class NettyServer {
    public static void main(String[] args) throws InterruptedException {

        // NioEventLoopGroup线程数默认为 CPU核心数 * 2
        // BossGroup   : 主Reactor, 用于建立连接
        // WorkerGroup : 从Reactor, 用于处理逻辑
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup(4);

        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workerGroup)
                    // 服务器channel实现类
                    .channel(NioServerSocketChannel.class)
                    // 通过ChannelInitializer把一个个ChannelHandler注册到ChannelPipeline中
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        // 给pipeline设置处理器
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new NettyServerHandler());
                        }
                    });
            // 绑定端口, 启动服务器
            // sync()会阻塞, 直到绑定完成
            ChannelFuture cf = bootstrap.bind(9999).sync();
            // 阻塞, 直到服务器的Channel关闭
            cf.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}

public class NettyServerHandler extends ChannelInboundHandlerAdapter {

    // 2. 收到客户端的消息
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buffer = (ByteBuf) msg;
        System.out.println("from client: " + buffer.toString(CharsetUtil.UTF_8));
        System.out.println("client address: " + ctx.channel().remoteAddress());
        // 给客户端回个消息
        ctx.writeAndFlush(Unpooled.copiedBuffer("Hello Client", CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}

// 客户端
public class NettyClient {
    public static void main(String[] args) throws InterruptedException {
        // 客户端只需要一个EventLoopGroup
        NioEventLoopGroup clientGroup = new NioEventLoopGroup();

        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(clientGroup)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new NettyClientHandler());
                        }
                    });
            // 连接服务器
            ChannelFuture cf = bootstrap.connect("127.0.0.1", 9999).sync();
            cf.channel().closeFuture().sync();
        } finally {
            clientGroup.shutdownGracefully();
        }
    }
}

public class NettyClientHandler extends ChannelInboundHandlerAdapter {
	// 1. 建立连接后通道可用，就向服务器发送一条消息
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.copiedBuffer("Hello Server", CharsetUtil.UTF_8));
    }
	
    // 3. 收到服务器的消息
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buffer = (ByteBuf) msg;
        System.out.println("from server: " + buffer.toString(CharsetUtil.UTF_8));
        System.out.println("server address: " + ctx.channel().remoteAddress());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

## Netty重要组件

> 原文链接：[Netty系列五、Netty中的核心组件关系分析 ](https://juejin.cn/post/6872681547179229197)

### 关系总览

![image-20201122173154407](https://gitee.com/p8t/picbed/raw/master/imgs/20201122173156.png)

### ChannelInitializer

通过继承`ChannelInitializer`这个类来将`ChannelHandler`注册到`ChannelPipeline`中

### Channel

`Channel`接口定义了一组和`ChannelInboundHandlerAPI`相关的状态模型，其生命周期如下

![image-20201122175330233](https://gitee.com/p8t/picbed/raw/master/imgs/20201122175331.png)

1、当有客户端连接进来时，会创建通道并注册到`EventLoop`，对应`channelRegistered`状态

2、成功连接后，通道处于可用状态，对应`channelActive`状态

3、断开连接后，通道处于不可用状态，对应`channelInactive`状态

4、通道从`EventLoop`中移除，对应`channelUnregistered`状态

注：一个`channel`对应着一个客户端，因此在客户端的一次连接与断开过程中，上述每个状态只会触发一次。在建立连接时触发`channelRegistered`和`channelActive`；在断开连接时触发`channelInactive`和`channelUnregistered`

代码说明，只展示服务端自定义`Handler`的部分（这些方法并不是来自`Channel`，但从语义上来说我认为是一样的）

```java
public class NettyServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        System.out.println("1. channel registered");
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("2. channel active");
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("3. channel inactive");
    }

    @Override
    public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
        System.out.println("4. channel unregistered");
    }
}
/*
 * 当客户端连接服务端时, 服务端输出
 * 1. channel registered
 * 2. channel active
 * 当客户端断开连接时, 服务端输出
 * 3. channel inactive
 * 4. channel unregistered
 */
```

### ChannelHandler

#### 生命周期

| 类型              | 描述                                            |
| ----------------- | ----------------------------------------------- |
| `handlerAdded`    | 把`ChannelHandler`添加到`ChannelPipeline`时调用 |
| `handlerRemoved`  | 把`ChannelHandler`从`ChannelPipeline`移除时调用 |
| `exceptionCaught` | 在`ChannelPipeline`里发生异常时调用             |

`handlerAdded`和`handlerRemoved`分别在建立连接和断开连接时调用

注：一个`Channel`会有多个`ChannelHandler`，因此在一个服务器与客户端的连接中，每一个`ChannelHandler`都会触发它自己的生命周期方法

代码说明，就比上面多了两个方法

```java
public class NettyServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        System.out.println("1. handler added ");
    }

    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        System.out.println("2. channel registered");
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("3. channel active");
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("4. channel inactive");
    }

    @Override
    public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
        System.out.println("5. channel unregistered");
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        System.out.println("6. handler removed");
    }
}
/* 
 * 建立连接
 * 1. handler added
 * 2. channel registered
 * 3. channel active
 * 断开连接
 * 4. channel inactive
 * 5. channel unregistered
 * 6. handler removed
 */
```

### ChannelInboundHandler

除去与`Channel`生命周期相关的方法外，剩余事件如下

#### channelRead

`channel`上有数据到来时触发，由于一次发送的数据可能超出`ByteBuf`的容量，因此可能会多次调用这个方法

发送的数据必须指定结束标记, 否则只会触发`channelReadComplete`

#### channelReadComplete

一次传输的数据读完后调用，也就是每一次数据传输调用完`channelRead`后调用，分两种情况：
1. 本次`read`的长度小于`ByteBuf`的长度，说明数据已经发送完了
2. 本次`read`的长度等于0，这是因为如果一个`msg`长度等于`ByteBuf`长度的整数倍，那么就无法通过第一种情况判断数据是否读取完了，只能多读一次发现读到空来确定本次读完了

二者配合使用例子如下

> 来源：[channelRead()和channelReadComplete() 方法的区别是什么？](https://segmentfault.com/q/1010000018753423)

首先看下面这段代码，这个例子是`Netty in action`里的第二章里的例子，这个是Server的回调方法。

`channelRead`表示接收消息，可以看到`msg`转换成了`ByteBuf`，然后打印，也就是把Client传过来的消息打印了一下，你会发现每次打印完后，`channelReadComplete`也会调用，如果你试着传一个超长的字符串过来，超过1024个字母长度，你会发现`channelRead`会调用多次，而`channelReadComplete`只调用一次。

所以这就比较清晰了吧，因为`ByteBuf`是有长度限制的，所以超长了，就会多次读取，也就是调用多次`channelRead`，而`channelReadComplete`则是每条消息只会调用一次，无论你多长，分多少次读取，只在该条消息最后一次读取完成的时候调用，所以这段代码把关闭`Channel`的操作放在`channelReadComplete`里，放到`channelRead`里可能消息太长了，结果第一次读完就关掉连接了，后面的消息全丢了。

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ByteBuf in = (ByteBuf) msg;
    //将消息记录到控制台
    System.out.println( "Server received: " + in.toString(CharsetUtil.UTF_8) + "，size is " + in.toString(CharsetUtil.UTF_8).length());
    ctx.write(in);
}

@Override
public void channelReadComplete(ChannelHandlerContext ctx) {
    System.out.println("channelReadComplete");
    ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
}
```

#### userEventTriggered

搭配`IdleStateHandler()`使用，判断连接是否有事件发生，用于心跳机制

#### channelWritabilityChanged

// TODO

### ChannelOutboundHandler







`ChannelHandler`是一个接口, 联想我们使用`Netty`的步骤, 会利用各种`ChannelHandler`来对数据进行处理, 对数据流的编码解码都是基于`ChannelHandler`来完成的, 站在服务器的角度上, 各个不同类型的`ChannelHandler`构成了服务器处理客户端消息的链条, 数据在`ChannelHandler-A`中处理完后传到`ChannelHandler-B`, 最后再回传回来,`ChannelHandler`中定义了一个个的接口方法, 这些接口方法是整个数据处理流程生命周期的回调, 在不同的生命周期时间点触发不同的回调方法

ChannelHandler通常会分为三类, 分别是入站处理器、出站处理器以及入站出站处理器, 入站处理器仅仅会数据开始处理到处理完毕起作用, 通常用于对数据的解码, 以及业务操作等, 出站处理器作用于数据处理完毕后回传给客户端之前起作用, 一般用于对数据进行编码操作, 而入站出站处理器则是整个生命周期均起作用

相信大家如果使用过Netty的话对这些应该很熟悉了, 我们假设有入站出站处理器以下几个:InboundHandler_1 -> OutboundHandler_1 -> InbondHandler_2 -> OutboundHandler_2


他们以链表的形式存放到了一起, 假设目前的有数据从客户端发送过来, 那么会经过InboundHandler_1将完成的字节数据组织到一起, 其实就是解决TCP中的粘包半包问题, 确保我们能够拿到客户端发送的一条完整消息, 然后经过InbondHandler_2对数据进行处理, 在该处理器中完成业务操作, 比如输出这条消息, 然后将消息通过
writeAndFlush回传给客户端, 当我们触发了该writeAndFlush方法后, 就会同时触发OutboundHandler的处理流
程, 先经过OutboundHandler_2往数据中增加一些信息, 比如数据的前缀等等(根据业务需求), 然后通过
OutboundHandler_1对数据进行编码处理, 这样客户端就能同样通过相同的流程获取到服务器发送的一条完整消息,这就是ChannelHandler的大致处理流程, 相信这些大家都应该很熟悉了, 在此进行简单的回顾

可以看到, 服务器对数据的处理, 一定会先通过InbondHandler, 再通过OutbondHandler, 上面的链表组织形式中
InboundHandler_1处理完毕后会跳过OutboundHandler_1, 进而利用InbondHandler_2对数据继续进行处理, 其实在整个处理的流程中, Netty会遍历这个链表中所有的ChannelHandler, 通过ChannelHandler instanceof ChannelInboundHandler 或者 ChannelOutboundHandler来判断是何种类型的处
理器, 所以判断条件是很简单的

## Netty实现群聊

```java
public class ChatServer {
    public static void main(String[] args) throws InterruptedException {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup(4);
        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            pipeline.addLast(new DelimiterBasedFrameDecoder(4096, Delimiters.lineDelimiter()));
                            pipeline.addLast(new StringDecoder(CharsetUtil.UTF_8));
                            pipeline.addLast(new StringEncoder(CharsetUtil.UTF_8));
                            pipeline.addLast(new ChatServerHandler());
                        }
                    });
            ChannelFuture cf = bootstrap.bind(9999).sync();
            cf.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}

// 泛型就是发送的数据类型
public class ChatServerHandler extends SimpleChannelInboundHandler<String> {
    // 保存已经建立连接的channel
    private final static ChannelGroup channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);

    /*
     * 有连接建立时调用
     */
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        // 1. 获取连接的对象
        Channel channel = ctx.channel();
        // for (Channel ch : channelGroup) {
        //     ch.writeAndFlush("[" + channel.remoteAddress() + "] join chat\n");
        // }
        // 2. 提醒其它用户有人上线了, 这是Netty对上面的封装, 不需要我们遍历写数据
        channelGroup.writeAndFlush("[" + channel.remoteAddress() + "] join chat\n");
        // 3. 把当前用户添加到channelGroup
        channelGroup.add(channel);
    }

    /*
     * 有连接断开时调用
     */
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        // 获取连接的对象
        Channel channel = ctx.channel();

        // 把当前用户移除channelGroup
        // 实际上不写也行, netty会自动移除断开的客户端
        // 有个问题: 自己定义的成员变量, netty怎么做到自动调用的
        // channelGroup.remove(channel);

        channelGroup.writeAndFlush("[" + channel.remoteAddress() + "] leave chat\n");
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        System.out.println("[" + channel.remoteAddress() + "] online\n");
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        System.out.println("[" + channel.remoteAddress() + "] outline\n");
    }

    /*
     * 服务端收到客户端发送消息的时候调用
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        Channel channel = ctx.channel();
        channelGroup.forEach(ch -> {
            if (ch != channel) {
                ch.writeAndFlush("[" + channel.remoteAddress() + "] send msg : " + msg + "\n");
            } else {
                ch.writeAndFlush("[me] : " + msg + "\n");
            }
        });
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}

public class ChatClient {
    public static void main(String[] args) throws InterruptedException {
        NioEventLoopGroup eventLoopGroup = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(eventLoopGroup)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline()
                                    .addLast(new DelimiterBasedFrameDecoder(4096, Delimiters.lineDelimiter()))
                                    .addLast(new StringDecoder(CharsetUtil.UTF_8))
                                    .addLast(new StringEncoder(CharsetUtil.UTF_8))
                                    .addLast(new ChatClientHandler());
                        }
                    });
            ChannelFuture cf = bootstrap.connect("127.0.0.1", 9999).sync();
            
            // 起个线程用于读取用户输入模拟聊天
            new Thread(() -> {
                BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
                try {
                    for (;;) {
                        try {
                            cf.channel().writeAndFlush(br.readLine() + "\n");
                        } catch (IOException e) {
                            break;
                        }
                    }
                } finally {
                    try {
                        br.close();
                    } catch (IOException ignored) {
                    }
                }
            }).start();
            cf.channel().closeFuture().sync();
        } finally {
            eventLoopGroup.shutdownGracefully();
        }
    }
}

public class ChatClientHandler extends SimpleChannelInboundHandler<String> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        System.out.println(msg);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

## WebSocket

基于HTTP协议的通信方式，服务端是不能主动向客户端发送数据的。因此如果需要实现服务端向客户端推送消息的功能时，往往是比较麻烦且耗资源的。比如实时聊天，由于服务端不能主动通知客户端有人给他发消息，所以客户端只能轮询服务端，每隔一定时间询问服务器是否有发给他发消息

`WebSocket`是随`H5`诞生的产物，基于`HTTP`，它通过在`HTTP`包中携带`WebSocket`相关参数来实现服务端与客户端真正意义上的全双工通信。Netty实现了对`WebSocket`的支持。

### WebSocket效果

```yml
# response headers
HTTP/1.1 101 Switching Protocols
upgrade: websocket
connection: upgrade
sec-websocket-accept: ykIala+RcnJNVzQGKkfi44eCtTo=

# request headers
GET ws://localhost:9999/ea HTTP/1.1
Host: localhost:9999
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Key: L4OAYx4RnZL4j4ZqTlXyXg==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```

### 代码

```java
public class WebSocketServer {
    public static void main(String[] args) throws InterruptedException {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup(4);
        ServerBootstrap bootstrap = new ServerBootstrap();
        try {
            bootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline()
                                    .addLast(new HttpServerCodec())
                                    .addLast(new ChunkedWriteHandler())
                                    .addLast(new HttpObjectAggregator(8192))
                                 	// ws://localhost:9999/z
                                    .addLast(new WebSocketServerProtocolHandler("/z"))
                                    .addLast(new TextWebSocketFrameHandler());
                        }
                    });
            ChannelFuture cf = bootstrap.bind(9999).sync();
            cf.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}

public class TextWebSocketFrameHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {
        System.out.println("from client : " + msg.text());
        ctx.channel().writeAndFlush(new TextWebSocketFrame("from server : " + LocalDateTime.now()));
    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        System.out.println("handlerAdded : " + ctx.channel().id().asLongText());
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        System.out.println("handlerRemoved : " + ctx.channel().id().asLongText());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>WebSocket</title>
    </head>
    <body>
        <form onsubmit="return false">
            <textarea name="msg" style="width:400px;height:200px"></textarea>
            <button type="submit" onclick="send(this.form.msg.value)">send</button>
            <textarea id="response" style="width:400px;height:200px"></textarea>
            <button type="submit" onclick=
                    "document.querySelector('#response').value=''">clear
            </button>
        </form>
    </body>
    <script type="text/javascript">
        let socket = window.WebSocket;
        const text = document.querySelector('#response');
        if (socket) {
            socket = new WebSocket('ws://localhost:9999/ea')
            socket.onmessage = function (evt) {
                text.value += '\n' + evt.data
            }
            socket.onopen = function (evt) {
                text.value += '\nconnected'
            }
            socket.onclose = function (evt) {
                text.value += '\ndisconnected'
            }
        } else {
            alert('WebSocket Not Supported!!!')
        }

        function send(msg) {
            if (!socket) return;
            if (socket.readyState == WebSocket.OPEN) {
                socket.send(msg)
            } else {
                alert('WebSocket Not Supported!!!')
            }
        }
    </script>
</html>
```

## ByteBuf

> 相比于`JDK`的`ByteBuffer`，`Netty`的`ByteBuf`给我的直观感受就是在操作上更加方便

`Netty`的数据处理API通过两个组件暴露——`abstract class ByteBuf`和`interface ByteBufHolder`。

下面是一些`ByteBuf API`的优点：

- 它可以被用户自定义的缓冲区类型扩展；

- 通过内置的复合缓冲区类型实现了透明的零拷贝；

- 容量可以按需增长（类似于JDK的`StringBuilder`）；

- 在读和写这两种模式之间切换不需要调用`ByteBuffer`的`flip()`方法；

- 读和写使用了不同的索引；

- 支持方法的链式调用；

- 支持引用计数；

- 支持池化。

### 工作原理

![image-20201123174424133](https://gitee.com/p8t/picbed/raw/master/imgs/20201123174425.png)

### 索引管理

`ByteBuf`为读写操作各设置了索引，不需要像`ByteBuffer`那样调用`flip()`来回切换。

除了个别整体调整`ByteBuf`的一些方法外（如`discardReadBytes()`）

- 只有`read*()`方法会移动`readerIndex`，`get*()`不会移动`readerIndex`
- 只有`write*()`方法会移动`writerIndex`，`set*()`不会移动`writerIndex`

```java
// 返回当前read_index / write_index, 不会移动索引
int readerIndex()
int writerIndex()
// 把read_index / write_index移到索引x处
ByteBuf readerIndex(x)
ByteBuf writerIndex(x)
// 把read_index / write_index移到索引0处
ByteBuf clear()
// 丢弃已读字节, 对应图中把readIndex和writeIndex之间的数据向左移动3位并修改索引
ByteBuf discardReadBytes()
// 标记重置
ByteBuf markReaderIndex()
ByteBuf resetReaderIndex()
ByteBuf markWriterIndex()
ByteBuf resetWriterIndex()
```

## Codec

### 数据包划分

#### LineBasedFrameDecoder

根据换行符划分数据包，发送方需要在每条消息末尾添加换行符。**可用于解决粘包问题**

```java
// 发送方, 这里没什么特别的, 只要在发送消息的地方加上换行符即可
@Override
protected void initChannel(SocketChannel ch) throws Exception {
    ch.pipeline()
        .addLast(new SHandler1());
}
// 接收方
@Override
protected void initChannel(SocketChannel ch) throws Exception {
    ch.pipeline()
        // 分界符解码器, 按换行符把数据包分开
        .addLast(new LineBasedFrameDecoder(4096))
        .addLast(new CHandler1());
}
```

#### DelimiterBasedFrameDecoder

按任意分隔符划分数据包

#### LengthFieldBasedFrameDecoder

消息额外附加长度信息。**可用于解决粘包问题**

附加长度信息可以手动添加，也可以用`LengthFieldPrepender`自动添加，它会自动获取`ByteBuf`可读长度，然后在消息前面添加指定长度的长度信息

```java
// 发送方
@Override
protected void initChannel(SocketChannel ch) throws Exception {
    ch.pipeline()
        // 把在发送的信息前面自动填充4字节的信息长度信息
        // 也可以不用这个, 自己手动填充
        .addLast(new LengthFieldPrepender(4))
        .addLast(new SHandler1());
}
// 接收方
@Override
protected void initChannel(SocketChannel ch) throws Exception {
    ch.pipeline()
        // 最大帧长4096, 超出报异常
        
        // 长度信息记录在这个信息的从0开始, 偏移4个字节的位置
        
        // 一般来说, 长度信息记录的是后面消息的长度, 这时第4个参数置0即可
        // 如果长度信息还包含了它本身的长度, 那么这里置为-4, 表示取消息的时候
        // 少取4个, 因为有4个字节被长度信息占用了
        // 高级用法见JavaDoc, 举了7个例子, 很详细
        
        // 第5个参数表示索引4之前的信息被丢弃, 很好理解, 因为前4个是长度信息
        // 而我们已经获得了正确的数据包, 就不需要它了
        .addLast(new LengthFieldBasedFrameDecoder(4096, 0, 4, 0, 4))
        .addLast(new CHandler1());
}
```

### HTTP

- `HttpResponseEncoder`
- `HttpResponseDecoder`
- `HttpRequestDecoder`
- `HttpRequestEncoder`
- `HttpServerCodec`
- `HttpClientCodec`

服务端使用`HttpResponseEncoder`和`HttpRequestDecoder`，可用`HttpServerCodec`简化；客户端使用`HttpResponseDecoder`和`HttpRequestEncoder`，可用`HttpClientCodec`简化