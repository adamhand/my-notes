# kafka在windows下的安装

## 安装环境

- jdk1.8
- zookeeper
- kafka_2.12-2.3.0

由于新版的kafka已经内置了zookeeper，所以不用再另外安装。jdk的安装也不再赘述。kafka的下载地址在[这里]( http://kafka.apache.org/downloads.html)，选择Scala 2.12版本的二进制包。

## zookeeper配置
将安装包下载下来之后进行解压，放在合适的目录，这里将其放在J:\ProfessionalSoftware\kafka_2.12-2.3.0。之后进行zookeeper的配置。

打开J:\ProfessionalSoftware\kafka_2.12-2.3.0\config中的zookeeper.properties文件，进行如下配置：

```
dataDir=J:\\ProfessionalSoftware\\kafka_2.12-2.3.0\\zk-logs

dataLogDir=J:\\ProfessionalSoftware\\kafka_2.12-2.3.0\\zk-logs

clientPort=2181

# maxClientCnxns=0

# 分别为：
# zk的基本时间单元，毫秒
# Leader-Follower初始通信时限 tickTime*10
# Leader-Follower同步通信时限 tickTime*5

tickTime=2000
initLimit=10
syncLimit=5

#设置broker Id的服务地址
server.0=localhost:2888:3888
```
之后在数据目录(J:\\ProfessionalSoftware\\kafka_2.12-2.3.0\\zk-logs)添加myid文件，下入服务broker.id的值，该值与zookeeper.properties中设置的server的id一致，这里是0。

## kafka配置
进入config目录下，修改server.properties文件。

```
# broker 的全局唯一编号，不能重复
broker.id=0

# 配置监听,修改为本机ip
advertised.listeners=PLAINTEXT://localhost:9092   

# 处理网络请求的线程数量，默认
num.network.threads=3

# 用来处理磁盘IO的线程数量，默认
num.io.threads=8

# 发送套接字的缓冲区大小，默认
socket.send.buffer.bytes=102400

# 接收套接字的缓冲区大小，默认
socket.receive.buffer.bytes=102400

# 请求套接字的缓冲区大小，默认
socket.request.max.bytes=104857600


zookeeper.connect=localhost:2181

# kafka 运行日志存放路径
log.dirs=J:\\ProfessionalSoftware\\kafka_2.12-2.3.0\\logs

# topic 在当前broker上的分片个数，与broker数量保持一致
num.partitions=1

# 用来恢复和清理data下数据的线程数量，默认
num.recovery.threads.per.data.dir=1

# segment文件保留的最长时间，超时将被删除，默认
log.retention.hours=168

# 滚动生成新的segment文件的最大时间，默认
log.roll.hours=168
```
到这里，配置就完成了。

## 启动kafka
打开一个cmd窗口，进入kafka的安装目录J:\ProfessionalSoftware\kafka_2.12-2.3.0，执行下面的命令启动zookeeper。

```
.\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
```
保持该窗口不关闭，打开另一个cmd，依旧进入J:\ProfessionalSoftware\kafka_2.12-2.3.0目录，执行以下命令启动kafka。

```
.\bin\windows\kafka-server-start.bat .\config\server.properties
```

这时，kafka就已经启动完成。然后创建一个名为test的topic和一个生产者一个消费者，进行测试。

打开cmd窗口进入J:\ProfessionalSoftware\kafka_2.12-2.3.0\bin\windows目录执行以下命令创建一个test。

```
kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
再同样的目录下创建生产者和消费者：

```
kafka-console-producer.bat --broker-list localhost:9092 --topic test
kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
```
之后，在生产者cmd窗口中输入信息，就能在消费者这边接收到，如下图所示。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/kafka-producer-consumer.PNG">
</div>

## 参考
[Kafka集群安裝部署(自带Zookeeper)](https://www.cnblogs.com/caoweixiong/p/11060533.html)</br>
[WINDOWS上KAFKA运行环境安装](https://www.cnblogs.com/lnice/p/9668750.html)</br>