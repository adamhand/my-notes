# netty源码阅读--ChannelPipeline
---

在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应, 它们的组成关系如下:
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/channelpipeline_01.png">
</center>

一个 Channel 包含了一个 ChannelPipeline, 而 ChannelPipeline 中又维护了一个由 ChannelHandlerContext 组成的双向链表。这个链表的头是 HeadContext, 链表的尾是 TailContext, 并且每个 ChannelHandlerContext 中又关联着一个 ChannelHandler。

# ChannelPipeline 实例化过程
在服务端和客户端启动的时候分析过，创建Channel需要使用父类的构造方法，而在调用AbstractChannel的构造方法的时候，会给channel绑定一个ChannelPipeline，这个构造函数如下所示：
```
protected AbstractChannel(Channel parent) {
    this.parent = parent;
    unsafe = newUnsafe();
    pipeline = new DefaultChannelPipeline(this);
}
```
顺着DefaultChannelPipeline往下看，它的构造函数如下：
```
public DefaultChannelPipeline(AbstractChannel channel) {
    if (channel == null) {
        throw new NullPointerException("channel");
    }
    this.channel = channel;

    tail = new TailContext(this);
    head = new HeadContext(this);

    head.next = tail;
    tail.prev = head;
}
```
在 DefaultChannelPipeline 的构造方法中, 将传入的 channel 赋值给字段 this.channel, 接着又实例化了两个特殊的字段: tail 与 head，构成一个双向链表。其实在 DefaultChannelPipeline 中, 就是维护了一个以 AbstractChannelHandlerContext 为节点的双向链表。

head 和 tail 的类层次结构如下:

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/tailcontext.PNG">

`TailContext 的继承层次结构`
</center>

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/headcontext.PNG">

`HeadContext 的继承层次结构`
</center>

从类层次结构图中可以很清楚地看到, head 实现了 ChannelInboundHandler, 而 tail 实现了 ChannelOutboundHandler 接口, 并且它们都实现了 ChannelHandlerContext 接口, 因此可以说 head 和 tail 即是一个 ChannelHandler, 又是一个 ChannelHandlerContext.

HeadContext和TailContext的构造器如下：
```
HeadContext(DefaultChannelPipeline pipeline) {
    super(pipeline, null, HEAD_NAME, false, true);
    unsafe = pipeline.channel().unsafe();
}

TailContext(DefaultChannelPipeline pipeline) {
    super(pipeline, null, TAIL_NAME, true, false);
}
```
HeadContext调用了父类 AbstractChannelHandlerContext 的构造器, 并传入参数 inbound = false, outbound = true。

TailContext 的构造器与 HeadContext 的相反, 它调用了父类 AbstractChannelHandlerContext 的构造器, 并传入参数 inbound = true, outbound = false。

即 header 是一个 outboundHandler, 而 tail 是一个inboundHandler。

# ChannelInitializer 的添加
最开始的时候 ChannelPipeline 中含有两个 ChannelHandlerContext(同时也是 ChannelHandler), 但是这个 Pipeline并不能实现什么特殊的功能, 因为还没有给它添加自定义的 ChannelHandler。

通常来说, 在初始化 Bootstrap时会添加自定义的 ChannelHandler, 比如 EchoClient：
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
ChannelInitializer 实现了 ChannelHandler, 顺着BootStrap.connect()方法往下走，可以发现hander()被添加到ChannelPipeline中的时机，调用链为：
```
Bootstrap.connect() -> 
    Bootstrap.doConnect() -> 
        AbstractBootstrap.initAndRegister() -> 
            Bootstrap.init()
```
具体代码块为：
```
void init(Channel channel) throws Exception {
    ChannelPipeline p = channel.pipeline();
    p.addLast(handler());
    ...
}
```
这里的handler()就是刚才初始化的hander(new Channelinitializer())。因此这里就是将 ChannelInitializer 插入到了 Pipeline 的末端。

进入addLast()方法，一路追踪，进入下面的addLast()方法：
```
public ChannelPipeline addLast(EventExecutorGroup group, final String name, ChannelHandler handler) {
    synchronized (this) {
        checkDuplicateName(name);

        AbstractChannelHandlerContext newCtx = new DefaultChannelHandlerContext(this, group, name, handler);
        addLast0(name, newCtx);
    }

    return this;
}
```
上面的 addLast 方法中, 首先检查这个 ChannelHandler 的名字是否是重复的, 如果不重复的话, 则为这个 Handler 创建一个对应的 DefaultChannelHandlerContext 实例, 并与之关联起来(Context 中有一个 handler 属性保存着对应的 Handler 实例)。判断此 Handler 是否重名的方法很简单: Netty 中有一个 name2ctx Map 字段, key 是 handler 的名字, 而 value 则是 handler 本身。因此通过如下代码就可以判断一个 handler 是否重名了:

```
private void checkDuplicateName(String name) {
    if (name2ctx.containsKey(name)) {
        throw new IllegalArgumentException("Duplicate handler name: " + name);
    }
}
```

为了添加一个 handler 到 pipeline 中, 必须把此 handler 包装成 ChannelHandlerContext. 因此在上面的代码中我们可以看到新实例化了一个 newCtx 对象, 并将 handler 作为参数传递到构造方法中。下面是DefaultChannelHandlerContext的构造器：
```
DefaultChannelHandlerContext(
        DefaultChannelPipeline pipeline, EventExecutorGroup group, String name, ChannelHandler handler) {
    super(pipeline, group, name, isInbound(handler), isOutbound(handler));
    if (handler == null) {
        throw new NullPointerException("handler");
    }
    this.handler = handler;
}
```
其中调用的isInbound()和isOutbound()方法如下所示：
```
private static boolean isInbound(ChannelHandler handler) {
    return handler instanceof ChannelInboundHandler;
}

private static boolean isOutbound(ChannelHandler handler) {
    return handler instanceof ChannelOutboundHandler;
}
```
当一个 handler 实现了 ChannelInboundHandler 接口, 则 isInbound 返回真; 相似地, 当一个 handler 实现了 ChannelOutboundHandler 接口, 则 isOutbound 就返回真。

而这两个 boolean 变量会传递到父类 AbstractChannelHandlerContext 中, 并初始化父类的两个字段: inbound 与 outbound。

想知道这里的 ChannelInitializer 所对应的 DefaultChannelHandlerContext 的 inbound 与 inbound 字段分别是什么，只需要看一下ChannelInitializer 实现了哪个接口就行了，下面是ChannelInitializer 的继承关系图：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/channelinitializer.PNG">
</center>

可以清楚地看到, ChannelInitializer 仅仅实现了 ChannelInboundHandler 接口, 因此这里实例化的 DefaultChannelHandlerContext 的 inbound = true, outbound = false。

当创建好 Context 后, 就将这个 Context 插入到 Pipeline 的双向链表中：
```
private void addLast0(final String name, AbstractChannelHandlerContext newCtx) {
    checkMultiplicity(newCtx);

    AbstractChannelHandlerContext prev = tail.prev;
    newCtx.prev = prev;
    newCtx.next = tail;
    prev.next = newCtx;
    tail.prev = newCtx;

    name2ctx.put(name, newCtx);

    callHandlerAdded(newCtx);
}
```

此时 Pipeline 的结构如下图所示:
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/channelpipeline_02.png">
</center>

# 自定义 ChannelHandler 的添加过程
下面看看一下自定义的 ChannelHandler 是如何插入到 Pipeline 中的。

在前面client初始化章节已经分析了Channel 的注册过程，如下：

- 首先在 AbstractBootstrap.initAndRegister中, 通过 group().register(channel), 调用 MultithreadEventLoopGroup.register 方法
- 在MultithreadEventLoopGroup.register 中, 通过 next() 获取一个可用的 SingleThreadEventLoop, 然后调用它的 register
- 在 SingleThreadEventLoop.register 中, 通过 channel.unsafe().register(this, promise) 来获取 channel 的 unsafe() 底层操作对象, 然后调用它的 register
- 在 AbstractUnsafe.register 方法中, 调用 register0 方法注册 Channel
- 在 AbstractUnsafe.register0 中, 调用 AbstractNioChannel#doRegister 方法
- AbstractNioChannel.doRegister 方法通过 javaChannel().register(eventLoop().selector, 0, this) 将 Channel 对应的 Java NIO SockerChannel 注册到一个 eventLoop 的 Selector 中, 并且将当前 Channel 作为 attachment

完整的调用链如下：
```
BootStrap.connect() -> 
    BootStrap.doConnect() -> 
        AbstractBootstrap.initAndRegister() -> 
            MultithreadEventLoopGroup.register() -> 
                SingleThreadEventLoopGroup.register() -> 
                    SingleThreadEventLoop.register() -> 
                        AbstractChannel.AbstractUnsafe.register() ->
                            AbstractChannel.AbstractUnsafe.register0() ->
                                AbstractNioChannel.doRegister() ->
                                    AbstractSelectableChannel.register() -> 
                                        SelectorImpl.register() ->
                                            WindowsSelectorImpl.implRegister()
```

自定义 ChannelHandler 的添加过程, 发生在 AbstractChanel.AbstractUnsafe.register0 中, 在这个方法中调用了 pipeline.fireChannelRegistered() 方法, 其实现如下：
```
public ChannelPipeline fireChannelRegistered() {
    head.fireChannelRegistered();
    return this;
}
```
上面方法的逻辑就是调用head.fireChannelRegistered()方法。这里需要注意的是，head.fireXXX 的调用形式, 是 Netty 中 Pipeline 传递事件的常用方式。进入AbstractChannelHandlerContext.fireChannelRegistered，它的逻辑如下：
```
public ChannelHandlerContext fireChannelRegistered() {
    final AbstractChannelHandlerContext next = findContextInbound();
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeChannelRegistered();
    } else {
        executor.execute(new OneTimeTask() {
            @Override
            public void run() {
                next.invokeChannelRegistered();
            }
        });
    }
    return this;
}
```
这个方法的第一句是调用 findContextInbound 获取一个 Context，进入这个方法看一下，它的逻辑如下：
```
private AbstractChannelHandlerContext findContextInbound() {
    AbstractChannelHandlerContext ctx = this;
    do {
        ctx = ctx.next;
    } while (!ctx.inbound);
    return ctx;
}
```
可以看到，这个方法会遍历Pipeline 的双向链表, 然后找到第一个属性 inbound 为 true 的 ChannelHandlerContext 实例。

由于ChannelInitializer 实现了 ChannelInboudHandler, 因此它所对应的 ChannelHandlerContext 的 inbound 属性就是 true, 因此这里返回就是 ChannelInitializer 实例所对应的 ChannelHandlerContext. 即:
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/channelpipeline_03.png">
</center>

接着，会调用它的invokeChannelRegistered方法：
```
private void invokeChannelRegistered() {
    try {
        ((ChannelInboundHandler) handler()).channelRegistered(this);
    } catch (Throwable t) {
        notifyHandlerException(t);
    }
}
```
由于每个 ChannelHandler 都与一个 ChannelHandlerContext 关联, 并且可以通过 ChannelHandlerContext 获取到对应的 ChannelHandler。因此这里 handler() 返回的, 其实就是一开始实例化的 ChannelInitializer 对象, 并接着调用了 ChannelInitializer.channelRegistered 方法。逻辑如下：
```
public final void channelRegistered(ChannelHandlerContext ctx) throws Exception {
    initChannel((C) ctx.channel());
    ctx.pipeline().remove(this);
    ctx.fireChannelRegistered();
}
```
这里的initChannel 方法就是在初始化 Bootstrap 时, 调用 handler 方法传入的匿名内部类所实现的方法:
```
.handler(new ChannelInitializer<SocketChannel>() {
     @Override
     public void initChannel(SocketChannel ch) throws Exception {
         ChannelPipeline p = ch.pipeline();
         p.addLast(new EchoClientHandler());
     }
 });
```

因此当调用了这个方法后, 自定义的 ChannelHandler 就插入到 Pipeline 了, 此时的 Pipeline 如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/channelpipeline_4.png">
</center>

当添加了自定义的 ChannelHandler 后, 会删除 ChannelInitializer 这个 ChannelHandler, 即 "ctx.pipeline().remove(this)", 因此最后的 Pipeline 如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/channelpipeline_5.png">
</center>

# ChannelHandler 的名字
上面说了，DefaultChannelPipeline有几个重载的addLast()方法，下面通过addLast()方法看一下ChannelHandlerContext的命名规则。下面的addLast()方法的第一个参数name就是ChannelHandlerContext的名字，或者说是channel的名字。
```
public ChannelPipeline addLast(String name, ChannelHandler handler) {
    return addLast(null, name, handler);
}
```
它会调用下面的重载方法：
```
public ChannelPipeline addLast(EventExecutorGroup group, final String name, ChannelHandler handler) {
    synchronized (this) {
        checkDuplicateName(name);

        AbstractChannelHandlerContext newCtx = new DefaultChannelHandlerContext(this, group, name, handler);
        addLast0(name, newCtx);
    }

    return this;
}
```
可以看到，在添加一个 handler 之前, 需要调用 checkDuplicateName 方法来确定此 handler 的名字是否和已添加的 handler 的名字重复。
```
private void checkDuplicateName(String name) {
    if (name2ctx.containsKey(name)) {
        throw new IllegalArgumentException("Duplicate handler name: " + name);
    }
}
```
Netty 判断一个 handler 的名字是否重复的依据很简单: DefaultChannelPipeline 中有一个 类型为 Map<String, AbstractChannelHandlerContext> 的 name2ctx 字段, 它的 key 是一个 handler 的名字, 而 value 则是这个 handler 所对应的 ChannelHandlerContext. 每当新添加一个 handler 时, 就会 put 到 name2ctx 中。因此检查 name2ctx 中是否包含这个 name 即可。

当没有重名的 handler 时, 就为这个 handler 生成一个关联的 DefaultChannelHandlerContext 对象, 然后就将 name 和 newCtx 作为 key-value 对 放到 name2Ctx 中。

## 自动生成 handler 的名字
还有一个重载的addLast()方法，如果不指定channel的名字而调用这个方法，就会自动生成一个名字，方法如下：
```
public ChannelPipeline addLast(ChannelHandler... handlers) {
    return addLast(null, handlers);
}
```
addLast()方法会调用generateName()方法生成一个名字。
```
private String generateName(ChannelHandler handler) {
    WeakHashMap<Class<?>, String> cache = nameCaches[(int) (Thread.currentThread().getId() % nameCaches.length)];
    Class<?> handlerType = handler.getClass();
    String name;
    synchronized (cache) {
        name = cache.get(handlerType);
        if (name == null) {
            name = generateName0(handlerType);
            cache.put(handlerType, name);
        }
    }

    synchronized (this) {
        // It's not very likely for a user to put more than one handler of the same type, but make sure to avoid
        // any name conflicts.  Note that we don't cache the names generated here.
        if (name2ctx.containsKey(name)) {
            String baseName = name.substring(0, name.length() - 1); // Strip the trailing '0'.
            for (int i = 1;; i ++) {
                String newName = baseName + i;
                if (!name2ctx.containsKey(newName)) {
                    name = newName;
                    break;
                }
            }
        }
    }

    return name;
}
```
generateName()方法又会调用generateName0()方法产生一个 handler 的名字。自动生成的名字的规则很简单, 就是 handler 的简单类名加上 "#0", 因此 EchoClientHandler 的名字就是 "EchoClientHandler#0"。
```
private static String generateName0(Class<?> handlerType) {
    return StringUtil.simpleClassName(handlerType) + "#0";
}
```

# Pipeline 的事件传输机制
根据前面的描述，可以知道，AbstractChannelHandlerContext 中有 inbound 和 outbound 两个 boolean 变量, 分别用于标识 Context 所对应的 handler 的类型, 即:

- inbound 为真时, 表示对应的 ChannelHandler 实现了 ChannelInboundHandler 方法.
- outbound 为真时, 表示对应的 ChannelHandler 实现了 ChannelOutboundHandler 方法.

 Netty 通过一个图示来解释这两个事件:
 ```
                                              I/O Request
                                        via Channel or
                                    ChannelHandlerContext
                                                  |
+---------------------------------------------------+---------------+
|                           ChannelPipeline         |               |
|                                                  \|/              |
|    +---------------------+            +-----------+----------+    |
|    | Inbound Handler  N  |            | Outbound Handler  1  |    |
|    +----------+----------+            +-----------+----------+    |
|              /|\                                  |               |
|               |                                  \|/              |
|    +----------+----------+            +-----------+----------+    |
|    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
|    +----------+----------+            +-----------+----------+    |
|              /|\                                  .               |
|               .                                   .               |
| ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
|        [ method call]                       [method call]         |
|               .                                   .               |
|               .                                  \|/              |
|    +----------+----------+            +-----------+----------+    |
|    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
|    +----------+----------+            +-----------+----------+    |
|              /|\                                  |               |
|               |                                  \|/              |
|    +----------+----------+            +-----------+----------+    |
|    | Inbound Handler  1  |            | Outbound Handler  M  |    |
|    +----------+----------+            +-----------+----------+    |
|              /|\                                  |               |
+---------------+-----------------------------------+---------------+
              |                                  \|/
+---------------+-----------------------------------+---------------+
|               |                                   |               |
|       [ Socket.read() ]                    [ Socket.write() ]     |
|                                                                   |
|  Netty Internal I/O Threads (Transport Implementation)            |
+-------------------------------------------------------------------+
 ```
 从上图可以看出, inbound 事件和 outbound 事件的流向是相反的。并且 inbound 的传递方式是通过调用相应的 ChannelHandlerContext.fireIN_EVT() 方法, 而 outbound 方法的的传递方式是通过调用 ChannelHandlerContext.OUT_EVT() 方法。
 
 例如 ChannelHandlerContext.fireChannelRegistered() 调用会发送一个 ChannelRegistered 的 inbound 给下一个ChannelHandlerContext, 而 ChannelHandlerContext.bind 调用会发送一个 bind 的 outbound 事件给 下一个 ChannelHandlerContext。
 
 Inbound 事件传播方法有:
```
ChannelHandlerContext.fireChannelRegistered()
ChannelHandlerContext.fireChannelActive()
ChannelHandlerContext.fireChannelRead(Object)
ChannelHandlerContext.fireChannelReadComplete()
ChannelHandlerContext.fireExceptionCaught(Throwable)
ChannelHandlerContext.fireUserEventTriggered(Object)
ChannelHandlerContext.fireChannelWritabilityChanged()
ChannelHandlerContext.fireChannelInactive()
ChannelHandlerContext.fireChannelUnregistered()
```
Oubound 事件传输方法有:
```
ChannelHandlerContext.bind(SocketAddress, ChannelPromise)
ChannelHandlerContext.connect(SocketAddress, SocketAddress, ChannelPromise)
ChannelHandlerContext.write(Object, ChannelPromise)
ChannelHandlerContext.flush()
ChannelHandlerContext.read()
ChannelHandlerContext.disconnect(ChannelPromise)
ChannelHandlerContext.close(ChannelPromise)
```
需要注意的是, 如果捕获了一个事件, 并且想让这个事件继续传递下去, 那么需要调用 Context 相应的传播方法。例如：
```
public class MyInboundHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        System.out.println("Connected!");
        ctx.fireChannelActive();
    }
}

public clas MyOutboundHandler extends ChannelOutboundHandlerAdapter {
    @Override
    public void close(ChannelHandlerContext ctx, ChannelPromise promise) {
        System.out.println("Closing ..");
        ctx.close(promise);
    }
}
```
上面的例子中, MyInboundHandler 收到了一个 channelActive 事件, 它在处理后, 如果希望将事件继续传播下去, 那么需要接着调用 ctx.fireChannelActive()。

## Outbound 操作(outbound operations of a channel)
**Outbound 事件都是请求事件(request event)**, 即请求某件事情的发生, 然后通过 Outbound 事件进行通知。Outbound 事件的传播方向是 tail -> customContext -> head。

接下来以 connect 事件为例, 分析一下 Outbound 事件的传播机制。
首先, 当用户调用了 Bootstrap.connect 方法时, 就会触发一个 Connect 请求事件, 此调用会触发如下调用链:
```
Bootstrap.connect -> 
    Bootstrap.doConnect -> 
        Bootstrap.doConnect0 -> 
            AbstractChannel.connect -> 
                DefaultChannelPipeline.connect()
```
DefaultChannelPipeline.connect()实现如下：
```
public ChannelFuture connect(SocketAddress remoteAddress, ChannelPromise promise) {
    return tail.connect(remoteAddress, promise);
}
```
可以看到, 当 outbound 事件(这里是 connect 事件)传递到 Pipeline 后, 它其实是以 tail 为起点开始传播的
而 tail.connect 其实调用的是 AbstractChannelHandlerContext.connect 方法:
```
public ChannelFuture connect(
        final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise promise) {
    ...
    final AbstractChannelHandlerContext next = findContextOutbound();
    EventExecutor executor = next.executor();
    ...
    next.invokeConnect(remoteAddress, localAddress, promise);
    ...
    return promise;
}
```
indContextOutbound()的作用是以当前 Context 为起点, 向 Pipeline 中的 Context 双向链表的前端寻找第一个 outbound 属性为真的 Context(即关联着 ChannelOutboundHandler 的 Context), 然后返回。
它的实现如下:
```
private AbstractChannelHandlerContext findContextOutbound() {
    AbstractChannelHandlerContext ctx = this;
    do {
        ctx = ctx.prev;
    } while (!ctx.outbound);
    return ctx;
}
```
当找到了一个 outbound 的 Context 后, 就调用它的 invokeConnect 方法, 这个方法中会调用 Context 所关联着的 ChannelHandler 的 connect 方法:
```
private void invokeConnect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise) {
    try {
        ((ChannelOutboundHandler) handler()).connect(this, remoteAddress, localAddress, promise);
    } catch (Throwable t) {
        notifyOutboundHandlerException(t, promise);
    }
}
```

如果用户没有重写 ChannelHandler 的 connect 方法, 那么会调用 ChannelOutboundHandlerAdapter 所实现的方法:
```
@Override
public void connect(ChannelHandlerContext ctx, SocketAddress remoteAddress,
        SocketAddress localAddress, ChannelPromise promise) throws Exception {
    ctx.connect(remoteAddress, localAddress, promise);
}
```
之后，会形成一个调用循环：
```
Context.connect -> Connect.findContextOutbound -> next.invokeConnect -> handler.connect -> Context.connect
```
这样的循环中, 直到 connect 事件传递到DefaultChannelPipeline 的双向链表的头节点, 即 head 中. head 实现了 ChannelOutboundHandler, 因此它的 outbound 属性是 true.

因为 head 本身既是一个 ChannelHandlerContext, 又实现了 ChannelOutboundHandler 接口, 因此当 connect 消息传递到 head 后, 会将消息转递到对应的 ChannelHandler 中处理, 而恰好, head 的 handler() 返回的就是 head 本身:
```
@Override
public ChannelHandler handler() {
    return this;
}
```
因此最终 connect 事件是在 head 中处理的. head 的 connect 事件处理方法如下:
```
@Override
public void connect(
        ChannelHandlerContext ctx,
        SocketAddress remoteAddress, SocketAddress localAddress,
        ChannelPromise promise) throws Exception {
    unsafe.connect(remoteAddress, localAddress, promise);
}
```
到这里, 整个 Connect 请求事件就结束了.下面以一幅图来描述一个整个 Connect 请求事件的处理过程:
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/channelpipeline_outbound.png">
</center>

## Inbound 事件
**Inbound 事件是一个通知事件, 即某件事已经发生了**, 然后通过 Inbound 事件进行通知. Inbound 通常发生在 Channel 的状态的改变或 IO 事件就绪.

Inbound 的特点是它传播方向是 head -> customContext -> tail.

接着分析Outbound事件 Connect 事件后会发生什么 Inbound 事件。当 Connect 这个 Outbound 传播到 unsafe 后, 其实是在 AbstractNioUnsafe.connect 方法中进行处理的:
```
@Override
public final void connect(
        final SocketAddress remoteAddress, final SocketAddress localAddress, final ChannelPromise promise) {
    ...
    if (doConnect(remoteAddress, localAddress)) {
        fulfillConnectPromise(promise, wasActive);
    } else {
        ...
    }
    ...
}
```
在 AbstractNioUnsafe.connect 中, 首先调用 doConnect 方法进行实际上的 Socket 连接, 当连接上后, 会调用 fulfillConnectPromise 方法:
```
private void fulfillConnectPromise(ChannelPromise promise, boolean wasActive) {
    ...
    // Regardless if the connection attempt was cancelled, channelActive() event should be triggered,
    // because what happened is what happened.
    if (!wasActive && isActive()) {
        pipeline().fireChannelActive();
    }
    ...
}
```
在 fulfillConnectPromise 中, 会通过调用 pipeline().fireChannelActive() 将通道激活的消息(即 Socket 连接成功)发送出去.

而这里, 当调用 pipeline.fireXXX 后, 就是 Inbound 事件的起点.因此当调用了 pipeline().fireChannelActive() 后, 就产生了一个 ChannelActive Inbound 事件, 可以从这里开始看看这个 Inbound 事件是怎么传播的。DefaultChannelPipeline.fireChannelActive()方法如下：
```
public ChannelPipeline fireChannelActive() {
    head.fireChannelActive();

    if (channel.config().isAutoRead()) {
        channel.read();
    }

    return this;
}
```
可以看到，在 fireChannelActive 方法中, 调用的是 head.fireChannelActive, 因此可以证明了, Inbound 事件在 Pipeline 中传输的起点是 head。继续往下看。
```
public ChannelHandlerContext fireChannelActive() {
    final AbstractChannelHandlerContext next = findContextInbound();
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeChannelActive();
    } else {
        executor.execute(new OneTimeTask() {
            @Override
            public void run() {
                next.invokeChannelActive();
            }
        });
    }
    return this;
}
```
这个地方就和Outbound事件类似了，

- 首先调用 findContextInbound, 从 Pipeline 的双向链表中中找到第一个属性 inbound 为真的 Context, 然后返回
- 调用这个 Context 的 invokeChannelActive

invokeChannelActive 方法如下:
```
private void invokeChannelActive() {
    try {
        ((ChannelInboundHandler) handler()).channelActive(this);
    } catch (Throwable t) {
        notifyHandlerException(t);
    }
}
```
这个方法和 Outbound 的对应方法(例如 invokeConnect) 如出一辙. 同 Outbound 一样, 如果用户没有重写 channelActive 方法, 那么会调用 ChannelInboundHandlerAdapter 的 channelActive 方法:
```
public void channelActive(ChannelHandlerContext ctx) throws Exception {
    ctx.fireChannelActive();
}
```
同样地, 在 ChannelInboundHandlerAdapter.channelActive 中，会产生事件调用循环：
```
Context.fireChannelActive -> Connect.findContextInbound -> nextContext.invokeChannelActive -> nextHandler.channelActive -> nextContext.fireChannelActive
```
同理, tail 本身 既实现了 ChannelInboundHandler 接口, 又实现了 ChannelHandlerContext 接口, 因此当 channelActive 消息传递到 tail 后, 会将消息转递到对应的 ChannelHandler 中处理, 而恰好, tail 的 handler() 返回的就是 tail 本身:
```
@Override
public ChannelHandler handler() {
    return this;
}
```
因此 channelActive Inbound 事件最终是在 tail 中处理的, 看一下它的处理方法:
```
@Override
public void channelActive(ChannelHandlerContext ctx) throws Exception { }
```
TailContext.channelActive 方法是空的. 如果查看 TailContext 的 Inbound 处理方法时, 会发现, 它们的实现都是空的. 可见, 如果是 Inbound, 当用户没有实现自定义的处理器时, 那么默认是不处理的.

用一幅图来总结一下 Inbound 的传输过程:
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/channelpipeline_inbound.png">
</center>

## 总结
对于 Outbound事件:

- Outbound 事件是请求事件(由 Connect 发起一个请求, 并最终由 unsafe 处理这个请求)
- Outbound 事件的发起者是 Channel
- Outbound 事件的处理者是 unsafe
- Outbound 事件在 Pipeline 中的传输方向是 tail -> head.
- 在 ChannelHandler 中处理事件时, 如果这个 Handler 不是最后一个 Hnalder, 则需要调用 ctx.xxx (例如 ctx.connect) 将此事件继续传播下去. 如果不这样做, 那么此事件的传播会提前终止.
- Outbound 事件流: Context.OUT_EVT -> Connect.findContextOutbound -> nextContext.invokeOUT_EVT -> nextHandler.OUT_EVT -> nextContext.OUT_EVT

对于 Inbound 事件:

- Inbound 事件是通知事件, 当某件事情已经就绪后, 通知上层.
- Inbound 事件发起者是 unsafe
- Inbound 事件的处理者是 Channel, 如果用户没有实现自定义的处理方法, 那么Inbound 事件默认的处理者是 TailContext, 并且其处理方法是空实现.
- Inbound 事件在 Pipeline 中传输方向是 head -> tail
- 在 ChannelHandler 中处理事件时, 如果这个 Handler 不是最后一个 Hnalder, 则需要调用 ctx.fireIN_EVT (例如 ctx.fireChannelActive) 将此事件继续传播下去. 如果不这样做, 那么此事件的传播会提前终止.
- Outbound 事件流: Context.fireIN_EVT -> Connect.findContextInbound -> nextContext.invokeIN_EVT -> nextHandler.IN_EVT -> nextContext.fireIN_EVT