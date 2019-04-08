# RabbitMQ系列(4)--Spring 整合RabbitMQ
---

# 简介
Spring整合RabbitMQ需要添加以下Maven依赖：
```xml
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>2.0.2.RELEASE</version>
</dependency>
```
使用Spring-rabbitmq时主要有三个API比较重要：
```java
MessageListenerContainer: 用来监听容器，为消息入队提供异步处理
RabbitAdmin: 提供了一些列方法，比如declareExchange()、declareQueue()、declareBinding()用于声明交换机、队列、绑定等
RabbitTemplate: 用于RabbitMQ消息的发送和接收
```
下面分别讲解**Spring AMQP手动声明**、**Spring AMQP自动声明**、**Spring AMQP自定义消息处理器**、**Spring AMQP消息转换器**。

# Spring AMQP手动声明
手动声明的意思是要用户手动声明交换器、队列和绑定关系，主要步骤如下：

## 创建生产者配置类
在生产者配置类中将RabbitAdmin、RabbitTemplate纳入Spring管理，通过Spring自动扫描Bean的方式可以得到RabbitAdmin和RabbitTemplate对象。
```java
@Configuration
public class ProducerConfig {
    private static final String IPADDRESS = "192.168.1.107";
    private static final int PORT = 5672;
    private static final String VIRTUALHOST = "/";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "root";

    @Bean
    public CachingConnectionFactory connectionFactory(){
        ConnectionFactory factory = new ConnectionFactory();

        /**
         * 配置连接信息
         */
        factory.setHost(IPADDRESS);
        factory.setPort(PORT);
        factory.setVirtualHost(VIRTUALHOST);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);

        /**
         * 网络异常自动回复，每隔10秒重连一次
         */
        factory.setAutomaticRecoveryEnabled(true);
        factory.setNetworkRecoveryInterval(1000);

        /**
         * factory的属性
         */
        Map<String, Object> propertiesMap = new HashMap<>();
        propertiesMap.put("prindipal", "adamhand");
        propertiesMap.put("description", "Order System");
        propertiesMap.put("email", "adaihand@163.com");
        factory.setClientProperties(propertiesMap);

        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory(factory);

        return cachingConnectionFactory;
    }

    /**
     * 用于声明交换机 队列 绑定等
     * @param factory
     * @return
     */
    @Bean
    public RabbitAdmin rabbitAdmin(CachingConnectionFactory factory){
        return new RabbitAdmin(factory);
    }

    /**
     * 用于RabbitMQ消息的发送和接收
     * @param factory
     * @return
     */
    @Bean
    public RabbitTemplate rabbitTemplate(CachingConnectionFactory factory){
        return new RabbitTemplate(factory);
    }
}
```
## 生产者启动类
在生产者启动类中声明交换器、队列和绑定关系（注意`@ComponentScan`扫描的包不能错）。
```java
@ComponentScan(basePackages = "com.rabbitmq.SpringAMQP.ManualDeclare.Producer")
public class ProducerBootstrap {
    private static final String EXCHANGE_NAME = "adamhand.order";
    private static final String ROUTING_KEUY = "add";
    private static final String CONTENT_TYPE = "UTF-8";

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(ProducerBootstrap.class);
        RabbitAdmin rabbitAdmin = context.getBean(RabbitAdmin.class);
        RabbitTemplate rabbitTemplate = context.getBean(RabbitTemplate.class);

        /**
         * 声明交换机
         */
        rabbitAdmin.declareExchange(
                new DirectExchange(EXCHANGE_NAME, true, false,
                new HashMap<String, Object>()));

        /**
         * 声明要发送的消息
         */
        MessageProperties msgProperties = new MessageProperties();
        msgProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
        msgProperties.setContentType(CONTENT_TYPE);
        Message msg = new Message("order information".getBytes(), msgProperties);

        /**
         * 发布消息
         */
        rabbitTemplate.send(EXCHANGE_NAME, ROUTING_KEUY, msg);
    }
}
```

## 消费者配置类
创建消费者配置类，将RabbitAdmin纳入Spring管理，并在MessageListenerContainer类中定义了消息消费的逻辑。
```java
@Configuration
public class ConsumerConfig {
    private static final String IPADDRESS = "192.168.1.107";
    private static final int PORT = 5672;
    private static final String VIRTUALHOST = "/";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "root";
    private static final String QUEUE_NAMES = "adamhand.order.add";
    private static final int CONSUMER_NUMBER = 5;
    private static final int MAX_CONSUMER_NUMBER = 10;

    @Bean
    public CachingConnectionFactory connectionFactory(){
        ConnectionFactory factory = new ConnectionFactory();

        factory.setHost(IPADDRESS);
        factory.setPort(PORT);
        factory.setVirtualHost(VIRTUALHOST);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);

        factory.setAutomaticRecoveryEnabled(true);
        factory.setNetworkRecoveryInterval(1000);

        Map<String, Object> propertiesMap = new HashMap<>();
        propertiesMap.put("prindipal", "adamhand");
        propertiesMap.put("description", "Order System");
        propertiesMap.put("email", "adaihand@163.com");
        factory.setClientProperties(propertiesMap);

        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory(factory);

        return cachingConnectionFactory;
    }

    @Bean
    public RabbitAdmin rabbitAdmin(CachingConnectionFactory factory){
        return new RabbitAdmin(factory);
    }

    /**
     * 监听容器 为消息入队提供异步处理
     * @param factory
     * @return
     */
    @Bean
    public MessageListenerContainer messageListenerContainer(CachingConnectionFactory factory){
        SimpleMessageListenerContainer msgLIstenerContainer = new SimpleMessageListenerContainer();
        msgLIstenerContainer.setConnectionFactory(factory);
        msgLIstenerContainer.setQueueNames(QUEUE_NAMES);

        /**
         * 设置消费者线程数和最大线程数
         */
        msgLIstenerContainer.setConcurrentConsumers(CONSUMER_NUMBER);
        msgLIstenerContainer.setMaxConcurrentConsumers(MAX_CONSUMER_NUMBER);

        /**
         * 设置消费者属性
         */
        Map<String, Object> argsMap = new HashMap<>();
        msgLIstenerContainer.setConsumerArguments(argsMap);

        /**
         * 设置消费者标签
         */
        msgLIstenerContainer.setConsumerTagStrategy(new ConsumerTagStrategy() {
            @Override
            public String createConsumerTag(String s) {
                return "Consumers of Order System";
            }
        });

        /**
         * 选择手动设置消费时机
         */
        msgLIstenerContainer.setAutoStartup(false);

        /**
         * 消息后置处理器。接收到消息之后打印出来。
         */
        msgLIstenerContainer.setMessageListener(new MessageListener() {
            @Override
            public void onMessage(Message message) {
                try {
                    System.out.println(new String(message.getBody(), "UTF-8"));
                    System.out.println(message.getMessageProperties());
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            }
        });

        return msgLIstenerContainer;
    }
}
```

## 创建消费者启动类
```java
@ComponentScan(basePackages = "com.rabbitmq.SpringAMQP.ManualDeclare.Consumer")
public class ConsumerBootstrap {
    private static final String EXCHANGE_NAME = "adamhand.order";
    private static final String QUEUE_NAME = "adamhand.order.add";
    private static final String ROUTING_KEY = "add";

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(ConsumerBootstrap.class);
        RabbitAdmin rabbitAdmin = context.getBean(RabbitAdmin.class);
        MessageListenerContainer msgListenerContainer =
                context.getBean(MessageListenerContainer.class);

        /**
         * 声明一个direct类型的、持久化、非排他的交换器
         */
        rabbitAdmin.declareExchange(new DirectExchange(EXCHANGE_NAME,
                             true, false,
                                    new HashMap<String, Object>()));

        /**
         * 声明一个持久化、非排他、非自动删除的队列
         */
        rabbitAdmin.declareQueue(new Queue(QUEUE_NAME,
                                            true, false, false,
                                                    new HashMap<String, Object>()));

        /**
         * 将交换器和队列绑定
         */
        rabbitAdmin.declareBinding(BindingBuilder.bind(new Queue(QUEUE_NAME)).
                                                        to(new DirectExchange(EXCHANGE_NAME)).
                                                        with(ROUTING_KEY));

        /**
         * 开始监听
         */
        msgListenerContainer.start();
    }
}
```

## 结果
打开`rabbitmq-server`(`cmd` 中输入`rabbitmq-server.bat`，如果安装了`rabbitmq-service`，可以通过`rabbitmq-service start`的方式打开)。接着打开消费者和she

# Spring AMQP自动声明
Spring AMQP还提供了自动声明方式交换机、队列和绑定。可以直接把要自动声明的组件纳入Spring容器中管理即可。

自动声明发生在RabbitMQ第一次连接创建的时候，自动声明支持单个和批量自动声明。使用自动声明需要符合如下条件(下面的几个条件定义在RabbitAdmin的afterPropertiesSet方法中):

- 需要有连接产生
- RabbitAdmin必须交由Spring管理，且autoStartup必须为true(默认)
- 如果ConnectionFactory使用的是CachingConnectionFactory，则cacheMode必须要为CacheMode.CHANNEL
- 所有要声明的组件的shouldDeclare必须为true
- 要声明的Queue名称不能以amq.开头

## 创建生产者配置类
将RabbitAdmin、RabbitTemplate纳入Spring管理。

```java
@Configuration
public class ProducerConfig {
    private static final String IPADDRESS = "192.168.1.107";
    private static final int PORT = 5672;
    private static final String VIRTUALHOST = "/";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "root";

    @Bean
    public CachingConnectionFactory connectionFactory(){
        ConnectionFactory factory = new ConnectionFactory();

        /**
         * 配置连接信息
         */
        factory.setHost(IPADDRESS);
        factory.setPort(PORT);
        factory.setVirtualHost(VIRTUALHOST);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);

        /**
         * 网络异常自动回复，每隔10秒重连一次
         */
        factory.setAutomaticRecoveryEnabled(true);
        factory.setNetworkRecoveryInterval(1000);

        /**
         * factory的属性
         */
        Map<String, Object> propertiesMap = new HashMap<>();
        propertiesMap.put("prindipal", "adamhand");
        propertiesMap.put("description", "Order System");
        propertiesMap.put("email", "adaihand@163.com");
        factory.setClientProperties(propertiesMap);

        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory(factory);

        return cachingConnectionFactory;
    }

    /**
     * 用于声明交换机 队列 绑定等
     * @param factory
     * @return
     */
    @Bean
    public RabbitAdmin rabbitAdmin(CachingConnectionFactory factory){
        return new RabbitAdmin(factory);
    }

    /**
     * 用于RabbitMQ消息的发送和接收
     * @param factory
     * @return
     */
    @Bean
    public RabbitTemplate rabbitTemplate(CachingConnectionFactory factory){
        return new RabbitTemplate(factory);
    }
}
```

## 创建生产者启动类

```java
@ComponentScan(basePackages = "com.rabbitmq.SpringAMQP.AutoDeclare.Producer")
public class ProducerBootstrap {
    private static final String EXCHANGE_NAME = "adamhand.order";
    private static final String ROUTING_KEUY = "add";
    private static final String CONTENT_TYPE = "UTF-8";

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(ProducerBootstrap.class);
        RabbitAdmin rabbitAdmin = context.getBean(RabbitAdmin.class);
        RabbitTemplate rabbitTemplate = context.getBean(RabbitTemplate.class);

        /**
         * 声明交换机
         */
        rabbitAdmin.declareExchange(
                new DirectExchange(EXCHANGE_NAME, true, false,
                new HashMap<String, Object>()));

        /**
         * 声明要发送的消息
         */
        MessageProperties msgProperties = new MessageProperties();
        msgProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
        msgProperties.setContentType(CONTENT_TYPE);
        Message msg = new Message("order information".getBytes(), msgProperties);

        /**
         * 发布消息
         */
        rabbitTemplate.send(EXCHANGE_NAME, ROUTING_KEUY, msg);
    }
}
```

## 创建消费者配置类
将RabbitAdmin纳入Spring管理，并在MessageListenerContainer类中定义了消息消费的逻辑，并且在该配置类中声明交换机，队列，绑定。

```java
@Configuration
public class ConsumerConfig {
    private static final String IPADDRESS = "192.168.1.107";
    private static final int PORT = 5672;
    private static final String VIRTUALHOST = "/";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "root";
    private static final String QUEUE_NAMES = "adamhand.order.add";
    private static final String EXCHANGE_NAMES = "adamhand.order";
    private static final String ROUTING_KEY = "add";
    private static final int CONSUMER_NUMBER = 5;
    private static final int MAX_CONSUMER_NUMBER = 10;

    @Bean
    public CachingConnectionFactory connectionFactory(){
        ConnectionFactory factory = new ConnectionFactory();

        factory.setHost(IPADDRESS);
        factory.setPort(PORT);
        factory.setVirtualHost(VIRTUALHOST);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);

        factory.setAutomaticRecoveryEnabled(true);
        factory.setNetworkRecoveryInterval(1000);

        Map<String, Object> propertiesMap = new HashMap<>();
        propertiesMap.put("prindipal", "adamhand");
        propertiesMap.put("description", "Order System");
        propertiesMap.put("email", "adaihand@163.com");
        factory.setClientProperties(propertiesMap);

        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory(factory);

        return cachingConnectionFactory;
    }

    @Bean
    public RabbitAdmin rabbitAdmin(CachingConnectionFactory factory){
        return new RabbitAdmin(factory);
    }

    /**
     * 自动声明队列
     * @return
     */
    @Bean
    public Queue queue(){
        return new Queue(QUEUE_NAMES, true , false, false,
                new HashMap<String, Object>());
    }

    @Bean
    public Exchange exchange(){
        return new DirectExchange(EXCHANGE_NAMES, true, false,
                new HashMap<String, Object>());
    }

    @Bean
    public Binding binding(){
        return new Binding(QUEUE_NAMES, Binding.DestinationType.QUEUE,
                EXCHANGE_NAMES, ROUTING_KEY, new HashMap<String, Object>());
    }



    /**
     * 监听容器 为消息入队提供异步处理
     * @param factory
     * @return
     */
    @Bean
    public MessageListenerContainer messageListenerContainer(CachingConnectionFactory factory){
        SimpleMessageListenerContainer msgLIstenerContainer = new SimpleMessageListenerContainer();
        msgLIstenerContainer.setConnectionFactory(factory);
        msgLIstenerContainer.setQueueNames(QUEUE_NAMES);

        /**
         * 设置消费者线程数和最大线程数
         */
        msgLIstenerContainer.setConcurrentConsumers(CONSUMER_NUMBER);
        msgLIstenerContainer.setMaxConcurrentConsumers(MAX_CONSUMER_NUMBER);

        /**
         * 设置消费者属性
         */
        Map<String, Object> argsMap = new HashMap<>();
        msgLIstenerContainer.setConsumerArguments(argsMap);

        /**
         * 设置消费者标签
         */
        msgLIstenerContainer.setConsumerTagStrategy(new ConsumerTagStrategy() {
            @Override
            public String createConsumerTag(String s) {
                return "Consumers of Order System";
            }
        });

        /**
         * 选择自动设置消费时机
         */
        msgLIstenerContainer.setAutoStartup(true);

        /**
         * 消息后置处理器。接收到消息之后打印出来。
         */
        msgLIstenerContainer.setMessageListener(new MessageListener() {
            @Override
            public void onMessage(Message message) {
                try {
                    System.out.println(new String(message.getBody(), "UTF-8"));
                    System.out.println(message.getMessageProperties());
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            }
        });

        return msgLIstenerContainer;
    }
}

```

## 创建消费者启动类

```java
@ComponentScan(basePackages = "com.rabbitmq.SpringAMQP.AutoDeclare.Consumer")
public class ConsumerBootstrap {
    public static void main(String[] args) {
        new AnnotationConfigApplicationContext(ConsumerBootstrap.class);
    }
}
```

## 结果
启动消费者和生产者，控制台打印结果如下：

```java
order information
MessageProperties [headers={}, contentType=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=adamhand.order, receivedRoutingKey=add, deliveryTag=1, consumerTag=Consumers of Order System, consumerQueue=adamhand.order.add]
```

# 自定义消息处理器--MessageListenerAdapte
上面的两个例子都是使用MessageListenerContainer中传递了MessageListener的方式来处理消费者的消息处理逻辑的，但是这样有一个缺点：不好扩展。如果想再多加上一些消息处理的逻辑，势必要修改MessageListener的代码。

SpringAMQP提供了一种消息处理器适配器的功能，它可以把一个纯POJO类适配成一个可以处理消息的处理器，默认处理消息的方法为handleMessage，可以通过setDefaultListenerMethod方法进行修改。

## 创建生产者配置类

```java
@Configuration
public class ProducerConfig {
    private static final String IPADDRESS = "192.168.1.107";
    private static final int PORT = 5672;
    private static final String VIRTUALHOST = "/";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "root";

    @Bean
    public CachingConnectionFactory connectionFactory(){
        ConnectionFactory factory = new ConnectionFactory();

        /**
         * 配置连接信息
         */
        factory.setHost(IPADDRESS);
        factory.setPort(PORT);
        factory.setVirtualHost(VIRTUALHOST);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);

        /**
         * 网络异常自动回复，每隔10秒重连一次
         */
        factory.setAutomaticRecoveryEnabled(true);
        factory.setNetworkRecoveryInterval(1000);

        /**
         * factory的属性
         */
        Map<String, Object> propertiesMap = new HashMap<>();
        propertiesMap.put("prindipal", "adamhand");
        propertiesMap.put("description", "Order System");
        propertiesMap.put("email", "adaihand@163.com");
        factory.setClientProperties(propertiesMap);

        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory(factory);

        return cachingConnectionFactory;
    }

    /**
     * 用于声明交换机 队列 绑定等
     * @param factory
     * @return
     */
    @Bean
    public RabbitAdmin rabbitAdmin(CachingConnectionFactory factory){
        return new RabbitAdmin(factory);
    }

    /**
     * 用于RabbitMQ消息的发送和接收
     * @param factory
     * @return
     */
    @Bean
    public RabbitTemplate rabbitTemplate(CachingConnectionFactory factory){
        return new RabbitTemplate(factory);
    }
}
```

## 创建生产者启动类

```java
@ComponentScan(basePackages = "com.rabbitmq.SpringAMQP.MsgListenerAdapte.Producer")
public class ProducerBootstrap {
    private static final String EXCHANGE_NAME = "adamhand.order";
    private static final String ROUTING_KEUY = "add";
    private static final String CONTENT_TYPE = "UTF-8";

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(ProducerBootstrap.class);
        RabbitAdmin rabbitAdmin = context.getBean(RabbitAdmin.class);
        RabbitTemplate rabbitTemplate = context.getBean(RabbitTemplate.class);

        /**
         * 声明交换机
         */
        rabbitAdmin.declareExchange(
                new DirectExchange(EXCHANGE_NAME, true, false,
                new HashMap<String, Object>()));

        /**
         * 声明要发送的消息
         */
        MessageProperties msgProperties = new MessageProperties();
        msgProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
        msgProperties.setContentType(CONTENT_TYPE);
        Message msg = new Message("order information".getBytes(), msgProperties);

        /**
         * 发布消息
         */
        rabbitTemplate.send(EXCHANGE_NAME, ROUTING_KEUY, msg);
    }
}
```

## 创建消费者消息处理器类
它可是是纯POJO类。

```java
public class MessageHandle {
    public void add(byte[] message){
        try {
            System.out.println(new String(message,"UTF-8"));
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }
}
```

## 创建消费者配置类
配置自定义消息处理器(将adamhand.order.add队列使用自定义消息处理类的add方法进行处理)。

```java
@Configuration
public class ConsumerConfig {
    private static final String IPADDRESS = "192.168.1.107";
    private static final int PORT = 5672;
    private static final String VIRTUALHOST = "/";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "root";
    private static final String QUEUE_NAMES = "adamhand.order.add";
    private static final String EXCHANGE_NAMES = "adamhand.order";
    private static final String ROUTING_KEY = "add";
    private static final int CONSUMER_NUMBER = 5;
    private static final int MAX_CONSUMER_NUMBER = 10;

    @Bean
    public CachingConnectionFactory connectionFactory(){
        ConnectionFactory factory = new ConnectionFactory();

        factory.setHost(IPADDRESS);
        factory.setPort(PORT);
        factory.setVirtualHost(VIRTUALHOST);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);

        factory.setAutomaticRecoveryEnabled(true);
        factory.setNetworkRecoveryInterval(1000);

        Map<String, Object> propertiesMap = new HashMap<>();
        propertiesMap.put("prindipal", "adamhand");
        propertiesMap.put("description", "Order System");
        propertiesMap.put("email", "adaihand@163.com");
        factory.setClientProperties(propertiesMap);

        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory(factory);

        return cachingConnectionFactory;
    }

    @Bean
    public RabbitAdmin rabbitAdmin(CachingConnectionFactory factory){
        return new RabbitAdmin(factory);
    }

    /**
     * 自动声明队列
     * @return
     */
    @Bean
    public Exchange exchange(){
        return new DirectExchange(EXCHANGE_NAMES, true, false,
                new HashMap<String, Object>());
    }

    @Bean
    public Binding binding(){
        return new Binding(QUEUE_NAMES, Binding.DestinationType.QUEUE,
                EXCHANGE_NAMES, ROUTING_KEY, new HashMap<String, Object>());
    }



    /**
     * 监听容器 为消息入队提供异步处理
     * @param factory
     * @return
     */
    @Bean
    public MessageListenerContainer messageListenerContainer(CachingConnectionFactory factory){
        SimpleMessageListenerContainer msgLIstenerContainer = new SimpleMessageListenerContainer();
        msgLIstenerContainer.setConnectionFactory(factory);
        msgLIstenerContainer.setQueueNames(QUEUE_NAMES);

        /**
         * 设置消费者线程数和最大线程数
         */
        msgLIstenerContainer.setConcurrentConsumers(CONSUMER_NUMBER);
        msgLIstenerContainer.setMaxConcurrentConsumers(MAX_CONSUMER_NUMBER);

        /**
         * 设置消费者属性
         */
        Map<String, Object> argsMap = new HashMap<>();
        msgLIstenerContainer.setConsumerArguments(argsMap);

        /**
         * 设置消费者标签
         */
        msgLIstenerContainer.setConsumerTagStrategy(new ConsumerTagStrategy() {
            @Override
            public String createConsumerTag(String s) {
                return "Consumers of Order System";
            }
        });

        /**
         * 选择自动设置消费时机
         */
        msgLIstenerContainer.setAutoStartup(true);

        /**
         * 创建消息适配器
         */
        MessageListenerAdapter msgListenerAdapter = new MessageListenerAdapter(new MessageHandle());
        msgListenerAdapter.setDefaultListenerMethod("handleMessage");
        Map<String, String> map = new HashMap<>();
        map.put(QUEUE_NAMES, ROUTING_KEY);
        msgListenerAdapter.setQueueOrTagToMethodName(map);
        msgLIstenerContainer.setMessageListener(msgListenerAdapter);

        return msgLIstenerContainer;
    }
}
```

## 创建消费者启动类

```java
@ComponentScan(basePackages = "com.rabbitmq.SpringAMQP.MsgListenerAdapte.Consumer")
public class ConsumerBootstrap {
    public static void main(String[] args) {
        new AnnotationConfigApplicationContext(ConsumerBootstrap.class);
    }
}
```

## 结果
启动消费者和生产者，打印结果如下：

```java
order information
```

如上`Demo`说明我们可以将一个纯`POJO`类定义为消息处理器，并且不用去扩展`MessageListener/ChannelAwareMessageListener`接口，关于自定义处理器方法的参数默认情况下为`byte[]`类型，这是由`Spring AMQP`默认消息转换器(`SimpleMessageConverter`)决定的，接下来将介绍`Spring AMQP`的消息转换器功能。

# Spring AMQP消息转换器
在上面自定义的MessageHandle函数中，add方法的参数为byte[]，但是有时候我们往RabbitMQ中发送的是一个JSON对象，我们希望在处理消息的时候它已经自动帮我们转为JAVA对象；又或者我们往RabbitMQ中发送的是一张图片或其他格式的文件，我们希望在处理消息的时候它已经自动帮我们转成文件格式，我们可以手动设置MessageConverter来实现如上需求，如果未设置MessageConverter则使用Spring AMQP默认提供的**SimpleMessageConverter**。

以下例子使用**MessageConverter**实现了当生产者往RabbitMQ发送不同类型的数据的时候，使用MessageHandle不同的方法进行处理，需要注意的是当生产者在发送JSON数据的时候，需要指定这个JSON是哪个对象，用于Spring AMQP转换，规则如下：

```java
当发送普通对象的JSON数据时，需要在消息的header中增加一个__TypeId__的属性告知消费者是哪个对象

当发送List集合对象的JSON数据时，需要在消息的header中将__TypeId__指定为java.util.List，并且需要额外指定属性__ContentTypeId__用户告知消费者List集合中的对象类型

当发送Map集合对象的JSON数据时，需要在消息的header中将__TypeId__指定为java.util.Map，并且需要额外指定属性__KeyTypeId__用于告知客户端Map中key的类型，__ContentTypeId__用于告知客户端Map中Value的类型
```

## 创建订单实体类
使用lombok管理getter和setter方法。
```java
@Data
public class Order {
    private String id;
    private float price;

    public Order() {
    }

    public Order(String id, float price) {
        this.id = id;
        this.price = price;
    }
}
```

## 自定义转换器
自定义文件转换器。
```java
public class FileMessageConverter implements MessageConverter {
    @Override
    public Message toMessage(Object o, MessageProperties messageProperties) throws MessageConversionException {
        return null;
    }

    @Override
    public Object fromMessage(Message message) throws MessageConversionException {
        String exName = (String) message.getMessageProperties().getHeaders().get("_extName");
        byte[] data = message.getBody();

        String fileName = UUID.randomUUID().toString();
        String filePath = System.getProperty("java.io.temdir")+ fileName +"." + exName;
        File temFile = new File(filePath);

        try {
            FileCopyUtils.copy(data, temFile);
        } catch (IOException e) {
            throw new MessageConversionException("file message convert fails", e);
        }

        return temFile;
    }
}
```

自定义字符串转换器。
```java
public class StringMessageConverter implements MessageConverter {
    @Override
    public Message toMessage(Object o, MessageProperties messageProperties) throws MessageConversionException {
        return null;
    }

    @Override
    public Object fromMessage(Message message) throws MessageConversionException {
        try {
            return new String(message.getBody(), "UTF-8");
        } catch (UnsupportedEncodingException e) {
            throw new MessageConversionException("string message convert fails", e);
        }
    }
}
```

自定义消息处理类。
```java
public class MessageHandle {
    public void add(byte[] data){
        System.out.println("use byte[] to handle");
        System.out.println(data.toString());
    }

    public void add(String msg){
        System.out.println("use String to handle");
        System.out.println(msg);
    }

    public void add(File file){
        System.out.println("use File to handle");
        System.out.println(file.length() +" "+ file.getName()+ " " + file.getAbsolutePath());
    }

    public void add(Order order){
        System.out.println("use Order to handle");
        System.out.println(order.getId()+" "+ order.getPrice());
    }

    public void add(List<Order> list){
        System.out.println("use list to handle");
        System.out.println(list.size());
        for (Order o : list){
            System.out.println(o.getId()+" "+o.getPrice());
        }
    }

    public void add(Map<String, Order> map){
        System.out.println("use Map to handle");
        for(Map.Entry<String, Order> entry : map.entrySet()){
            System.out.println(entry.getKey() +" "+ entry.getValue());
        }
    }
}
```

## 创建生产者配置类
```java
@Configuration
public class ProducerConfig {
    private static final String IPADDRESS = "192.168.1.107";
    private static final int PORT = 5672;
    private static final String VIRTUALHOST = "/";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "root";

    @Bean
    public CachingConnectionFactory connectionFactory(){
        ConnectionFactory factory = new ConnectionFactory();

        /**
         * 配置连接信息
         */
        factory.setHost(IPADDRESS);
        factory.setPort(PORT);
        factory.setVirtualHost(VIRTUALHOST);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);

        /**
         * 网络异常自动回复，每隔10秒重连一次
         */
        factory.setAutomaticRecoveryEnabled(true);
        factory.setNetworkRecoveryInterval(1000);

        /**
         * factory的属性
         */
        Map<String, Object> propertiesMap = new HashMap<>();
        propertiesMap.put("prindipal", "adamhand");
        propertiesMap.put("description", "Order System");
        propertiesMap.put("email", "adaihand@163.com");
        factory.setClientProperties(propertiesMap);

        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory(factory);

        return cachingConnectionFactory;
    }

    /**
     * 用于声明交换机 队列 绑定等
     * @param factory
     * @return
     */
    @Bean
    public RabbitAdmin rabbitAdmin(CachingConnectionFactory factory){
        return new RabbitAdmin(factory);
    }

    /**
     * 用于RabbitMQ消息的发送和接收
     * @param factory
     * @return
     */
    @Bean
    public RabbitTemplate rabbitTemplate(CachingConnectionFactory factory){
        return new RabbitTemplate(factory);
    }
}
```

## 创建生产者启动方法
```java
@ComponentScan(basePackages = "com.rabbitmq.SpringAMQP.MessageConverter.Producer")
public class ProducerBootstrap {
    private static final String EXCHANGE_NAME = "adamhand.order";
    private static final String ROUTING_KEY = "add";
    private static final String CONTENT_TYPE_TEXT = "text/plain";
    private static final String CONTENT_TYPE_JSON = "application/json";
    private static final String CONTENT_TYPE_IMAGE = "image/jpg";

    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(ProducerBootstrap.class);
        RabbitAdmin rabbitAdmin = context.getBean(RabbitAdmin.class);
        RabbitTemplate rabbitTemplate = context.getBean(RabbitTemplate.class);

        /**
         * 声明交换机
         */
        rabbitAdmin.declareExchange(
                new DirectExchange(EXCHANGE_NAME, true, false,
                new HashMap<String, Object>()));

        /**
         * 发送String
         */
        sendString(rabbitTemplate);
        /**
         * 发送单个对象JSON
         */
        sendSingle(rabbitTemplate);
        /**
         * 发送List集合JSON
         */
        sendList(rabbitTemplate);
        /**
         * 发送Map集合JSON
         */
        sendMap(rabbitTemplate);
        /**
         * 发送图片
         */
        sendImage(rabbitTemplate);
    }

    public static void sendString(RabbitTemplate rabbitTemplate){
        MessageProperties msgProperties = new MessageProperties();
        msgProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
        msgProperties.setContentType(CONTENT_TYPE_TEXT);
        Message msg = new Message("order information".getBytes(), msgProperties);

        rabbitTemplate.send(EXCHANGE_NAME, ROUTING_KEY, msg);
    }

    public static void sendSingle(RabbitTemplate rabbitTemplate) throws JsonProcessingException {
        Order order = new Order("OD00000000001", (float) 5555.5555);
        ObjectMapper mapper = new ObjectMapper();

        MessageProperties msgProperties = new MessageProperties();
        msgProperties.getHeaders().put("_TypeId_", "order");
        msgProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
        msgProperties.setContentType(CONTENT_TYPE_JSON);
        Message msg = new Message(mapper.writeValueAsString(order).getBytes(), msgProperties);

        rabbitTemplate.send(EXCHANGE_NAME, ROUTING_KEY, msg);
    }

    public static void sendList(RabbitTemplate rabbitTemplate) throws JsonProcessingException {
        Order order1 = new Order("OD00000000001", (float) 5555.5555);
        Order order2 = new Order("OD00000000002", (float) 3333.3333);
        List<Order> orderList = Arrays.asList(order1, order2);

        ObjectMapper mapper = new ObjectMapper();

        MessageProperties msgProperties = new MessageProperties();
        msgProperties.getHeaders().put("__TypeId__", "java.util.List");
        msgProperties.getHeaders().put("__ContentTypeId__", "order");
        msgProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
        msgProperties.setContentType(CONTENT_TYPE_JSON);
        Message msg = new Message(mapper.writeValueAsString(orderList).getBytes(), msgProperties);

        rabbitTemplate.send(EXCHANGE_NAME, ROUTING_KEY, msg);
    }

    public static void sendMap(RabbitTemplate rabbitTemplate) throws Exception {
        Order order1 = new Order("OD0000001", (float)1111.1111);
        Order order2 = new Order("OD0000002", (float)2222.2222);
        Map<String, Order> orderMap = new HashMap<>();
        orderMap.put(order1.getId(), order1);
        orderMap.put(order2.getId(), order2);

        ObjectMapper objectMapper = new ObjectMapper();
        // 声明消息 (消息体, 消息属性)
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.getHeaders().put("__TypeId__", "java.util.Map");
        messageProperties.getHeaders().put("__KeyTypeId__", "java.lang.String");
        messageProperties.getHeaders().put("__ContentTypeId__", "order");
        messageProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
        messageProperties.setContentType(CONTENT_TYPE_JSON);
        Message message = new Message(objectMapper.writeValueAsString(orderMap).getBytes(), messageProperties);

        rabbitTemplate.send(EXCHANGE_NAME, ROUTING_KEY, message);
    }

    public static void sendImage(RabbitTemplate rabbitTemplate) throws Exception {
        File file = new File(System.getProperty("user.dir")+"\\naruto.jpg"); //程序运行之前该图片要存在
        FileInputStream fileInputStream = new FileInputStream(file);
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream(1024);
        int length;
        byte[] b = new byte[1024];
        while ((length = fileInputStream.read(b)) != -1) {
            byteArrayOutputStream.write(b, 0, length);
        }
        fileInputStream.close();
        byteArrayOutputStream.close();
        byte[] buffer = byteArrayOutputStream.toByteArray();

        // 声明消息 (消息体, 消息属性)
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.getHeaders().put("_extName", "jpg");
        messageProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
        messageProperties.setContentType(CONTENT_TYPE_IMAGE);
        Message message = new Message(buffer, messageProperties);

        rabbitTemplate.send(EXCHANGE_NAME, ROUTING_KEY, message);
    }
}
```

## 创建消费者配置类
```java
@Configuration
public class ConsumerConfig {
    private static final String IPADDRESS = "192.168.1.107";
    private static final int PORT = 5672;
    private static final String VIRTUALHOST = "/";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "root";
    private static final String QUEUE_NAMES = "adamhand.order.add";
    private static final String EXCHANGE_NAMES = "adamhand.order";
    private static final String ROUTING_KEY = "add";
    private static final int CONSUMER_NUMBER = 5;
    private static final int MAX_CONSUMER_NUMBER = 10;

    @Bean
    public CachingConnectionFactory connectionFactory(){
        ConnectionFactory factory = new ConnectionFactory();

        factory.setHost(IPADDRESS);
        factory.setPort(PORT);
        factory.setVirtualHost(VIRTUALHOST);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);

        factory.setAutomaticRecoveryEnabled(true);
        factory.setNetworkRecoveryInterval(1000);

        Map<String, Object> propertiesMap = new HashMap<>();
        propertiesMap.put("prindipal", "adamhand");
        propertiesMap.put("description", "Order System");
        propertiesMap.put("email", "adaihand@163.com");
        factory.setClientProperties(propertiesMap);

        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory(factory);

        return cachingConnectionFactory;
    }

    @Bean
    public RabbitAdmin rabbitAdmin(CachingConnectionFactory factory){
        return new RabbitAdmin(factory);
    }

    /**
     * 自动声明队列
     * @return
     */
    @Bean
    public Exchange exchange(){
        return new DirectExchange(EXCHANGE_NAMES, true, false,
                new HashMap<String, Object>());
    }

    @Bean
    public Binding binding(){
        return new Binding(QUEUE_NAMES, Binding.DestinationType.QUEUE,
                EXCHANGE_NAMES, ROUTING_KEY, new HashMap<String, Object>());
    }



    /**
     * 监听容器 为消息入队提供异步处理
     * @param factory
     * @return
     */
    @Bean
    public MessageListenerContainer messageListenerContainer(CachingConnectionFactory factory){
        SimpleMessageListenerContainer msgLIstenerContainer = new SimpleMessageListenerContainer();
        msgLIstenerContainer.setConnectionFactory(factory);
        msgLIstenerContainer.setQueueNames(QUEUE_NAMES);

        /**
         * 设置消费者线程数和最大线程数
         */
        msgLIstenerContainer.setConcurrentConsumers(CONSUMER_NUMBER);
        msgLIstenerContainer.setMaxConcurrentConsumers(MAX_CONSUMER_NUMBER);

        /**
         * 设置消费者属性
         */
        Map<String, Object> argsMap = new HashMap<>();
        msgLIstenerContainer.setConsumerArguments(argsMap);

        /**
         * 设置消费者标签
         */
        msgLIstenerContainer.setConsumerTagStrategy(new ConsumerTagStrategy() {
            @Override
            public String createConsumerTag(String s) {
                return "Consumers of Order System";
            }
        });

        /**
         * 选择自动设置消费时机
         */
        msgLIstenerContainer.setAutoStartup(true);

        //注意这个MessageHandle()一定要写对
        MessageListenerAdapter msgListenerAdapter = new MessageListenerAdapter(new MessageHandle());
        msgListenerAdapter.setDefaultListenerMethod("handleMessage");
        Map<String, String> map = new HashMap<>();
        map.put(QUEUE_NAMES, ROUTING_KEY);
        msgListenerAdapter.setQueueOrTagToMethodName(map);
        msgLIstenerContainer.setMessageListener(msgListenerAdapter);

        /**
         * 设置消息转换器
         */
        ContentTypeDelegatingMessageConverter converter =
                new ContentTypeDelegatingMessageConverter();
        StringMessageConverter stringMessageConverter = new StringMessageConverter();
        FileMessageConverter fileMessageConverter = new FileMessageConverter();
        Jackson2JsonMessageConverter jackson2JsonMessageConverter = new
                Jackson2JsonMessageConverter();
        Map<String, Class<?>> idClassMapping = new HashMap<>();
        idClassMapping.put("order", Order.class);
        DefaultJackson2JavaTypeMapper javaTypeMapper = new DefaultJackson2JavaTypeMapper();
        javaTypeMapper.setIdClassMapping(idClassMapping);
        jackson2JsonMessageConverter.setJavaTypeMapper(javaTypeMapper);

        /**
         * 设置text/html text/plain 使用StringMessageConverter
         */
        converter.addDelegate("text/html", stringMessageConverter);
        converter.addDelegate("text/plain", stringMessageConverter);
        /**
         * 设置application/json 使用Jackson2JsonMessageConverter
         */
        converter.addDelegate("application/json", jackson2JsonMessageConverter);
        /**
         * 设置image/jpg image/png 使用FileMessageConverter
         */
        converter.addDelegate("image/jpg", fileMessageConverter);
        converter.addDelegate("image/png", fileMessageConverter);
        msgListenerAdapter.setMessageConverter(converter);

        return msgLIstenerContainer;
    }
}
```

## 创建消费者启动类
```java
@ComponentScan(basePackages = "com.rabbitmq.SpringAMQP.MessageConverter.Consumer")
public class ConsumerBootstrap {
    public static void main(String[] args) {
        new AnnotationConfigApplicationContext(ConsumerBootstrap.class);
    }
}
```

## 结果
启动消费者启动类和生产者启动类，结果如下：
```java
use File to handle
45550 nulldc5f8104-e9cd-48f7-a8a1-2082556565ab.jpg D:\Prom\rabbitmq-master\nulldc5f8104-e9cd-48f7-a8a1-2082556565ab.jpg
use Map to handle
id OD00000000001
price 5555.5557
use list to handle
2
use Map to handle
OD00000000001 5555.5557
OD00000000002 3333.3333
OD0000001 Order(id=OD0000001, price=1111.1111)
OD0000002 Order(id=OD0000002, price=2222.2222)
```

# 参考
[(三) RabbitMQ实战教程(面向Java开发人员)之Spring整合RabbitMQ](https://blog.csdn.net/RobertoHuang/article/details/79541635)
《分布式消息中间件实践》