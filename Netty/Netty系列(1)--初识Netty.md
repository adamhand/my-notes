﻿---
# Netty系列(1)--初识Netty
---

# Netty是什么
Netty 是一款**异步**的**事件驱动**的**网络应用程序框架**，支持快速地开发可维护的高性能的面向协议的服务器和客户端。 

## 异步和同步
**同步（Sync）**：所谓同步，就是线程发出一个功能调用时，在没有得到结果之前，该调用就不返回或继续执行后续操作，而是等待可以调用。当可以调用之后，线程**自行进行调用**。

**异步（Async）**：异步与同步相对，当一个异步过程调用发出后，调用者在没有得到结果之前，就可以继续执行后续操作。当这个**调用完成**后，一般通过状态、通知和回调来通知调用者。对于异步调用，调用的返回并不受调用者控制。

总结来说，同步和异步的区别大概有两个：

- 请求发出后，是否**可以返回**执行其他操作。
- 当可以进行调用时，**同步情况是发起调用的线程自己去调用并得到结果，异步情况是内核将调用动作完成，将结果返回给调用线程**。

## 事件驱动
一个典型的事件驱动的程序，就是一个死循环，并以一个线程的形式存在，这个死循环包括两个部分，第一个部分是按照一定的条件接收并选择一个要处理的事件，第二个部分就是事件的处理过程。程序的执行过程就是选择事件和处理事件，而当没有任何事件触发时，程序会因查询事件队列失败而进入睡眠状态，从而释放cpu。

也就是说，事件驱动就是有事件就处理，没事件就等待。

# BIO、NIO和AIO的区别
## 事件分离器

在IO读写时，把 IO请求 与 读写操作 分离调配进行，需要用到事件分离器。根据处理机制的不同，事件分离器又分为：同步的Reactor和异步的Proactor。

**Reactor模型：**

- 应用程序在事件分离器注册 读就绪事件 和 读就绪事件处理器
- 事件分离器等待读就绪事件发生
- 读就绪事件发生，激活事件分离器，分离器调用 读就绪事件处理器（即：可以进行读操作了，开始读）
- 读事件处理器开始进行读操作，把读到的数据提供给程序使用

**Proactor模型：**

- 应用程序在事件分离器注册 读完成事件 和读完成事件处理器，并向操作系统发出异步读请求
- 事件分离器等待操作系统完成读取
- 在分离器等待过程中，操作系统利用并行的内核线程执行实际的读操作，并将结果数据存入用户自定义缓冲区，最后通知事件分离器读操作完成
- 事件分离器监听到 读完成事件 后，激活 读完成事件的处理器
- 读完成事件处理器 处理用户自定义缓冲区中的数据给应用程序使用

同步和异步的区别就在于 **读** 操作由谁完成：同步的Reactor是指程序发出读请求后，由分离器监听到可以进行读操作时（需要获得读操作条件）通知事件处理器进行读操作，异步的Proactor是指程序发出读请求后，操作系统立刻异步地进行读操作了，读完之后在通知分离器，分离器激活处理器直接取用已读到的数据。

**同步阻塞IO（BIO）：**
在此种方式下，用户进程在发起一个IO操作以后，必须等待IO操作的完成，只有当真正完成了IO操作以后，用户进程才能运行。JAVA传统的IO模型属于此种方式！

**同步非阻塞IO（NIO）:**
在此种方式下，用户进程发起一个IO操作以后边可返回做其它事情，但是用户进程需要时不时的询问IO操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的CPU资源浪费。其中目前JAVA的NIO就属于同步非阻塞IO。

**异步阻塞IO（AIO）：**
此种方式下是指应用发起一个IO操作以后，不等待内核IO操作的完成，等内核完成IO操作以后会通知应用程序，这其实就是同步和异步最关键的区别，同步必须等待或者主动的去询问IO是否完成，那么为什么说是阻塞的呢？因为此时是通过select系统调用来完成的，而select函数本身的实现方式是阻塞的，而采用select函数有个好处就是它可以同时监听多个文件句柄，从而提高系统的并发性！

**异步非阻塞IO:**
在此种模式下，用户进程只需要发起一个IO操作然后立即返回，等IO操作真正的完成以后，应用程序会得到IO操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的IO读写操作，因为真正的IO读取或者写入操作已经由内核完成了。目前Java中还没有支持此种IO模型。

# Netty相关概念和基本架构
Netty的核心组件包括以下几个部分：

- BootStrap和ServerBootstrap 
BootStrap通常称为引导类，提供一个用于应用程序网络层配置的容器。

- Channel 
底层网路传输API必须提供给应用I/O操作的接口，如读、写、连接、绑定等。它结构类似一个“Socket”。它有很多类似于socket的函数：bind、close、config、connect、isActive、isOpen、isWritable、read、write等等。

- ChannelHandler 
Handle称之为处理器，支持很多协议，提供用于数据处理的容器。 
常用的一个接口是ChannelInboundHandler，这个类型可以处理入站事件(即外部应用连接到本应用的事件)； 
反之有ChannelOutboundHandler接口，处理出站事件。业务逻辑经常在一个或多个ChannelInboundHandler中操作。

- ChannelPipeline 
Netty的数据处理流程其实是一种责任链和拦截过滤器模式，ChannelPipeline 提供了一个链容器，该链包含一个或多个ChannelHandler ，并提供了一个API用于管理沿着链入站和出站事件的流动。

- EventLoop 
EventLoop 用于处理 Channel 的 I/O操作。一个EventLoop可以处理多个Channel事件。而EventLoopGroup是一个Group，可以包括多个EventLoop。

- ChannelFuture 
Netty是一种异步I/O模型，一个操作可能无法立即返回结果，所以它提供了ChannelFuture类，可以通过addListener添加监听器，操作完成时可以作出通知。

Netty的基本架构如下图所示:
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/netty.jpg">
</center>

# 补充
## netty是对nio的封装，nio是同步非阻塞，那么为什么netty又是异步的呢？
确实，netty是对nio的封装，如果按照nio的做法来，线程发起读数据请求之后不会立刻返回，而是通过selector不但询问cpu数据是否准备好。但是netty实现了一个mpsc(多生产者单消费者)队列，所有外部线程的任务都给扔到这个队列里，同时把回调，也就是future绑定在这个任务上，reactor线程会挨个执行这些任务，执行完之后callback。

## 为什么netty使用nio而不是aio

- Netty不看重Windows上的使用，在Linux系统上，AIO的底层实现仍使用EPOLL，没有很好实现AIO，因此在性能上没有明显的优势，而且被JDK封装了一层不容易深度优化。
- AIO还有个缺点是接收数据需要预先分配缓存, 而不是NIO那种需要接收时才需要分配缓存, 所以对连接数量非常大但流量小的情况, 内存浪费很多。


---
参考：

[【面试题】Netty相关](https://blog.csdn.net/baiye_xing/article/details/76735113)</br>
[为什么Netty受欢迎？](https://www.jianshu.com/p/b9f3f6a16911)</br>
[Netty入门教程2——动手搭建HttpServer](https://www.jianshu.com/p/ed0177a9b2e3)</br>
[BIO、NIO和AIO的区别（简明版）](https://www.cnblogs.com/ygj0930/p/6543960.html)</br>
[reactor和proactor模式](https://blog.csdn.net/caiwenfeng_for_23/article/details/8458299)</br>
[基本概念_同步、异步有什么区别](https://www.cnblogs.com/weiyi1314/p/6723913.html)</br>
[事件驱动机制跟消息驱动机制相比](http://www.cnblogs.com/welen/articles/5115213.html)</br>
[Netty学习笔记之三——认识Netty架构](https://blog.csdn.net/u012525096/article/details/79832927)</br>
[Netty4详解三：Netty架构设计](https://www.cnblogs.com/DaTouDaddy/p/6801906.html)</br>
[Netty整体架构](https://blog.csdn.net/u013857458/article/details/82527722)</br>
[netty为什么是异步的](https://coding.m.imooc.com/questiondetail.html?qid=100186)</br>
[Java NIO浅析](https://tech.meituan.com/2016/11/04/nio.html)</br>
[为什么Netty使用NIO而不是AIO？](https://www.jianshu.com/p/df1d6d8c3f9d)
[为什么Netty使用NIO而不是AIO？](https://www.jianshu.com/p/df1d6d8c3f9d)
---
