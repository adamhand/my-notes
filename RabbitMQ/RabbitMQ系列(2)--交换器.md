# RabbitMQ系列(2)--交换器
---
# 几个概念

- QUeue：队列。用于存储消息。
- Exchange：交换器。用于将生产者生产的消息路由到一个或多个队列中。
- Routing Key：路由键。用于指定某个消息的路由规则。Routing Key需要和Binding Key联合使用
- Binding：绑定。RabbitMQ中通过绑定将交换器和队列关联起来，在绑定的时候一般会指定一个绑定建(Binding Key)，这样交换器就会知道如何将消息路由到队列了。在绑定多个队列到一个交换器的时候，允许使用相同的Binding Key。

**可以用一个形象的比喻来描述路由过程**：**交换器**相当于投递包裹的邮箱，**Routing Key**相当于填写在包裹上的地址，**Binding Key**相当于包裹的目的地址，当填写在包裹上的地址和包裹的实际地址相匹配时，这个包裹才会被正确投递到目的地。这个包裹的“主人”——**Queue**可以保留这个包裹。

如果填写的地址出错，没有找到匹配的目的地址，包裹便不能被正确投递，有可能会还给寄件人，有可能会被丢弃。

# 交换器类型
RabbitMQ常用的交换器类型有fanout、direct、topic、headers这四种。AMQP协议里还有另外两种类型：System和自定义，但是不是很常用。

## fanout
会将所有发送到该交换器的消息路由到所有与该交换器绑定的队列中。很像子网广播，每台子网内的主机都获得了一份复制的消息。Fanout交换机转发消息是最快的。 

## direct
direct类型的交换机会将消息路由到那些Binding Key和Routing Key**完全匹配**的队列中。

如下图所示，交换器的类型为direct，如果某条消息的路由键为“warning”，则消息会路由到Queue1和Queue2中；如果设置路由键为“info”或者“debug”，消息只会路由到Queue2。

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/direct-exchange.PNG">
</center>

## topic
direct中的路由方式是RoutingKey和BindingKey的**完全匹配**，而topic提供了一种更为宽松的“**模糊匹配**”方式。

在topic中，每一个RoutingKey和BindingKey都做是一个被“.”分隔的字符串（被“.”分隔的每一个独立的字符串都成为一个**单词**）。比如“java.util.concurrent”。

BindingKey中可以存在两种特殊字符串“`*`”和“`#`”用作模糊匹配。其中**“`*`”用于匹配一个单词，“`#`”用于匹配多个单词，也可以是0个**。

如下图所示，

- 路由键"com.rabbitmq.client"的消息只会被路由到Queue1和Queue2
- 路由键"com.hiddin.client"的消息只会被路由到Queue2
- 路由键"com.hiddin.demo"的消息只会被路由到Queue2
- 路由键"java.rabbitmq.demo"的消息只会被路由到Queue1
- 路由键"java.util.concurrent"的消息会被丢弃或返回给生产者，具体需要设置mandatory参数

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/topic-exchange.PNG">
</center>

## headers
headers属性的交换器不依赖与路由键的规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配。

在绑定队列和交换器的时候指定一组键值对，当发送消息到交换机时，RabbitMQ会获得消息中的headers(也是一个键值对)属性，对比其中的键值对是否完全匹配队列和交换机绑定时指定的键值对，如果完全匹配会将消息路由到该队列，否则不会进行路由。

headers交换机的性能很差且不实用，基本上不会被用到。

# 参考
《RabbitMQ实战指南》
[【RabbitMQ】三种类型交换器 Fanout,Direct,Topic](https://blog.csdn.net/fxq8866/article/details/62049393/)