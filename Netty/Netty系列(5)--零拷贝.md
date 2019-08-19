# Netty系列(5)--零拷贝
---
# 简述
Netty中的零拷贝包括两种：**OS级别**的和**应用程序级别**的。

Java NIO中的FileChannel.transferTo()方法都实现了零拷贝的功能，在Netty中也通过在FileRegion中包装了NIO的FileChannel.transferTo()方法实现了零拷贝。所以，**OS级别的零拷贝是通过transferTo()实现的。**

**应用程序级别的零拷贝主要是通过CompositeByteBuf等实现的。**

# OS级别的零拷贝
Zero-copy, 就是在操作数据时, 不需要将数据 从一个内存区域拷贝到另一个内存区域. 因为少了一次内存的拷贝, 因此 CPU 的效率就得到的提升。

在 OS 层面上的 Zero-copy 通常指避免在 用户态(User-space) 与 内核态(Kernel-space) 之间来回拷贝数据. 例如 Linux 提供的 mmap 系统调用, 它可以将一段用户空间内存映射到内核空间, 当映射成功后, 用户对这段内存区域的修改可以直接反映到内核空间; 同样地, 内核空间对这段区域的修改也直接反映用户空间. 正因为有这样的映射关系, 我们就不需要在 用户态(User-space) 与 内核态(Kernel-space) 之间拷贝数据, 提高了数据传输的效率.

举一个例子，假如要将一个磁盘中的文件通过Socket发送，使用的伪代码如下：
```java
File.read(bytes)
Socket.send(bytes)
```

如果不适用零拷贝，需要四次数据拷贝和四次上下文切换：

- 数据从磁盘读取到内核的read buffer
- 数据从内核缓冲区拷贝到用户缓冲区
- 数据从用户缓冲区拷贝到内核的socket buffer
- 数据从内核的socket buffer拷贝到网卡接口的缓冲区

如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/zero-copy1.jpg">
</center>

可以看到，2操作和3操作的两次复制其实是没有必要的。如果使用FileChannel.transferTo方法，可以避免上述的两个多余操作。如下图所示

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/zero-copy2.jpg">
</center>

这样，就避免了在内核和应用程序之间不必要的复制操作。

# 应用级别的零拷贝
应用界别的Zero-copy 体现在如下几个个方面:

- Netty 提供了 CompositeByteBuf 类, 它可以将多个 ByteBuf 合并为一个逻辑上的 ByteBuf, 避免了各个 ByteBuf 之间的拷贝.
- 通过 wrap 操作, 我们可以将 byte[] 数组、ByteBuf、ByteBuffer等包装成一个 Netty ByteBuf 对象, 进而避免了拷贝操作.
- ByteBuf 支持 slice 操作, 因此可以将 ByteBuf 分解为多个共享同一个存储区域的 ByteBuf, 避免了内存的拷贝.

## 通过 CompositeByteBuf 实现零拷贝
假设我们有一份协议数据, 它由头部和消息体组成, 而头部和消息体是分别存放在两个 ByteBuf 中的, 即:
```java
ByteBuf header = ...
ByteBuf body = ...
```
我们在代码处理中, 通常希望将 header 和 body 合并为一个 ByteBuf, 方便处理, 那么通常的做法是:
```java
ByteBuf allBuf = Unpooled.buffer(header.readableBytes() + body.readableBytes());
allBuf.writeBytes(header);
allBuf.writeBytes(body);
````
可以看到, 我们将 header 和 body 都拷贝到了新的 allBuf 中了, 这无形中增加了两次额外的数据拷贝操作了.

那么有没有更加高效优雅的方式实现相同的目的呢? 我们来看一下 CompositeByteBuf 是如何实现这样的需求的吧.
```java
ByteBuf header = ...
ByteBuf body = ...

CompositeByteBuf compositeByteBuf = Unpooled.compositeBuffer();
compositeByteBuf.addComponents(true, header, body);
```
上面代码中, 我们定义了一个 CompositeByteBuf 对象, 然后调用
```java
public CompositeByteBuf addComponents(boolean increaseWriterIndex, ByteBuf... buffers) {
...
}
```
方法将 header 与 body 合并为一个**逻辑上**的 ByteBuf。不过在 CompositeByteBuf 内部, 这两个 ByteBuf 都是单独存在的。


待续......

# 参考
[对于 Netty ByteBuf 的零拷贝(Zero Copy)的理解](https://segmentfault.com/a/1190000007560884)</br>
[Netty中的零拷贝](https://www.jianshu.com/p/a199ca28e80d)</br>
[理解Netty中的零拷贝（Zero-Copy）机制](https://my.oschina.net/plucury/blog/192577)</br>
[netty学习十三:零拷贝底层实现原理](https://blog.csdn.net/linsongbin1/article/details/77650105)</br>
[Java-NIO（三）：直接缓冲区与非直接缓冲区](https://www.cnblogs.com/yy3b2007com/archive/2017/07/31/7262453.html)</br>
[Netty 系列之 Netty 高性能之道](https://www.infoq.cn/article/netty-high-performance?utm_source=infoq&utm_medium=popular_links...)</br>
[Netty in action—Netty中的ByteBuf](https://blog.csdn.net/yjw123456/article/details/77843931)</br>