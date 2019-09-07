# kafka在windows下的安装和使用

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

## kafka的生产者消费者Java实现
目前有两种Java API的版本，一种是 kafka.javaapi.* :

```java
import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;
```
另一种是 org.apache.kafka.* :

```java
import org.apache.kafka.clients.producer.KafkaProducer<K,V>
```

在Kafka 0.8.2之前，kafka.javaapi.producer.Producer是为唯一官方用Scala实现的Java Client。
在Kafka 0.8.2之后，有新的Java Producer API，org.apache.kafka.clients.producer.KafkaProducer,完全用Java实现的。

下面的例子使用maven来做版本控制，需要在pom文件中添加以下依赖：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka_2.12</artifactId>
    <version>2.3.0</version>
</dependency>
```
这里的版本和安装的kafka版本一定要对应，否则运行时会出现如下的错误：

```java
SLF4J: Failed toString() invocation on an object of type [org.apache.kafka.clients.NodeApiVersions]
Reported exception:
java.lang.NullPointerException
	at org.apache.kafka.clients.NodeApiVersions.apiVersionToText(NodeApiVersions.java:167)
...
```

### 生产者代码

```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

public class Producer {
    private final KafkaProducer<String,String> producer;
    private final static String TOPIC = "first-kafka-test";

    Producer () {
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

        producer = new KafkaProducer<String, String>(properties);
    }

    public void produce() {
        int msgNum = 1;
        final int COUNT = 10;

        while (msgNum < COUNT) {
            String msg = String.format("hello kafka %s", String.valueOf(msgNum));
            try {
                producer.send(new ProducerRecord<String, String>(TOPIC, msg));
            } catch (Exception e) {
                e.printStackTrace();
            }
            msgNum++;
        }
        producer.close();
    }

    public static void main(String[] args) {
        new Producer().produce();
    }
}
```

### 消费者代码

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.util.Arrays;
import java.util.Properties;

public class Consumer {
    private KafkaConsumer<String, String> consumer;
    private static String group = "group-0";
    private static String TOPIC = "first-kafka-test";

    Consumer() {
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, group);
        properties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest");
        properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        properties.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
        properties.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "30000");
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

        consumer = new KafkaConsumer<String, String>(properties);
    }

    public void consume() {
        consumer.subscribe(Arrays.asList(TOPIC));

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(1000);
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("offset = %d, key = %s, value = %s \n", record.offset(), record.key(), record.value());
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        new Consumer().consume();
    }
}
```

启动kafka后，依次打开消费者和生产者，消费者端会将收到的消息打印，结果如下：

```java
offset = 81, key = null, value = hello kafka 1 
offset = 82, key = null, value = hello kafka 2 
offset = 83, key = null, value = hello kafka 3 
offset = 84, key = null, value = hello kafka 4 
offset = 85, key = null, value = hello kafka 5 
offset = 86, key = null, value = hello kafka 6 
offset = 87, key = null, value = hello kafka 7 
offset = 88, key = null, value = hello kafka 8 
offset = 89, key = null, value = hello kafka 9 
```

## 参考
[Kafka集群安裝部署(自带Zookeeper)](https://www.cnblogs.com/caoweixiong/p/11060533.html)</br>
[WINDOWS上KAFKA运行环境安装](https://www.cnblogs.com/lnice/p/9668750.html)</br>
[Kafka Java示例](https://blog.csdn.net/lavorange/article/details/78970977)