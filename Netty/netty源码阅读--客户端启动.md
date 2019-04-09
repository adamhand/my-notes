# netty源码阅读--客户端启动

# netty版本

- `netty 4.1`

# Bootstrap
`Bootstrp`用于`netty`客户端和服务端的初始化。下面从一个例子开始阅读源码。代码路径为：`netty-4.1\example\src\main\java\io\netty\example\echo`。

# 客户端
下面是客户端启动的关键代码：
```java
EventLoopGroup group = new NioEventLoopGroup();
try {
    Bootstrap b = new Bootstrap();
    b.group(group)
     .channel(NioSocketChannel.class)
     .option(ChannelOption.TCP_NODELAY, true)
     .handler(new ChannelInitializer<SocketChannel>() {
         @Override
         public void initChannel(SocketChannel ch) throws Exception {
             ChannelPipeline p = ch.pipeline();
             if (sslCtx != null) {
                 p.addLast(sslCtx.newHandler(ch.alloc(), HOST, PORT));
             }
             //p.addLast(new LoggingHandler(LogLevel.INFO));
             p.addLast(new EchoClientHandler());
         }
     });

    // Start the client.
    ChannelFuture f = b.connect(HOST, PORT).sync();

    // Wait until the connection is closed.
    f.channel().closeFuture().sync();
} finally {
    // Shut down the event loop to terminate all threads.
    group.shutdownGracefully();
}
```

客户端初始化的基本过程如下：

- 新建`EventLoopGroup `并添加到`BootStrap`中
- 添加`channel`并指定类型和选项
- 添加`ChannelHandler`和`ChannelPipeline `
- 通过`connect`开启`channel`

## Channel建立过程
在 Netty 中, Channel 是一个 Socket 的抽象, 它为用户提供了关于 Socket 状态以及对 Socket 的读写等操作。这里新建的是一个 NioSocketChannel，它的继承关系图如下图所示：

<div align = "center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/niosocketchannel.PNG">
</div>

下面看一下channel()方法：
```java
public B channel(Class<? extends C> channelClass) {
    if (channelClass == null) {
        throw new NullPointerException("channelClass");
    }
    return channelFactory(new ReflectiveChannelFactory<C>(channelClass));
}
```
这个方法返回了一个ReflectiveChannelFactory类型的ChannelFactory，由名字可以看出，这是一个工厂类，用于生产Channel，ReflectiveChannelFactory中用于产生Channel的函数为newChannel，它的逻辑如下：
```java
public T newChannel() {
    try {
        return constructor.newInstance();
    } catch (Throwable t) {
        throw new ChannelException("Unable to create Channel from class " + constructor.getDeclaringClass(), t);
    }
}
```
不过此时这个方法还没有调用，而是等到下面使用BootStrap.connet()方法的时候才会调用，这里只是生成了一个Channel工厂。

当调用BootStrap.connet()方法时，才真正开始实例化一个Channel，它的引用链为：
```java
Bootstrap.connect -> Bootstrap.doResolveAndConnect -> AbstractBootstrap.initAndRegister
```
initAndRegister的关键语句为：
```java
channel = channelFactory.newChannel();
init(channel);
ChannelFuture regFuture = config().group().register(channel);
```
newChannel()方法就是刚才所说的实例化Channel的方法，它的关键逻辑如下：
```java
return constructor.newInstance();
```
这里的constructor是Constructor的一个实例，Constructor的作用是调用相应类的构造方法对对象进行初始化，constructor的实例代码如下：
```java
private final Constructor<? extends T> constructor;
```
而在`b.group(group).channel(NioSocketChannel.class)`时传入的是`NioSocketChannel.class`，所以，这里其实是调用的NioSocketChannel的构造方法进行的初始化，该构造方法如下：
```java
 public NioSocketChannel() {
        this(DEFAULT_SELECTOR_PROVIDER);
    }
```
该构造方法依次调用以下构造方法1、2、3，在构造方法1中会进行newSocket操作，在构造方法3中显式调用了父类AbstractNioByteChannel的构造函数4。构造函数4会继续调用父类AbstractNioChannel的构造函数5。然后继续调用父类AbstractChannel的构造函数6。构造函数6中进行了两个比较重要的操作：`unsafe = newUnsafe();pipeline = newChannelPipeline();`，这个在后面详细看。

newSocket()操作是先于父类构造函数执行的，也就是说，执行顺序是：

- newSocket(provider)   
- AbstractChannel()
- AbstractNioChannel()
- AbstractNioByteChannel()
- NioSocketChannel(Channel parent, SocketChannel socket)
- NioSocketChannel(SocketChannel socket)
- NioSocketChannel(SelectorProvider provider)

```
//构造方法1
public NioSocketChannel(SelectorProvider provider) {
    this(newSocket(provider));
}
//构造方法2
public NioSocketChannel(SocketChannel socket) {
    this(null, socket);
}
//构造方法3
public NioSocketChannel(Channel parent, SocketChannel socket) {
    super(parent, socket);
    config = new NioSocketChannelConfig(this, socket.socket());
}
//构造函数4
protected AbstractNioByteChannel(Channel parent, SelectableChannel ch) {
    super(parent, ch, SelectionKey.OP_READ);
}
//构造函数5
protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent);
    this.ch = ch;
    this.readInterestOp = readInterestOp;
    //配置 Java NIO SocketChannel 为非阻塞的
    ch.configureBlocking(false);
    ...
}
//构造函数6
protected AbstractChannel(Channel parent) {
    this.parent = parent;
    id = newId();
    unsafe = newUnsafe();
    pipeline = newChannelPipeline();
}
```

下面看一下newSocket()的执行过程。它的引用链为：
```
NioSocketChannel.newSocket() -> SelectorProviderImpl.openSocketChannel() -> SocketChannelImpl.SocketChannelImpl() -> Net.socket() -> Net.socket(ProtocolFamily var0, boolean var1) -> IOUtil.newFD() ->  FileDescriptor var1 = new FileDescriptor();
```
可以看到，最终是绑定了一个FileDescriptor。

到这里, 一个完整的 NioSocketChannel 就初始化完成了, 可以稍微总结一下构造一个 NioSocketChannel 所需要做的工作:

- 调用 NioSocketChannel.newSocket(DEFAULT_SELECTOR_PROVIDER) 打开一个新的 Java NIO SocketChannel
- AbstractChannel(Channel parent) 中初始化 AbstractChannel 的属性:
    - parent 属性置为 null
    - unsafe 通过newUnsafe() 实例化一个 unsafe 对象, 它的类型是 AbstractNioByteChannel.NioByteUnsafe 内部类
    - pipeline 是 new DefaultChannelPipeline(this) 新创建的实例. 这里体现了:Each channel has its own pipeline and it is created automatically when a new channel is created.
- AbstractNioChannel 中的属性:
    - SelectableChannel ch 被设置为 Java SocketChannel, 即 NioSocketChannel#newSocket 返回的 Java NIO SocketChannel.
    - readInterestOp 被设置为 SelectionKey.OP_READ
    - SelectableChannel ch 被配置为非阻塞的 ch.configureBlocking(false)
- NioSocketChannel 中的属性:
    - SocketChannelConfig config = new NioSocketChannelConfig(this, socket.socket())
    
## Unsafe字段的初始化
刚才说到，AbstractChannel()构造函数中对Unsafe字段进行了初始化：
```
unsafe = newUnsafe();
```
Unsafe接口的逻辑如下，它封装了对 Java 底层 Socket 的操作, 因此Unsafe实际上是沟通 Netty 上层和 Java 底层的重要的桥梁。
```
interface Unsafe {
    SocketAddress localAddress();
    SocketAddress remoteAddress();
    void register(EventLoop eventLoop, ChannelPromise promise);
    void bind(SocketAddress localAddress, ChannelPromise promise);
    void connect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise);
    void disconnect(ChannelPromise promise);
    void close(ChannelPromise promise);
    void closeForcibly();
    void deregister(ChannelPromise promise);
    void beginRead();
    void write(Object msg, ChannelPromise promise);
    void flush();
    ChannelPromise voidPromise();
    ChannelOutboundBuffer outboundBuffer();
}
```
newUnsafe()函数的逻辑如下，它返回一个 NioSocketChannelUnsafe 实例。
```
protected AbstractNioUnsafe newUnsafe() {
    return new NioSocketChannelUnsafe();
}
```
NioSocketChannelUnsafe的继承关系如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/niosocketchannelunsafe.PNG">
</center>

## pipeline的初始化
每初始化一个Channel，都会伴随着初始化一个pipeline。在AbstractChannel中对pipeline进行了初始化：
```
pipeline = newChannelPipeline();
```
DefaultChannelPipeline 构造器如下：
```
protected DefaultChannelPipeline(Channel channel) {
    this.channel = ObjectUtil.checkNotNull(channel, "channel");
    succeededFuture = new SucceededChannelFuture(channel, null);
    voidPromise =  new VoidChannelPromise(channel, true);

    tail = new TailContext(this);
    head = new HeadContext(this);

    head.next = tail;
    tail.prev = head;
}
```
调用 DefaultChannelPipeline 的构造器, 传入了一个 channel, 而这个 channel 其实就是刚才实例化的 NioSocketChannel, DefaultChannelPipeline 会将这个 NioSocketChannel 对象保存在channel 字段中. DefaultChannelPipeline 中, 还有两个特殊的字段, 即 head 和 tail, 而这两个字段是一个双向链表的头和尾。 其实在 DefaultChannelPipeline 中, 维护了一个以 AbstractChannelHandlerContext 为节点的双向链表, 这个链表是 Netty 实现 Pipeline 机制的关键。

TailContext 的继承层次结构如下所示:

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/tailcontext.PNG">
</center>

HeadContext 的继承层次结构如下所示:

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/headcontext.PNG">
</center>

## NioEventLoopGroup的初始化
NioEventLoopGroup的继承关系图如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/NioEventLoopGroup.PNG">
</center>

NioEventLoopGroup一共有几个构造函数：
```
// 1
public NioEventLoopGroup() {
    this(0);
}
public NioEventLoopGroup(int nThreads) {
    this(nThreads, (Executor) null);
}
    public NioEventLoopGroup(int nThreads, Executor executor) {
    this(nThreads, executor, SelectorProvider.provider());
}    
public NioEventLoopGroup(
        int nThreads, Executor executor, final SelectorProvider selectorProvider) {
    this(nThreads, executor, selectorProvider, DefaultSelectStrategyFactory.INSTANCE);
}
public NioEventLoopGroup(int nThreads, Executor executor, final SelectorProvider selectorProvider,final SelectStrategyFactory selectStrategyFactory) {
    super(nThreads, executor, selectorProvider, selectStrategyFactory, RejectedExecutionHandlers.reject());
}
```
这几个构造函数最后调用到父类的构造函数，如下所示：
```
// 2
protected MultithreadEventLoopGroup(int nThreads, ThreadFactory threadFactory, Object... args) {
    super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, threadFactory, args);
}
```
可以看到，这里会对nTreads做一个判断，如果nThreads为0，就创建DEFAULT_EVENT_LOOP_THREADS个线程。而在构造函数1中可以看到，如果在创建NioEventLoopGroup时不指定线程数，传入的就是0。也就是说，如果不传入参数，默认创建DEFAULT_EVENT_LOOP_THREADS个线程，DEFAULT_EVENT_LOOP_THREADS会在静态语句块中被初始化，为当前CPU内核数的两倍，如下：
```
static {
    DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
            "io.netty.eventLoopThreads", NettyRuntime.availableProcessors() * 2));

    if (logger.isDebugEnabled()) {
        logger.debug("-Dio.netty.eventLoopThreads: {}", DEFAULT_EVENT_LOOP_THREADS);
    }
}
```
构造函数2又会继续调用父类的构造函数：
```
protected MultithreadEventExecutorGroup(int nThreads, Executor executor,
                                            EventExecutorChooserFactory chooserFactory, Object... args) { ...   }
```
这个构造器代码比较长，可以分为三部分：

- new ThreadPerTaskExecutor();创建一个线程选择器
- for (){new Child()}构造NioEventLoop
- chooserFactory.newChooser(children);创建线程选择器

### 第一部分
第一部分对应的代码如下：
```
if (executor == null) {
    executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
}
```
ThreadPerTaskExecutor的作用是每次执行任务之前都会创建一个线程实体。它的构造器如下：
```
public ThreadPerTaskExecutor(ThreadFactory threadFactory) {
    ...
    this.threadFactory = threadFactory;
}

@Override
public void execute(Runnable command) {
    threadFactory.newThread(command).start();
}
```
可以看到，会根据创建来的线程工厂ThreadFactory来新建线程并启动。这里比较重点的是newThread()方法。这个方法在DefalutThreadFactory类中，关键代码下：
```
public Thread newThread(Runnable r) {
    Thread t = newThread(FastThreadLocalRunnable.wrap(r), prefix + nextId.incrementAndGet());
    ...
}
```
继续跟进newThread方法：
```
protected Thread newThread(Runnable r, String name) {
    return new FastThreadLocalThread(threadGroup, r, name);
}
```
这里的FastThreadLocalThread继承自Java的Thread，对ThreadLocal中的操作做了优化，并且自己包装了一个ThreadLocalMap。


另外，还需要知道一点，即NioEventLoop的命名规则为：nioEventLoop-xx-yy。其中，xx为EventLoopGroup的编号，yy为EventLoop在Group中的编号。这一点是从newDefaultThreadFactory()方法中得知的。该方法的逻辑如下：
```
protected ThreadFactory newDefaultThreadFactory() {
    return new DefaultThreadFactory(getClass());
}
```
沿着DefaultThreadFactory的构造函数调用链一路追踪，直到下面的构造函数：
```
public DefaultThreadFactory(Class<?> poolType, boolean daemon, int priority) {
    this(toPoolName(poolType), daemon, priority);
}
```
这里的poolName()方法的作用就是将NioEventLoop名字的首字母变为小写。继续跟进，发现下面的构造函数：
```
public DefaultThreadFactory(String poolName, boolean daemon, int priority, ThreadGroup threadGroup) {
    ...
    prefix = poolName + '-' + poolId.incrementAndGet() + '-';
    ...
}
```
这里就清楚了，是将poolName做了自增操作，并加上了下划线。

### 第二部分
回到MultithreadEventExecutorGroup()构造函数，看第二部分newChild()的操作。

这一部分其实主要做了两件事：

- 创建一个selector
- 创建一个MpscQueue

从newChild()方法出发，经过如下引用链：
```
NioEventLoopGroup.new Child() -> NioEventLoop.NioEventLoop()
```
在NioEventLoop()构造函数中有两句句比较重要的逻辑：
```
NioEventLoop(NioEventLoopGroup parent, Executor executor, SelectorProvider selectorProvider,
             SelectStrategy strategy, RejectedExecutionHandler rejectedExecutionHandler) {
    super(parent, executor, false, DEFAULT_MAX_PENDING_TASKS, rejectedExecutionHandler);
    final SelectorTuple selectorTuple = openSelector();
    selector = selectorTuple.selector;
}
```
其中selector = selectorTuple.selector;的意思就是创建一个selector。openSelector()中的逻辑比较简单，主要是通过SelectorProvider来打开一个Selector。

沿着super看向父类的构造函数，在SingleThreadEventExecutor中有重要逻辑如下：
```
taskQueue = newTaskQueue(this.maxPendingTasks);
```
进入NioEventLoop中的newTaskQueue方法看一下，如下：
```
protected Queue<Runnable> newTaskQueue(int maxPendingTasks) {
    // This event loop never calls takeTask()
    return maxPendingTasks == Integer.MAX_VALUE ? PlatformDependent.<Runnable>newMpscQueue()
                                                : PlatformDependent.<Runnable>newMpscQueue(maxPendingTasks);
}
```
通过newMpscQueue方法建立了MpscQueue(mpsc-multiple producers (different threads) and a singl consumer (one thread))。MpscQueue是netty实现的线程安全的队列，与JDK通过锁实现的BlockingQueue不同，MpscLinkedQueue是一种针对Netty中NIO任务设计的一种队列。这里先不深入去探究。

### 第三部分
看一下创建选择器的代码：
```
chooser = chooserFactory.newChooser(children);
```
这段代码的意思就是创建选择器，选择器的作用就是当创建连接的之后，选择哪个EventLoop与其绑定。它的逻辑很简单，就是通过next()方法在上面创建了的EventLoop数组中寻找EventLoop进行绑定，即每来一个连接都会找下一个EventLoop进行绑定。但是netty对next()方法做了一个优化，点进去看newChooser()的实现：
```
public EventExecutorChooser newChooser(EventExecutor[] executors) {
    if (isPowerOfTwo(executors.length)) {
        return new PowerOfTwoEventExecutorChooser(executors);
    } else {
        return new GenericEventExecutorChooser(executors);
    }
}
```
这里判断executors即EventLoop的数量是否为2的幂次方，如果是的话，会调用PowerOfTwoEventExecutorChooser，否则会调用普通的GenericEventExecutorChooser方法。PowerOfTwoEventExecutorChooser方法对next()做了优化，如下所示：
```
public EventExecutor next() {
    return executors[idx.getAndIncrement() & executors.length - 1];
}
```
即`idx++ & executors.length - 1`。而GenericEventExecutorChooser的next()方法如下：
```
public EventExecutor next() {
    return executors[Math.abs(idx.getAndIncrement() % executors.length)];
}
```
即`idx++ % executors.length`。可以看到，PowerOfTwoEventExecutorChooser方法将除法运算替换为取模运算，运算速度提高了。

## channel 的注册过程
回到AbstractBootstrap.initAndRegister()方法，这个方法会不仅会创建channel，还会将channel注册到EventLoopGroup上，如下代码：
```
ChannelFuture regFuture = config().group().register(channel);
```
追踪register的调用链，发现最终调用了unsafe的register()方法，如下：
```
AbstractBootstrap.initAndRegister -> 
    MultithreadEventLoopGroup.register -> 
        SingleThreadEventLoop.register -> 
            AbstractUnsafe.register
```
AbstractUnsafe.register的关键逻辑如下：
```
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    AbstractChannel.this.eventLoop = eventLoop;
    register0(promise);
}
```
首先将eventLoop 赋值给 Channel 的 eventLoop 属性，接着调用register0()方法，register0()方法又会调用AbstractNioChannel.doRegister：
```
protected void doRegister() throws Exception {
    ...
    selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), 0, this);
    ...
}
```
javaChannel()返回的是一个 Java NIO SocketChannel, 之后将这个 SocketChannel 注册到与 eventLoop 关联的 selector 上了。

总结一下 Channel 的注册过程:

- 首先在 AbstractBootstrap.initAndRegister中, 通过 group().register(channel), 调用 MultithreadEventLoopGroup.register 方法
- 在MultithreadEventLoopGroup.register 中, 通过 next() 获取一个可用的 SingleThreadEventLoop, 然后调用它的 register
- 在 SingleThreadEventLoop.register 中, 通过 channel.unsafe().register(this, promise) 来获取 channel 的 unsafe() 底层操作对象, 然后调用它的 register
- 在 AbstractUnsafe.register 方法中, 调用 register0 方法注册 Channel
- 在 AbstractUnsafe.register0 中, 调用 AbstractNioChannel.doRegister 方法
- AbstractNioChannel.doRegister 方法通过 javaChannel().register(eventLoop().selector, 0, this) 将 Channel 对应的 Java NIO SockerChannel 注册到一个 eventLoop 的 Selector 中, 并且将当前 Channel 作为 attachment

## handler 的添加过程
hander的添加代码如下：
```
handler(new ChannelInitializer<SocketChannel>() {
     @Override
     public void initChannel(SocketChannel ch) throws Exception {
         ChannelPipeline p = ch.pipeline();
         if (sslCtx != null) {
             p.addLast(sslCtx.newHandler(ch.alloc(), HOST, PORT));
         }
         //p.addLast(new LoggingHandler(LogLevel.INFO));
         p.addLast(new EchoClientHandler());
     }
 })
```
ChannelInitializer 是一个抽象类, 它有一个抽象的方法 initChannel, 添加Channel的时候需要实现这个方法, 并在这个方法中添加的自定义的 handler 。initChannel 会在ChannelInitializer.channelRegistered 方法中被调用。

channelRegistered方法源码如下：
```
public final void channelRegistered(ChannelHandlerContext ctx) throws Exception {
    if (initChannel(ctx)) {
        ctx.pipeline().fireChannelRegistered();
        removeState(ctx);
    } else {
        ctx.fireChannelRegistered();
    }
}
```

从上面的源码中可以看到, 在 channelRegistered 方法中, 会调用 initChannel 方法, 将自定义的 handler 添加到 ChannelPipeline 中, 然后调用 ctx.pipeline().remove(this) 将自己从 ChannelPipeline 中删除。 上面的分析过程, 可以用如下图片展示:

一开始, ChannelPipeline 中只有三个 handler, head, tail 和自定义添加的 ChannelInitializer。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/channelpipeline_1.png">
</center>

接着 initChannel 方法调用后, 添加了自定义的 handler：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/channelpipeline_2.png">
</center>

最后将 ChannelInitializer 删除：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/channelpipeline_3.png">
</center>

## 客户端连接分析
下面看看一下客户端是怎么发起TCP连接的。

首先, 客户端通过调用 Bootstrap 的 connect 方法进行连接。引用链为：
```
BootStrap.connect() -> BootStrap.doResolveAndConnect() -> BootStrap.doResolveAndConnect0() ->  BootStrap.doConnect()
```
BootStrap.doConnect()关键代码如下：
```
private static void doConnect(
        final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise connectPromise) {
    final Channel channel = connectPromise.channel();
    channel.eventLoop().execute(new Runnable() {
        @Override
        public void run() {
            if (localAddress == null) {
                channel.connect(remoteAddress, connectPromise);
            } else {
                channel.connect(remoteAddress, localAddress, connectPromise);
            }
            connectPromise.addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
        }
    });
}
```
在 doConnect 中, 会在 event loop 线程中调用 Channel 的 connect 方法, 而这个 Channel 的具体类型是NioSocketChannel。

进行跟踪到 channel.connect 中, 发现它调用的是 DefaultChannelPipeline#connect, 而, pipeline 的 connect 代码如下:
```
public final ChannelFuture connect(SocketAddress remoteAddress, ChannelPromise promise) {
    return tail.connect(remoteAddress, promise);
}
```
继续追踪，进入AbstractChannelHandlerContext.connect，代码如下：
```
public ChannelFuture connect(
        final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise promise) {
    if (remoteAddress == null) {
        throw new NullPointerException("remoteAddress");
    }
    if (isNotValidPromise(promise, false)) {
        return promise;
    }

    final AbstractChannelHandlerContext next = findContextOutbound();
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeConnect(remoteAddress, localAddress, promise);
    } else {
        safeExecute(executor, new Runnable() {
            @Override
            public void run() {
                next.invokeConnect(remoteAddress, localAddress, promise);
            }
        }, promise, null);
    }
    return promise;
}
```
上面的代码中有一个关键的地方, 即 final AbstractChannelHandlerContext next = findContextOutbound(), 这里调用 findContextOutbound 方法, 从 DefaultChannelPipeline 内的双向链表的 tail 开始, 不断向前寻找第一个 outbound 为 true 的 AbstractChannelHandlerContext, 然后调用它的 invokeConnect 方法, 其代码如下:
```
private void invokeConnect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise) {
    // 忽略 try 块
    ((ChannelOutboundHandler) handler()).connect(this, remoteAddress, localAddress, promise);
}
```
而第一个 outbound 为 true 的 AbstractChannelHandlerContext就是HeadContext。接着跟踪到 HeadContext.connect, 其代码如下:
```
@Override
public void connect(
        ChannelHandlerContext ctx,
        SocketAddress remoteAddress, SocketAddress localAddress,
        ChannelPromise promise) throws Exception {
    unsafe.connect(remoteAddress, localAddress, promise);
}
```
这个 connect 方法很简单, 仅仅调用了 unsafe 的 connect 方法。而 unsafe 是 pipeline.channel().unsafe() 返回的, 而 Channel 的 unsafe 字段, 在这个例子中,其实是 AbstractNioByteChannel.NioByteUnsafe 内部类。进行跟踪 NioByteUnsafe -> AbstractNioUnsafe.connect:
```
@Override
public final void connect(
        final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise promise) {
    boolean wasActive = isActive();
    if (doConnect(remoteAddress, localAddress)) {
        fulfillConnectPromise(promise, wasActive);
    } else {
        ...
    }
}
```
AbstractNioUnsafe.connect 的实现如上代码所示, 在这个 connect 方法中, 调用了 doConnect 方法, 注意, 这个方法并不是 AbstractNioUnsafe 的方法, 而是 AbstractNioChannel 的抽象方法. doConnect 方法是在 NioSocketChannel 中实现的, 因此进入到 NioSocketChannel.doConnect 中:
```
@Override
protected boolean doConnect(SocketAddress remoteAddress, SocketAddress localAddress) throws Exception {
    if (localAddress != null) {
        javaChannel().socket().bind(localAddress);
    }

    boolean success = false;
    try {
        boolean connected = javaChannel().connect(remoteAddress);
        if (!connected) {
            selectionKey().interestOps(SelectionKey.OP_CONNECT);
        }
        success = true;
        return connected;
    } finally {
        if (!success) {
            doClose();
        }
    }
}
```
上面的代码首先是获取 Java NIO SocketChannel, 即从 NioSocketChannel.newSocket 返回的 SocketChannel 对象; 然后是调用 SocketChannel.connect 方法完成 Java NIO 层面上的 Socket 的连接。

最后, 上面的代码流程可以用如下时序图直观地展示:
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/netty_1111111.png">
</center>

## 参考
[netty](https://github.com/netty/netty)
[Netty 源码分析之 一 揭开 Bootstrap 神秘的红盖头 (客户端)](https://segmentfault.com/a/1190000007282789)
慕课网闪电侠netty源码分析