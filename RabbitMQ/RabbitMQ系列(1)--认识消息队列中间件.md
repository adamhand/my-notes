# RabbitMQ系列(1)--认识消息队列中间件
---
# 初识消息队列中间件
消息(Message)是指在应用之间传递的数据。

消息队列中间件(Message Queue Middleware，MQ)是一种“中间件”，这个“中间”有两层含义：一种是**上下层之间的中间(纵向的“中间”)**，一种是**左右之间的中间（横向的“中间”）**。

上下层之间的中间是指，消息中间件被夹在“应用层”和“传输层”之间，它对传输层的协议进行了封装，为上层应用程序提供跨平台的数据交流。

左右之间的中间是指，消息中间件常常被用于分布式环境中，它可以为两个分布式节点之间提供通信服务。

# 传递模式
消息中间件一般有两种传递模式：**点对点**(P2P，Point to point)模式和**发布/订阅**(Pub/Sub)模式。

## 点对点模式
点对点模式是基于队列(queue)的，消息生产者生产消息发送到queue中，然后消息消费者从queue中取出并且消费消息。队列的存在使得消息的异步传输成为可能。消息被消费以后，queue中不再存储，所以消息消费者不可能消费到已经被消费的消息。

Queue支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。

示意图如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mq-p2p.jpg">
</center>

## 发布/订阅模式
Pub/Sub发布订阅（广播）：使用topic作为通信载体。消息生产者（发布）将消息发布到topic中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费。

示意图如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mq-pub-sub.jpg">
</center>

# 常用协议
## AMQP
AMQP即**Advanced Message Queuing Protocol(高级队列消息协议)**。AMQP协议定义了消息路由规则和方式。生产者将消息发送到Broker，Broker中的Exchange充当了路由器的角色，由它将消息路由到一个或者多个队列中。如果路由不到，或许会返回给生产者，获取会丢弃。

多个消费者可以订阅同一个队列中的消息，这是队列中的消息会通过“轮询”的方式平均分配给每个消费者，而不是每个消费者都会消费所有消息。

AMQP的模型如下所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/amqp.jpg">
</center>

## JMS
Java消息服务（Java Message Service，JMS）应用程序接口是一个Java平台中关于面向消息中间件（MOM）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。点对点与发布订阅最初是由JMS定义的。

严格来说，JMS不是一种消息队列协议，它是一种跨平台的API，换句话说，可以使用JMS API来连接支持AMQP、STOMP等协议的消息中间件产品，它的作用更像JDBC，我们可以使用JDBC API来连接具体的数据库产品。

# 常见消息中间件对比
待续。

---
# 补丁：消息的消费模式
消息的消费模式有两种：**推模式(push)**和**拉模式(pull)**。

**推(push)模式**是一种基于客户器/服务器机制、由服务器主动将信息送到客户器的技术。在push模式应用中，服务器把信息送给客户器之前，并没有明显的客户请求。push事务由服务器发起。push模式可以让信息主动、快速地寻找用户/客户器，信息的主动性和实时性比较好。但精确性较差，可能推送的信息并不一定满足客户的需求。推送模式不能保证能把信息送到客户器，因为推模式采用了广播机制，如果客户器正好联网并且和服务器在同一个频道上，推送模式才是有效的。push模式无法跟踪状态，采用了开环控制模式，没有用户反馈信息。在实际应用中，由客户器向服务器发送一个申请，并把自己的地址（如IP、port）告知服务器，然后服务器就源源不断地把信息推送到指定地址。在多媒体信息广播中也采用了推模式。另外，如手机*、qq广播。

**拉（pull）模式**与推模式相反，是由客户器主动发起的事务。服务器把自己所拥有的信息放在指定地址（如IP、port），客户器向指定地址发送请求，把自己需要的资源“拉”回来。不仅可以准确获取自己需要的资源，还可以及时把客户端的状态反馈给服务器。

# 参考
《RabbitMQ实战指南》
《分布式消息中间件实战》
[消息队列中点对点与发布订阅区别](https://blog.csdn.net/lizhitao/article/details/47723105)
[消息中间件（一）MQ详解及四大MQ比较](https://blog.csdn.net/wqc19920906/article/details/82193316)
[Java消息中间件(入门篇)](https://blog.csdn.net/btt2013/article/details/80211599)
[消息模式--推模式和拉模式](https://blog.csdn.net/zeng_z/article/details/77246368)