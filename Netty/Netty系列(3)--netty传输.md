# Netty系列(3)--netty传输
---
# 前言
数据在网络中是以**字节**的形式传输的，这些字节如何流动取决于网络传输--一个将底层数据传输机制进行抽象的概念。

Netty提供了诸多的传输方式，而且提供了通用API供用户调用。

# Netty传输API
传输API的核心是interfaceChannel，它被用于所有的I/O操作。Channel类的层次结构如图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/netty-transport.PNG">
</center>

如图所示，每个Channel都将被分配一个ChannelPipeline和ChannelConfig。ChannelConfig包含了该Channel的所有配置设置，并且支持热更新。

ChannelPipeline持有所有将应用于入站和出站数据以及事件的ChannelHandler实例，这些ChannelHandler实现了应用程序用于处理状态变化以及数据处理的逻辑。

ChannelHandler的典型用途包括：

- 将数据从一种格式转为另外一种格式
- 提供异常通知
- 提供Channel变为活动或非活动的通知
- 提供当Channel注册到EventLoop或者从EventLoop注销时通知
- 提供有关用户自定义事件的通知

Netty 所提供的广泛功能只依赖于少量的接口。这意味着，可以对应用程序逻辑进行重大的修改，而无需大规模的重构代码库。

# Netty内置的传输
Netty能够提供的传输包含以下几个：

|名称	|描述|使用场景|
|-|-|-|
|NIO|	使用Java.nio.channels包作为基础—————基于选择器的方式|非阻塞代码库或一个常规的起点|
|Epoll|	由JNI驱动的epoll()和非阻塞IO。这个传输支持只有在Linux上可用的多种特性。如SO_Reuseport，比NIO传输更快且完全非阻塞的|非阻塞代码库或一个常规的起点(在Linux上)|
|OIO|使用java.net包作为基础—————使用阻塞流|阻塞代码库|
|Local|可以在VM内部通过管道进行通信的文本传输|同一个JVM内部通信	|
|Embedded|Embedded传输，允许使用ChannelHandler而又不需要真正的基于网络的传输。这在测试ChannelHandler时很有用|测试ChannelHandler	|

## NIO--非阻塞I/O
NIO的使用
```java
EventLoopGroup goup = new NioEventLoopGroup();
...
new ServerBootstrap().channel(NioServerSockerChannel.class) //服务端

new Bootstrap().channel(NioSockerChannel.class) //客户端
```
NIO提供了一个所有I/O操作的全异步实现。选择器背后的概念相当于一个“注册表”，在那里你可以请求在Channel的状态发生变化时得到通知。可能的状态有：

- 新的Channel已被接受并且就绪
- Channel连接已经完成
- Channel有已经就绪的可供读取的数据
- Channel可用于写数据

状态变化集的定义如下：

|名称|描述|
|-|-|
|OP_ACCEPT|	请求在接受新连接并创建Channel时获得通知|
|OP_CONNECT|	请求在建立一个连接时获得通知|
|OP_READ|	请求当数据已就绪 ，可以在Channel中读取数据时获得通知|
|OP_WRITE|	请求当可以向Channel中写更多数据时获得通知。这处理了套接字缓冲区被完全填满时的情况，该情况通知发生在数据的发送速度比远程节点可处理的速度更快时|

选择并处理状态变化集的过程如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/netty-selector.PNG">
</center>

## Epoll————用于LINUX的本地非阻塞传输
Netty为Linux提供了一组NIO API，其以一种和它本身的设计更加一致的方法使用epoll，并且以一种更加轻量的方式使用中断。如果应用程序指在运行于linux系统，应首先利用这个传输；在高负载下它的性能要优于JDK的NIO实现。

epoll的使用：
```java
...
EventLoopGroup group = new EpollEventLoopGroup();
...
new ServerBootstrap().channel(EpollServerSocketChannel.class) //服务端

new Bootstrap().channel(EpollSocketChannel.class) //客户端
...
```

## OIO--旧的阻塞I/O
Netty是如何能够使用和用于异步传输相同的API来支持OIO的呢？

在NIO中，一个 EventLoop 对应一个线程，一个Channel 绑定一个 EventLoop，而一个EventLoop 可以绑定多个Channel 来实现异步，也就是说一个线程可以处理多个 Channel。而OIO中，一个 EventLoop 仅绑定一个 Channel，也就是说每个线程只处理一个Channel ，这就有点像传统IO中，在服务端（ServerSocket）写了一个多线程来处理客户端的并发请求。

现在还有一个问题，channel是双向的，既可以读，也可以写。而stream是单向的，OIO中利用 InputStream 来读，OutputStream 来写。那么Channel 是如何实现阻塞的读和写的呢？答案就是， Netty利用了**SO_TIMEOUT**这个Socket标志，**它指定了等待一个I/O操作完成的最大毫秒数,I/O 操作期间Channel是阻塞的，如果操作在指定的时间间隔内没有完成，则将会抛出一个SocketTimeout Exception。 Netty将捕获这个异常并继续处理循环。在EventLoop下一次运行时，它将再次尝试。**这实际上也是类似于Netty这样的异步框架能够支持OIO的唯一方式。

OIO的使用
```java
...
EventLoopGroup group = new OioEventLoopGroup();
...
new ServerBootstrap().channel(OioServerSocketChannel.class)  //服务端

new Bootstrap().channel(OioSocketChannel.class)  //客户端
...
```

OIO的处理逻辑如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/netty-oio.PNG">
</center>

##Local —— 用于 JVM 内部通信的 Local 传输
Netty 提供了一个Local传输， 用于在同一个 JVM 中运行的客户端和服务器程序之间的异步通信。

 在这个传输中，和服务器 Channel 相关联的 SocketAddress 并没有绑定物理网络地址；相反，只要服务器还在运行， 它就会被存储在注册表里，并在 Channel 关闭时注销。 因为这个传输并不接受真正的网络流量，所以它并不能够和其他传输实现进行互操作。因此，客户端希望连接到（在同一个 JVM 中）使用了这个传输的服务器端时也必须使用它。
 
Local的使用
```java
...
EventLoopGroup group = new DefaultEventLoop();
...
new ServerBootstrap().channel(LocalServerChannel.class) //服务端

new Bootstrap().channel(LocalChannel.class) //客户端
...
```

## Embedded
Netty 提供了一种额外的传输， 使得你可以将一组 ChannelHandler 作为帮助器类嵌入到其他的 ChannelHandler 内部。 通过这种方式，你将可以扩展一个 ChannelHandler 的功能，而又不需要修改其内部代码。

Embedded 传输的关键是一个被称为 EmbeddedChannel 的具体的Channel实现。

如果你想要为自己的 ChannelHandler 实现编写单元测试， 那么请考虑使用 Embedded 传输。

# 参考
《Netty 实战》

[《Netty实战》--传输](https://www.jianshu.com/p/9581068f9739)

[Netty 系列二（传输）.](https://www.cnblogs.com/jmcui/p/9171733.html)