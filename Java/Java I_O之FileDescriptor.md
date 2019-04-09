# Java I/O之FileDescriptor

# 基本概念
FileDescriptor即“文件描述符”，可以用来表示开放文件、开放套接字。这个概念来自于Unix操作系统，Unix操作系统中“一切皆文件”，比如文件、目录、进程、网络socket、各种硬件设备等，都被看成文件；

在Windows下，FileDescriptor称之为“文件句柄”, 句柄是Windows下各种对象的标识符, 比如文件、资源、菜单、光标、位图等。

当应用程序请求打开或者操作文件时，操作系统会提供一个非负整数,作为一个索引号,它的作用就像地址或者说指针或者说偏移量，这个索引号就用来定位文件数据在内存中的位置。

但是只有FileDescriptor，我们是无法读写文件的，我们还需要FileInputStream、FileOutputStream或RandomAccessFile等类来实现文件的读写。

# 标准I/O流
操作系统有三个标准I/O流。

|FileDescriptor|	名称	|POSIX常量标识(unistd.h)|	标准IO常量标识(stdio.h)|
|-|-|-|-|
|0	|标准输入流	|STDIN_FILENO|	stdin|
|1	|标准输出流	|STDOUT_FILENO|	stdout|
|2|	标准错误流	|STDERR_FILENO|	stderr|

FileDescriptor类中也定义了这三个常量：
```
public static final FileDescriptor in = standardStream(0);
public static final FileDescriptor out = standardStream(1);
public static final FileDescriptor err = standardStream(2);
```

参考：
[Channel & FileDescriptor](http://www.udpwork.com/item/5618.html)

[java NIO之FileChannel实现原理](https://blog.csdn.net/qq_26222859/article/details/80885757)

[[一]FileDescriptor文件描述符 标准输入输出错误 文件描述符](https://cloud.tencent.com/developer/article/1333548)

[JavaIO流复习与巩固--FileDescriptor与File](https://blog.csdn.net/Holmofy/article/details/75269866)

[Java中的Java.io.FileDescriptor](http://www.breakyizhan.com/java/4303.html)

[Java NIO3：通道和文件通道](https://www.cnblogs.com/szlbm/p/5513155.html)
