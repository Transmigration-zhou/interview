## 工作模式

1. 简单模式 HelloWorld

   一个生产者、一个消费者，不需要设置交换机（使用默认 direct 的交换机）

   ![(P) -> [|||] -> (C)](https://www.liwenzhou.com/images/Go/rabbitmq/tutorials01/python-one.png)

2. 工作队列模式 Work Queue

   一个生产者、多个消费者（竞争关系），不需要设置交换机（使用默认 direct 的交换机）。

   有两种消息分发方式：

   轮询分发：一个消费者消费一条，按均分配，woek模式下默认是采用轮询分发方式。轮询分发就不写代码演示了，比较简单，比如生产费者发送了6条消息到队列中，如果有3个消费者同时监听着这一个队列，那么这3个消费者每人就会分得2条消息。

   公平分发：根据消费者的消费能力进行公平分发，处理得快的分得多，处理的慢的分得少，能者多劳。
   

   ![img](https://www.liwenzhou.com/images/Go/rabbitmq/tutorials02/python-two.png)

3. 发布订阅模式 Publish/subscribe

   需要设置类型为 fanout（扇出）的交换机，并且交换机和队列进行绑定，当发送消息到交换机后，它将接收到的所有消息广播到它知道的所有队列中。

   ![img](https://www.liwenzhou.com/images/Go/rabbitmq/tutorials03/exchanges.png)

4. 路由模式 Routing

   需要设置类型为 direct 的交换机，交换机和队列进行绑定，并且指定 routing key，当发送消息到交换机后，交换机会根据 routing key 将消息发送到对应的队列。

   ![img](https://www.liwenzhou.com/images/Go/rabbitmq/tutorials04/python-four.png)

5. 通配符模式 Topic

   需要设置类型为 topic 的交换机，交换机和队列进行绑定，并且指定通配符方式的 routing key，当发送消息到交换机后，交换机会根据 routing key 将消息发送到对应的队列。

   > `*`代替一个单词
   >
   > `＃`替代零个或多个单词

   ![img](https://www.liwenzhou.com/images/Go/rabbitmq/tutorials05/python-five.png)

6. RPC模式 

   支持生产者和消费者不在同一个系统中，即允许远程调用的情况。

   ![img](https://www.liwenzhou.com/images/Go/rabbitmq/tutorials06/python-six.png)

- 客户端启动时，它将创建一个匿名排他回调队列。
- 对于RPC请求，客户端发送一条消息，该消息具有两个属性：`reply_to`（设置为回调队列）和`correlation_id`（设置为每个请求的唯一值）。
- 该请求被发送到`rpc_queue`队列。
- RPC工作程序（又名：服务器）正在等待该队列上的请求。当出现请求时，它会完成计算工作并把结果作为消息使用`replay_to`字段中的队列发回给客户端。
- 客户端等待回调队列上的数据。出现消息时，它将检查`correlation_id`属性。如果它与请求中的值匹配，则将响应返回给应用程序。



### 消息投递的过程

一个消息从生产者发送到RabbitMQ服务器，需要经历这么几个步骤：

1. 生产者准备好需要投递的消息。（生产消息）
2. 生产者与RabbitMQ服务器建立连接。（建立连接）
3. 生产者发送消息。（发送消息）
4. RabbitMQ服务器接收到消息，并将其路由到指定队列。（路由消息）
5. RabbitMQ服务器发起回调，告知生产者消息发送成功。 所谓可靠投递，就是确保消息能够百分百从生产者发送到服务器。（回调确认）



### RabbitMQ怎么保证消息不丢失

https://blog.csdn.net/w20001118/article/details/126595970

TODO：针对生产者，MQ，消费者三个角度考虑

1. 消息持久化：RabbitMQ支持将消息写入磁盘，以确保即使服务器重启或者崩溃时，消息也不会丢失。
2. 生产者确认：生产者确认是指在消息被发送到队列之后，RabbitMQ会给生产者一个确认信号。如果没有收到确认信号，生产者可以重试发送消息。
3. 消费者确认：消费者确认是指在消息被消费者处理之后，消费者会向 RabbitMQ 发送一个确认信号。如果 RabbitMQ 没有收到确认信号，它会重新将这条消息加入队列，直到收到确认信号。
4. 事务机制：RabbitMQ 支持事务机制，可以通过开启事务来确保消息不会在发送过程中丢失。但是，事务机制的性能较差，因此不建议在高并发场景下使用。
5. 设置集群镜像模式