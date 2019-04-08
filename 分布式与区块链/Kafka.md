# Kafka
---

# kafka相关概念
**Producer：**
生产者。提供数据源生产的地方，对于同一个topic，生产者只能有一个，这样可以确保同一个topic数据来自同一个业务数据，支持多并发。但是同一个生产者可能向多个topic里生产数据。

 **Consumer：**
 消费者。消费数据的客户端，对于同一个topic，可以有多个消费者，比如spark，storm等等
 
 **Broker**
消息中间件处理结点，一个Kafka节点就是一个broker，多个broker可以组成一个Kafka集群。

**Topic**
同一类消息的统称，Kafka集群能够同时负载多个topic分发。

**Partition**
topic物理上的分组，一个topic可以分为多个partition，每个partition是一个有序的队列，同一个topic里面的数据是存放在不同的分区中。

**Replication**
每个分区或者topic都是有副本的，副本的数量也是可以在创建topic的时候就指定好，保证数据的安全性，以及提供高并发读取效率。

**Segment**
partition物理上由多个segment组成。

**Offset**
每个partition都由一系列有序的、不可变的消息组成，这些消息被连续的追加到partition中。partition中的每个消息都有一个连续的序列号叫做offset，用于partition唯一标识一条消息

**Leader和Follower**
同一个topic的同一partion可能在多个broker上存在Replication。对于同一个partition，它所在任何一个broker，都有能扮演两种角色：leader、follower。leader需要处理读写请求，follower只复制leader

**ISR**
Kafka会动态维护一个与Leader保持一致的同步副本（in-sync replicas （ISR））集合，并且会将最新的同步副本（ISR ）集合持久化到zookeeper。如果leader出现问题了，就会从该partition的followers中选举一个作为新的leader。

kafka的简单框架图如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/kafka.jpg">
`topic1 : 4个partition，每个partition复制3份`
`tipic2 : 3个partition，每个partition复制2份`
`topic3 : 4个partition，每个partition复制4份`
</center>

上图中，红色的代表此partition所在的broker是该partition组的leader。

**对于topic1的4个partition:**

- `Part1`的`leader`是`broker1`，`followers`是`broker2\3`
- `Part2`的`leader`是`broker2`，`followers`是`broker1\4`
- `Part3`的`leader`是`broker3`，`followers`是`broker1\3`
- `Part4`的`leader`是`broker4`，`followers`是`broker2\3`

**对于topic2的3个partition：**

- `Part1`的`leader`是`broker1`，`followers`是`broker2`
- `Part2`的`leader`是`broker2`，`followers`是`broker3`
- `Part3`的`leader`是`broker3`，`followers`是`broker4`

**对于topic2的4个partition：**

- `Part1`的`leader`是`broker4`，`followers`是`broker1\2\3`
- `Part2`的`leader`是`broker2`，`followers`是`broker1\3\4`
- `Part3`的`leader`是`broker3`，`followers`是`broker1\2\4`
- `Part4`的`leader`是`broker1`，`followers`是`broker2\3\4`

---

# kafka消息传递保障
消息传递保证语义通常有以下三种级别：

- `At most once`：最多一次。消息可能丢失，但绝不会重发。
- `At least once`：至少一次。消息绝不会丢失，但有可能重新发送。
- `Exactly once`：正好一次。这是人们真正想要的，每个消息传递一次且仅一次。

这三种语义在kafka中都有体现。在生产者端的体现主要是消息是否重发，在消费者端体现主要是消息是否丢失。

在生产者方面，生产者发出的消息被`commit`到日志之后会给生产发送响应，如果生产者没有收到响应，就会重发消息，这就是**生产者端“至少一次”**的体现。在`0.11.0.0`之前，如果原始请求实际上已成功，则在重新发送期间再次将消息写入到日志中其实会产生重复消息。但是在`0.11.0.0`之后，生产者支持幂等传递选项，保证重新发送不会导致日志中重复。 `broker`为每个生产者分配一个`ID`，并通过生产者发送的序列号为每个消息进行去重。

在消费者方面，每个消费者都会维护自己的消费信息和`offset`，如果某一个消费者崩溃了，会有另一个消费者接着消费消息，它需要从一个合适的offset继续处理。这种情况下可以有以下选择：

- **consumer可以先读取消息，然后将offset写入日志文件中，然后再处理消息**。这存在一种可能就是在存储offset后还没处理消息就crash了，新的consumer继续从这个offset处理，那么就会有些消息永远不会被处理，这就是“**最多一次**”。
- **consumer可以先读取消息，处理消息，最后记录offset**，当然如果在记录offset之前就crash了，新的consumer会重复的消费一些消息，这就是“**最少一次**”。
- “精确一次”可以通过将提交分为两个阶段来解决：保存了offset后提交一次，消息处理成功之后再提交一次。但是还有个更简单的做法：将消息的offset和消息被处理后的结果保存在一起。比如用Hadoop ETL处理消息时，将处理后的结果和offset同时保存在HDFS中，这样就能保证消息和offser同时被处理了。

kafka默认是保证“**至少一次**”传递，并允许用户通过禁止生产者重试和处理一批消息前提交它的偏移量来实现 “**最多一次**”传递。而“**正好一次**”传递需要与目标存储系统合作，但kafka提供了偏移量，所以实现这个很简单。

# 参考
[kafka中文教程](http://orchome.com/kafka/index)
[kafka 学习 非常详细的经典教程](https://blog.csdn.net/tangdong3415/article/details/53432166/)
[Apache Kafka核心概念-多图-形象易懂（入门教程轻松学）](https://blog.csdn.net/liyiming2017/article/details/82805479)
[Kafka](https://www.jianshu.com/p/c1d6725ebf86)
[Kafka：架构简介【转】](https://www.cnblogs.com/seaspring/p/6138080.html)
[Kafka框架基础](https://www.jianshu.com/p/a24af7a86392)
[PacificA：微软设计的分布式存储框架](https://www.colabug.com/225369.html)
[kafka：leader选举（broker /分区）](https://blog.csdn.net/weixin_38750084/article/details/83053936)

---

# leader选举（broker /分区）
## broker的leader
Kafka的Leader选举是通过在zookeeper上创建/controller临时节点来实现leader选举，并在该节点中写入当前broker的信息。这个broker的leader也叫KafkaController。它除了具有一般broker的功能外，还负责分区leader的选取，也就是负责选举partition的leader replica。

开始时broker从zookeeper获取/controller临时节点信息，如果还没有选举出leader，那么此节点是不存在的，返回-1。如果返回的不是-1，而是leader的json数据，那么说明已经有leader存在，选举结束。

如果broker检测到当前没有leader，会触发向临时节点/controller写入自己的信息。最先写入的就会成为leader。

有三种情况触发控制器选举：

- 集群启动
- 控制器所在代理发生故障
- zookeeper心跳感知，控制器与自己的session过期

## 分区的leader
首先Kafka会将接收到的消息分区（partition），每个主题（topic）的消息有不同的分区。这样一方面消息的存储就不会受到单一服务器存储空间大小的限制，另一方面消息的处理也可以在多个服务器上并行。

其次为了保证高可用，每个分区都会有一定数量的副本（replica）。这样如果有部分服务器不可用，副本所在的服务器就会接替上来，保证应用的持续性。

**但是，为了保证较高的处理效率，消息的读写都是在固定的一个副本上完成。这个副本就是所谓的Leader，而其他副本则是Follower。而Follower则会定期地到Leader上同步数据。**

### Leader选举
如果某个分区所在的服务器除了问题，不可用，kafka会从该分区的其他的副本中选择一个作为新的Leader。之后所有的读写就会转移到这个新的Leader上。现在的问题是应当选择哪个作为新的Leader。显然，只有那些跟Leader保持同步的Follower才应该被选作新的Leader。

Kafka会在Zookeeper上针对每个Topic维护一个称为ISR（in-sync replica，已同步的副本）的集合，该集合中是一些分区的副本。只有当这些副本都跟Leader中的副本同步了之后，kafka才会认为消息已提交，并反馈给消息的生产者。如果这个集合有增减，kafka会更新zookeeper上的记录。

如果某个分区的Leader不可用，Kafka就会从ISR集合中选择一个副本作为新的Leader。这其实是**微软PacificA**的一种实现。

显然通过ISR，kafka需要的冗余度较低，可以容忍的失败数比较高。假设某个topic有f+1个副本，kafka可以容忍f个服务器不可用。

### 为什么不用少数服从多数的方法
少数服从多数是一种比较常见的一致性算法和Leader选举法。

它的含义是只有超过半数的副本同步了，系统才会认为数据已同步；

选择Leader时也是从超过半数的同步的副本中选择。

这种算法需要较高的冗余度。

譬如只允许一台机器失败，需要有三个副本；而如果只容忍两台机器失败，则需要五个副本。

而kafka的ISR集合方法，分别只需要两个和三个副本。

**如果所有的ISR副本都失败了怎么办？**
此时有两种方法可选，一种是等待ISR集合中的副本复活，一种是选择任何一个立即可用的副本，而这个副本不一定是在ISR集合中。这两种方法各有利弊，实际生产中按需选择。

如果要等待ISR副本复活，虽然可以保证一致性，但可能需要很长时间。而如果选择立即可用的副本，则很可能该副本并不一致。

参考：
[kafka：leader选举（broker /分区）](https://blog.csdn.net/weixin_38750084/article/details/83053936)
[PacificA：微软设计的分布式存储框架](https://www.colabug.com/225369.html)
[Apache Kafka 核心组件和流程-控制器-设计-原理（入门教程轻松学）](https://blog.csdn.net/liyiming2017/article/details/82843036)

---

