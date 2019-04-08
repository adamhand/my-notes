# RabbitMQ系列(3)--第一个程序
---
# 简介
这是使用RabbitMQ写的第一个练习程序。生产者与RabbitMQ server连接之后，通过"direct"的方式发送一条"hello world"消息到队列中；消费者与RabbitMQ server连接之后就可以从队列中取出消息，然后打印。

要运行以下程序，首先通过`rabbitmq-service start`命令启动RabbitMQ server，然后运行`RabbitMQProducer`发送消息，接着运行`RabbitMQConsumer`就可以取出消息。

# 程序
`Producer`端的程序如下：
```java
public class RabbitMQProducer {
    private static final String EXCHANGE_NAME = "exchange_demo";
    private static final String ROUTING_KEY =   "routingkey_demo";
    private static final String QUEUE_NAME = "queue_demo";
    private static final String IPADDRESS = "192.168.1.107";
    private static final int PORT = 5672;        //RabbitMQ服务端默认端口号为5672

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(IPADDRESS);
        factory.setPort(PORT);
        factory.setUsername("root");     //要连接到RabbitMQ的用户名
        factory.setPassword("root");     //要连接到RabbitMQ的密码

        Connection connection = factory.newConnection();   //创建连接
        Channel channel = connection.createChannel();      //创建channel
        //创建一个交换类型为direct、持久化、非自动删除的交换器
        channel.exchangeDeclare(EXCHANGE_NAME, "direct", true, false, null);
        //创建一个持久化、非排他的、非自动删除的队列
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        //将交换机与队列通过路由键绑定
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);

        //发送一条持久化消息：hello world
        String msg = "hello world";
        channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY,
                MessageProperties.PERSISTENT_TEXT_PLAIN,
                msg.getBytes());

        //关闭资源
        channel.close();
        connection.close();
    }
}
```

`Consumer`端的程序如下：
```java
public class RabbitMQConsumer {
    private static final String QUEUE_NAME = "queue_demo";
    private static final String IPADDRESS = "192.168.1.107";
    private static final int PORT = 5672;

    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        Address[] addresses = new Address[]{
                new Address(IPADDRESS, PORT)
        };

        ConnectionFactory factory = new ConnectionFactory();
        factory.setUsername("root");
        factory.setPassword("root");

        Connection connection = factory.newConnection(addresses);
        final Channel channel = connection.createChannel();
        channel.basicQos(64);    //设置客户端最多接受未被ack的消息的个数

        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("receive msg "+new String(body));  //body代表消息实体
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        };
        //等待回调函数执行完毕之后，关闭资源
        channel.basicConsume(QUEUE_NAME, consumer);
        TimeUnit.SECONDS.sleep(5);
        channel.close();
        connection.close();
    }
}
```

运行成功后，`Consumer`端会打印如下消息：
```java
receive msg hello world
```

可以通过登录`localhost:15672`来查看连接状态。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/rabbitmq-server-listener.PNG">
</center>

# 参考
《RabbitMQ实战指南》