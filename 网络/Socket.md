# Socket
---

# Socket是什么
Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/whatissocket.jpg" width="500">
</center>

# 缓存I/O
缓存 I/O 又被称作标准 I/O，大多数文件系统的默认 I/O 操作都是缓存 I/O。在 Linux 的缓存 I/O 机制中，操作系统会将 I/O 的数据缓存在文件系统的页缓存（ page cache ）中，也就是说，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。

**缓存 I/O 的缺点**：
数据在传输过程中需要在应用程序地址空间和内核进行多次数据拷贝操作，这些数据拷贝操作所带来的 CPU 以及内存开销是非常大的。

# I/O 模型
如上所说，对于一次IO访问（以read举例），数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。所以说，当一个read操作发生时，它会经历两个阶段：

- 等待数据准备 (Waiting for the data to be ready)
- 将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)

对于一个套接字上的输入操作，第一步通常涉及等待数据从网络中到达。当所等待数据到达时，它被复制到内核中的某个缓冲区。第二步就是把数据从内核缓冲区复制到应用进程缓冲区。

Unix 有五种 I/O 模型：

- 阻塞式 I/O
- 非阻塞式 I/O
- I/O 复用（select 和 poll）
- 信号驱动式 I/O（SIGIO）
- 异步 I/O（AIO）

## 阻塞式I/O
应用进程被阻塞，直到数据复制到应用进程缓冲区中才返回。在linux中，默认情况下所有的socket都是blocking。**blocking IO的特点就是在IO执行的两个阶段都被block了。**

应该注意到，在阻塞的过程中，其它程序还可以执行，因此阻塞不意味着整个操作系统都被阻塞。因为其他程序还可以执行，所以不消耗 CPU 时间，这种模型的 CPU 利用率效率会比较高。

下图中，recvfrom 用于接收 Socket 传来的数据，并复制到应用进程的缓冲区 buf 中。这里把 recvfrom() 当成系统调用。
```c
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
```
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/docs/notes/pics/1492928416812_4.png" width="500">
</center>

## 非阻塞I/O
linux下，可以通过设置socket使其变为non-blocking。nonblocking IO的特点是用户进程需要不断的主动询问kernel数据好了没有。

当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。用户进程判断结果是一个error时，它就知道数据还没有准备好，它可以继续执行，但是需要不断的执行系统调用来获知 I/O 是否完成，这种方式称为**轮询（polling）**。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。

由于 CPU 要处理更多的系统调用，因此这种模型的 CPU 利用率是比较低的。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/docs/notes/pics/1492929000361_5.png" width="500">
</center>

## I/O多路复用
IO multiplexing就是我们说的select，poll，epoll，有些地方也称这种IO方式为**event driven IO(事件驱动IO)**。select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select，poll，epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。

果一个 Web 服务器没有 I/O 复用，那么每一个 Socket 连接都需要创建一个线程去处理。如果同时有几万个连接，那么就需要创建相同数量的线程。相比于多进程和多线程技术，I/O 复用不需要进程线程创建和切换的开销，系统开销更小。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/docs/notes/pics/1492929444818_6.png" width="500">
</center>

## 信号驱动 I/O
应用进程使用 sigaction 系统调用，内核立即返回，应用进程可以继续执行，也就是说**等待数据阶段应用进程是非阻塞的**。内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 recvfrom 将数据从内核复制到应用进程中。

相比于非阻塞式 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/docs/notes/pics/1492929553651_7.png" width="500">
</center>

## 异步 I/O
用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，**当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了**。

异步 I/O 与信号驱动 I/O 的区别在于，异步 I/O 的信号是通知应用进程 I/O 完成，而信号驱动 I/O 的信号是通知应用进程可以开始 I/O。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/docs/notes/pics/1492930243286_8.png" width="500">
</center>

## 五大 I/O 模型比较
- 同步 I/O：将数据从内核缓冲区复制到应用进程缓冲区的阶段，应用进程会阻塞。
- 异步 I/O：不会阻塞。

阻塞式 I/O、非阻塞式 I/O、I/O 复用和信号驱动 I/O 都是同步 I/O，它们的主要区别在第一个阶段。

非阻塞式 I/O 、信号驱动 I/O 和异步 I/O 在第一阶段不会阻塞。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/docs/notes/pics/1492928105791_3.png" width="500">
</center>

# I/O复用
I/O多路复用的优势并不是对于单个连接能处理的更快，而是在于可以在单个线程/进程中处理更多的连接。

与多进程和多线程技术相比，I/O多路复用技术的最大优势是系统开销小，系统不必创建进程/线程，也不必维护这些进程/线程，从而大大减小了系统的开销。

select/poll/epoll 都是 I/O 多路复用的具体实现，select 出现的最早，之后是 poll，再是 epoll。

select是不断轮询去监听的socket，socket个数有限制，一般为1024个；

poll还是采用轮询方式监听，只不过没有个数限制；

epoll并不是采用轮询方式去监听了，而是当socket有变化时通过回调的方式主动告知用户进程。

# 参考
[Socket.md](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/Socket.md)
[Linux IO模式及 select、poll、epoll详解](https://segmentfault.com/a/1190000003063859)
[进程切换](http://guojing.me/linux-kernel-architecture/posts/process-switch/)
[IO多路复用](https://blog.csdn.net/chewbee/article/details/78108223)
[漫谈五种IO模型（主讲IO多路复用）](https://www.jianshu.com/p/6a6845464770)
[IO模式和IO多路复用](https://www.cnblogs.com/zingp/p/6863170.html)