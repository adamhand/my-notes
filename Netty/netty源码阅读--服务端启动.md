# netty源码阅读--服务端启动
---

# netty版本
`netty 4.1`

# 服务端
服务端启动的关键代码如下：
```
// Configure the server.
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
final EchoServerHandler serverHandler = new EchoServerHandler();
try {
    ServerBootstrap b = new ServerBootstrap();
    b.group(bossGroup, workerGroup)
     .channel(NioServerSocketChannel.class)
     .option(ChannelOption.SO_BACKLOG, 100)
     .handler(new LoggingHandler(LogLevel.INFO))
     .childHandler(new ChannelInitializer<SocketChannel>() {
         @Override
         public void initChannel(SocketChannel ch) throws Exception {
             ChannelPipeline p = ch.pipeline();
             if (sslCtx != null) {
                 p.addLast(sslCtx.newHandler(ch.alloc()));
             }
             //p.addLast(new LoggingHandler(LogLevel.INFO));
             p.addLast(serverHandler);
         }
     });

    // Start the server.
    ChannelFuture f = b.bind(PORT).sync();

    // Wait until the server socket is closed.
    f.channel().closeFuture().sync();
} finally {
    // Shut down all event loops to terminate all threads.
    bossGroup.shutdownGracefully();
    workerGroup.shutdownGracefully();
}
```
和客户端差不多，服务端启动的时候也是做了下面几件事：

- 初始化EventLoopGroup。这里初始化了bossGroup和workerGroup两个EventLoopGroup，实际上是用来reactor的主从模式。从名字也可以看出，一个是“老板”，一个是“工人”。“老板”负责从外面接活，接到的活分配给“工人”干，与之对应，bossGroup的作用就是不断地accept到新的连接，将新的连接丢给workerGroup来处理
- 通过bootstrap函数来初始化channel并制定option，并初始化ChannelHandler
- 通过bind函数启动服务端并绑定端口


## ChannelFacory的初始化过程
client端初始化channel的时候类型为NioSocketChannel，而此处服务端的类型为NioServerSocketChannel。

和客户端一样，`.channel()`其实是`AbstractBootstrap`类中的函数，它返回的是一个`ReflectiveChannelFactory`类的`ChannelFactory`，ReflectiveChannelFactory的构造函数关键代码如下，它最主要的作用就是将constructor的值保存，而clazz就是从`.channel()`中传进去的NioServerSocketChannel。此时Channel并没有初始化，而是等到bind函数的时候才初始化的。

```
public ReflectiveChannelFactory(Class<? extends T> clazz) {
    ...
    this.constructor = clazz.getConstructor();
    ...
}
```

## NioServerSocketChannel的实例化过程
还是先看一下NioServerSocketChannel的继承关系图：

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/nioserversocketchannel.PNG">
</center>

沿着bind()函数往下走，一路找到AbstractBootstrap.doBind()函数，它的关键逻辑如下：
```
private ChannelFuture doBind(final SocketAddress localAddress) {
    final ChannelFuture regFuture = initAndRegister();
    final Channel channel = regFuture.channel();
    ...
    ChannelPromise promise = channel.newPromise();
    doBind0(regFuture, channel, localAddress, promise);
    ...
}
```
接下来看一下initAndRegister()函数，它的关键逻辑如下：
```
final ChannelFuture initAndRegister() {
    Channel channel = null;
    channel = channelFactory.newChannel();
    init(channel);
    ...
    ChannelFuture regFuture = config().group().register(channel);
    ...
}
```
此处的channelFactory就是上一节中的ReflectiveChannelFactory，它的newChannel()方法如下，根据上一小节的分析，此处创建的是NioServerSocketChannel对象，调用的是NioServerSocketChannel的构造方法。
```
public T newChannel() {
    ...
    return constructor.newInstance();
    ...
}
```
NioServerSocketChannel的构造方法如下：
```
public NioServerSocketChannel() {
    this(newSocket(DEFAULT_SELECTOR_PROVIDER));
}
```
和client初始化时一样，此处也是经过了多个构造方法的调用，并且调用了父类的构造方法，下面直接看一下这些构造方法：
```
public NioServerSocketChannel(ServerSocketChannel channel) {
    super(null, channel, SelectionKey.OP_ACCEPT);
    config = new NioServerSocketChannelConfig(this, javaChannel().socket());
}

protected AbstractNioMessageChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent, ch, readInterestOp);
}

protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent);
    this.ch = ch;
    this.readInterestOp = readInterestOp;
    ...
    ch.configureBlocking(false);
    ...
}

protected AbstractChannel(Channel parent) {
    this.parent = parent;
    id = newId();
    unsafe = newUnsafe();
    pipeline = newChannelPipeline();
}
```
调用NioServerSocketChannel()方法之后首先会进入newSocket()函数，之后会调用AbstractChannel()、AbstractNioChannel()、AbstractNioMessageChannel()和NioServerSocketChannel()完成初始化。先看一下newSocket()方法。
```
private static ServerSocketChannel newSocket(SelectorProvider provider) {
    ..
    return provider.openServerSocketChannel();
    ...
}
```
可以看到，这里调用openServerSocketChannel()方法打开一个channel。如果顺着这个方法往下看，会发现最终绑定到了一个FileDescriptor。和client初始化时类似。这时，一个NioServerSocketChannel就创建完成了。

回到initAndRegister()方法，接下来会调用init()方法对channel进行初始化。追踪这个方法，进入ServerBootstrap.init()，关键代码如下：
```
void init(Channel channel) throws Exception {
    ...
    ChannelPipeline p = channel.pipeline();

    final EventLoopGroup currentChildGroup = childGroup;
    final ChannelHandler currentChildHandler = childHandler;
    final Entry<ChannelOption<?>, Object>[] currentChildOptions;
    final Entry<AttributeKey<?>, Object>[] currentChildAttrs;
    synchronized (childOptions) {
        currentChildOptions = childOptions.entrySet().toArray(newOptionArray(0));
    }
    ...
    p.addLast(new ChannelInitializer<Channel>() {
        @Override
        public void initChannel(final Channel ch) throws Exception {
            final ChannelPipeline pipeline = ch.pipeline();
            ChannelHandler handler = config.handler();
            if (handler != null) {
                pipeline.addLast(handler);
            }

            ch.eventLoop().execute(new Runnable() {
                @Override
                public void run() {
                    pipeline.addLast(new ServerBootstrapAcceptor(
                            ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                }
            });
        }
    });
}
```
这个方法主要是为channel绑定了一个ChannelPipeline，并且为这个ChannelPipeline添加了一个 ChannelInitializer, 而这个 ChannelInitializer 中添加了一个 ServerBootstrapAcceptor handler。下面是关注一下ServerBootstrapAcceptor handler类。

先看一下这个类的构造方法：
```
ServerBootstrapAcceptor(...) {
    ...
    enableAutoReadTask = new Runnable() {
        @Override
        public void run() {
            channel.config().setAutoRead(true);
        }
    };
}
```
这里起了一个线程，将AutoRead设置为true。看这个构造方法的注释得知，这个线程的作用是用来处理线程(EventLoop)接受connection失败的情况。当某个EventLoop接受connection失败时，会将AutoRead设置为false并切换至另一个EventLoop，这个EventLoop会将AutoRead设置为true尝试接受connection。

ServerBootstrapAcceptor 中重写了 channelRead 方法, 其主要代码如下:
```
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    final Channel child = (Channel) msg;
    child.pipeline().addLast(childHandler);
    ...
    childGroup.register(child).addListener(...);
}
```
ServerBootstrapAcceptor 中的 childGroup 是构造此对象是传入的 currentChildGroup, 即workerGroup, 而 Channel 是一个 NioSocketChannel 的实例, 因此这里的 childGroup.register 就是将 workerGroup 中的某个 EventLoop 和 NioSocketChannel 关联了。

那么现在的问题是, ServerBootstrapAcceptor.channelRead 方法是怎么被调用的呢? 其实当一个 client 连接到 server 时, Java 底层的 NIO ServerSocketChannel 会有一个 **SelectionKey.OP_ACCEPT** 就绪事件, 接着就会调用到 NioServerSocketChannel.doReadMessages：
```
protected int doReadMessages(List<Object> buf) throws Exception {
    SocketChannel ch = javaChannel().accept();
    buf.add(new NioSocketChannel(this, ch));
    return 1;
}
```
在 doReadMessages 中, 通过 javaChannel().accept() 获取到客户端新连接的 SocketChannel, 接着就实例化一个 NioSocketChannel, 并且传入 NioServerSocketChannel 对象(即 this), 由此可知, 创建的这个 NioSocketChannel 的父 Channel 就是 NioServerSocketChannel 实例 。

接下来就经由 Netty 的 ChannelPipeline 机制, 将读取事件逐级发送到各个 handler 中, 于是就会触发前面提到的 ServerBootstrapAcceptor.channelRead 方法。

回到initAndRegister()方法，当init()方法调用完之后，会调用register()方法对channel进行注册。这个注册的过程和client启动的时候差不多，就不再赘述了。

回到AbstractBootstrap.doBind()方法，当channel注册完之后，会调用doBind0()方法。按照doBind0()方法走下去，会走到AbstractChannel.bind()方法，继续走下去，会走到DefaultChannelPipeline.bind()，分别如下：
```
public ChannelFuture bind(SocketAddress localAddress, ChannelPromise promise) {
    return pipeline.bind(localAddress, promise);
}
```
```
public ChannelFuture bind(SocketAddress localAddress, ChannelPromise promise) {
    return tail.bind(localAddress, promise);
}
```
# 此处有一个问题，继续跟踪下去，发现一条引用链循环，可能是某些地方没有追踪正确。后面再仔细看看。

## handler 的添加过程
服务器端的 handler 的添加过程和客户端的有点区别, 和 EventLoopGroup 一样, 服务器端的 handler 也有两个, 一个是通过 handler() 方法设置 handler 字段, 另一个是通过 childHandler() 设置 childHandler 字段。通过前面的 bossGroup 和 workerGroup 的分析, 其实在这里可以大胆地猜测: handler 字段与 accept 过程有关, 即这个 handler 负责处理客户端的连接请求; 而 childHandler 就是负责和客户端的连接的 IO 交互。

回到init()函数：
```
void init(Channel channel) throws Exception {
    ...
    ChannelPipeline p = channel.pipeline();
    final EventLoopGroup currentChildGroup = childGroup;
    final ChannelHandler currentChildHandler = childHandler;
    final Entry<ChannelOption<?>, Object>[] currentChildOptions;
    final Entry<AttributeKey<?>, Object>[] currentChildAttrs;
    synchronized (childOptions) {
        currentChildOptions = childOptions.entrySet().toArray(newOptionArray(0));
    }
    ...
    p.addLast(new ChannelInitializer<Channel>() {
        @Override
        public void initChannel(final Channel ch) throws Exception {
            final ChannelPipeline pipeline = ch.pipeline();
            ChannelHandler handler = config.handler();
            if (handler != null) {
                pipeline.addLast(handler);
            }
            ch.eventLoop().execute(new Runnable() {
                @Override
                public void run() {
                    pipeline.addLast(new ServerBootstrapAcceptor(
                            ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                }
            });
        }
    });
}
```
上面代码的 initChannel 方法中, 首先通过 handler() 方法获取一个 handler, 如果获取的 handler 不为空,则添加到 pipeline 中。 然后接着, 添加了一个 ServerBootstrapAcceptor 实例。 这里 handler() 方法返回的就是在服务器端的启动代码中设置的:
```
b.group(bossGroup, workerGroup)
 ...
 .handler(new LoggingHandler(LogLevel.INFO))
```
那么这个时候, pipeline 中的 handler 情况如下:
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/netty_server_3.png">
</center>

根据分析客户端的经验, 当 channel 绑定到 eventLoop 后(在这里是 NioServerSocketChannel 绑定到 bossGroup)中时, 会在 pipeline 中发出 fireChannelRegistered 事件, 接着就会触发 ChannelInitializer.initChannel 方法的调用。

因此在绑定完成后, 此时的 pipeline 的内如如下:
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/netty_server_4.png">
</center>

前面在分析 bossGroup 和 workerGroup 时, 已经知道了在 ServerBootstrapAcceptor.channelRead 中会为新建的 Channel 设置 handler 并注册到一个 eventLoop 中, 即:
```
@Override
@SuppressWarnings("unchecked")
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    final Channel child = (Channel) msg;
    child.pipeline().addLast(childHandler);
    ...
    childGroup.register(child).addListener(...);
}
```
而这里的 childHandler 就是在服务器端启动代码中设置的 handler:
```
b.group(bossGroup, workerGroup)
 ...
 .childHandler(new ChannelInitializer<SocketChannel>() {
     @Override
     public void initChannel(SocketChannel ch) throws Exception {
         ChannelPipeline p = ch.pipeline();
         if (sslCtx != null) {
             p.addLast(sslCtx.newHandler(ch.alloc()));
         }
         //p.addLast(new LoggingHandler(LogLevel.INFO));
         p.addLast(new EchoServerHandler());
     }
 });
```
当这个客户端连接 Channel 注册后, 就会触发 ChannelInitializer.initChannel 方法的调用, 此后的客户端连接的 ChannelPipeline 状态如下:
 
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/netty_server_5.png">
</center>

最后总结一下服务器端的 handler 与 childHandler 的区别与联系:

- 在服务器 NioServerSocketChannel 的 pipeline 中添加的是 handler 与 ServerBootstrapAcceptor.
- 当有新的客户端连接请求时, ServerBootstrapAcceptor.channelRead 中负责新建此连接的 NioSocketChannel 并添加 childHandler 到 NioSocketChannel 对应的 pipeline 中, 并将此 channel 绑定到 workerGroup 中的某个 eventLoop 中.
- handler 是在 accept 阶段起作用, 它处理客户端的连接请求.
- childHandler 是在客户端连接建立以后起作用, 它负责客户端连接的 IO 交互.

下面用一幅图来总结一下服务器端的 handler 添加流程:

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/netty_server_6.png">
</center>

