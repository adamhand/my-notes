# Netty系列(4)--ByteBuf
---
# ByteBuf类—Netty的数据容器
因为所有的网络通信都涉及到字节序列的移动，一个有效而易用的数据结构是非常必要的。Netty的ByteBuf实现达到并超过这些需求。

## ByteBuf的工作原理
ByteBuf维护两个不同的索引：读索引和写索引。当你从ByteBuf中读，它的readerIndex增加了读取的字节数；同理，当你向ByteBuf中写，writerIndex增加。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/bytebuf.PNG">
</center>

如果读数据时readerIndex已经和writerIndex一样说明可读数据已经被读完。尝试继续往下读会引发一个IndexOutOfBoundsException，就像引发数组越界那样。

ByteBuf中名称以read或write开头的方法会推进相应的索引，而以set或get开头的不会。 
可以指定ByteBuf最大的容量，默认是Integer.MAX_VALUE。

## 堆缓冲区(heap buffer)
最常使用的ByteBuf模式将数据保存到JVM的堆中。被称为**支持数组(backing array)**,这个模式提供了在没有使用池技术的情况下快速分配和释放(在堆缓冲区中)。这种方法是非常适合于来处理**遗留数据（是啥？？）**。

```java
ByteBuf heapBuf = ...;
if (heapBuf.hasArray()){//检查是否有支持数组。当hasArray()返回false时尝试访问支持数组会抛出UnsupportedOperationException
    byte[] array = heapBuf.array();     //得到支持数组
    int offset = heapBuf.arrayOffset() + heapBuf.readerIndex();//计算第一个字节的偏移量
    int length = heapBuf.readableBytes();//计算可读字节数
    handleArray(array, offset, length); //调用你的方法来处理这个array
}
```

## 直接缓冲区(direct buffer)
在JDK1.4中引入的NIO的ByteBuffer类允许JVM 通过本地(native)方法调用分配内存,其目的是通过免去中间交换的内存拷贝, 提升IO处理速度。Netty的直接缓冲区与此类似。直接缓冲区在本地分配存储空间（有待考证）。

虽然直接缓冲区免去了数据在内存中的拷贝，但是由于它在堆外建立存储空间，不受JVM垃圾回收机制的约束，分配和释放消耗更大。

```java
ByteBuf directBuf = ...
if (!directBuf.hasArray()) {//false表示为这是直接缓冲
    int length = directBuf.readableBytes();//得到可读字节数
    byte[] array = new byte[length];    //分配一个具有length大小的数组
    directBuf.getBytes(directBuf.readerIndex(), array); //将缓冲区中的数据拷贝到这个数组中 
    handleArray(array, 0, length); //下一步处理
}
```

## 复合缓冲区(composite buffer)
Netty 提供了 CompositeByteBuf 类, 它可以将多个 ByteBuf 合并为一个逻辑上的 ByteBuf, 避免了各个 ByteBuf 之间的拷贝。

现在有这样一种情况：一种消息由header和body两部分组成，这种消息的body都是相同的，也就是说body可以重用。这种情况下，只需要为每一条消息创建不同的header，body只需创建一个，然后将它们两个组合即可。

下面先介绍如何通过JDK的ByteBuffer来实现这个需求：
```java
//通过一个数组来存储这条消息
ByteBuffer [] message = new ByteBuffer[]{header,body};
//使用副本来合并这两个部分
ByteBuffer message2 =ByteBuffer.allocate(header.remaining() + body.remaining());
message2.put(header);
message2.put(body);
message2.flip()
```

下面介绍如何通过CompositeByteBuf来实现：
```java
CompositeByteBuf messageBuf = Unpooled.compositeBuffer();
ByteBuf headerBuf = ...; //直接缓冲或堆缓冲都可
ByteBuf bodyBuf = ...; // 直接缓冲或堆缓冲都可
messageBuf.addComponents(headerBuf, bodyBuf);//将ByteBuf实例添加到CompositeByteBuf中
.....
messageBuf.removeComponent(0); //删除header
for (ByteBuf buf : messageBuf) {//遍历messageBuf中的ByteBuf
    System.out.println(buf.toString());
}
```

CompositeByteBuf可能不允许访问支持数组，所以访问CompositeByteBuf中的数据的方式类似于直接缓冲区模式，如下所示：
```java
CompositeByteBuf compBuf = Unpooled.compositeBuffer();
int length = compBuf.readableBytes();//得到可读的字节数
byte[] array = new byte[length];//分配一个字节数组
compBuf.getBytes(compBuf.readerIndex(), array);//将数据读到这个字节数组中
handleArray(array, 0, array.length);
```

未完待续...