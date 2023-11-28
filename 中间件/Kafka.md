## 消息队列

### 消息队列的两种模式

- **点对点模式**（一对一，消费者主动拉取数据，消息收到后消息清除）
  - 生产者生产消息发送到Queue中，然后消费者从Queue中取出并且消费消息。消息被消费以后，queue 中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue 支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。

- **发布订阅模式**（一对多，消费者消费数据之后不会清除消息）
  - 生产者将消息**发布**到 topic 中，同时有多个消费者**订阅**该消息。和点对点方式不同，发布到 topic 的消息会被所有订阅者消费。
  - Kafka 采取拉取模型，由自己控制消费速度，以及消费的进度，消费者可以按照任意的偏移量进行消费。

### 为什么使用消息队列

主要有三个核心场景：**解耦**、**异步**、**削峰**。

#### 解耦

在这个场景中，A 系统跟其它各种乱七八糟的系统严重耦合，A 系统产生一条比较关键的数据，很多系统都需要 A 系统将这个数据发送过来。

![mq-1](https://doocs.github.io/advanced-java/docs/high-concurrency/images/mq-1.png)

通过一个 MQ，Pub/Sub 发布订阅消息这么一个模型，A 系统就跟其它系统彻底解耦了。

一个系统或者一个模块，调用了多个系统或者模块，互相之间的调用很复杂，维护起来很麻烦。但是其实这个调用是不需要直接同步调用接口的，如果用 MQ 给它异步化解耦，也是可以的。

![mq-2](https://doocs.github.io/advanced-java/docs/high-concurrency/images/mq-2.png)

#### 异步

![mq-3](https://doocs.github.io/advanced-java/docs/high-concurrency/images/mq-3.png)

A 系统接收一个请求，需要在自己本地写库，还需要在 BCD 三个系统写库，自己本地写库要 3ms，BCD 三个系统分别写库要 300ms、450ms、200ms。最终请求总延时是 3 + 300 + 450 + 200 = 953ms。

![mq-4](https://doocs.github.io/advanced-java/docs/high-concurrency/images/mq-4.png)

如果使用 MQ，那么 A 系统连续发送 3 条消息到 MQ 队列中，假如耗时 5ms，A 系统从接受一个请求到返回响应给用户，总时长是 3 + 5 = 8ms。

#### 削峰

在高峰期每秒并发请求数量会到 5k+ 条。一般的 MySQL，扛到每秒 2k 个请求就差不多了，如果每秒请求到 5k 的话，可能就直接把 MySQL 给打死了，导致系统崩溃，用户也就没法再使用系统了。

![mq-6](https://doocs.github.io/advanced-java/docs/high-concurrency/images/mq-6.png)

只要高峰期一过，A 系统就会快速将积压的消息给解决掉。

### 消息队列有什么缺点

- 系统可用性降低
  - 系统引入的外部依赖越多，越容易挂掉
- 系统复杂度提高
  - 要保证消息没有重复消费、处理消息丢失的情况、保证消息传递的顺序性
- 一致性问题
  - A 系统处理完了直接返回成功了，但是BCD 三个系统那里，BD 两个系统写库成功了，结果 C 系统写库失败了，导致数据就不一致



## Kafka

### Kafka 的基本架构

![img](http://img.godjiyi.cn/csdnblogkafka-arc.jpg)

Kafka 集群由多个 broker 组成，每个 broker 是一个节点。你创建一个 topic，这个 topic 可以划分为多个 partition，每个 partition 可以存在于不同的 broker 上，每个 partition 就放一部分数据。

#### Topic（主题）

主题是一个逻辑上的概念。不同的生产者将不同的业务消息分发到不同的 **topic** 上，这样，消费者就可以根据 **topic** 进行对应的业务消息消费了。

#### Broker

- 一个 broker 就是一个 kafka 节点，多个 broker 构成一个 kafka 集群。

#### Partition（分区）

- 一个**主题**分成多个**分区**，一个**分区**只属于单个**主题**，，很多时候也会把分区称为主题分区（Topic-Partition），每个 partition 是一个有序的消息序列
- 同一主题下的不同分区包含的消息是不同的，分区在存储层面可以看作一个可追加的日志（Log）文件，消息在被追加到分区日志文件的时候都会分配一个特定的偏移量（offset）
- 多个 **producer** 生产消息可以并行入队，多个 **consumer** 可并行消费
- 同一个 **partition** 里保证消息有序，不同 **partition** 则不能完全保证有序
  - offset 是消息在分区中的唯一标识，Kafka 通过它来保证消息在分区内的顺序性，不过 offset 并不跨越分区，也就是说，Kafka 保证的是分区有序而不是主题有序。

> 选择分区的原则
>
> 1. 指定了 partition
> 2. 没有指定 partition 但设置了 key，则根据 key 的值 hash 出一个 partition
> 3. 没有指定分区，没有设置 key，则轮询各分区发送，即每次取一小段时间的数据写入某个 partition，下一小段时间的数据写入下一个 partition

#### Replica

Replica（副本）指的是一个分区（Partition）的备份。

![img_7654f69f203c098f49c7054dc84fc1b9.png](https://yqfile.alicdn.com/img_7654f69f203c098f49c7054dc84fc1b9.png)

简单的replica分配示意图（圆角矩形代表replica）

这种分配保证了，任何一台机器挂掉，kafka集群依然有备份可用。

分为 **Leader**（主节点，1） 和 **Follower**（从节点，N）两种角色。

生产者发送数据的对象，以及消费者消费数据的对象都是 leader。

follower 实时从 leader 中同步数据，保持和 leader 数据的同步。leader 发生故障时，某个 follower 会成为新的 leader。

#### Consumer Group

消费者组就是消费者组成的一个组，消费者在向 Kafka 拉取（pull）数据的时候需要提供一个组名，这个名称就是消费者组名。两种消息模式都可以在消费者组中得到实现。

> consumer 采用 pull（拉）模式从 broker 中读取数据。
>
> push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker 决定的。而pull 模式则可以根据 consumer的消费能力以适当的速率消费消息。

1. **点对点/队列模式**：一个消息只能被一个消费者消费，我们只需要将这些消费者放在同一个消费者组里就可以了，这样消费者在同一个组中，那么 topic 中的一条消息只会向一个消费者组发送一次；
2. **发布-订阅模式**：一个消息可被多个消费者消费，这种情况，我们只需要将各个消费者放在各自单独的组中，各个组均订阅了此消息 topic 就可以了。

注意点：

- 一个消费组消费一个 **topic** 的全量数据；
- 组内消费者消费一个或多个 **partition** 数据，如果一个组里的消费者数量少于订阅的 topic 的 partition 数量，那么组中必有一个消费者要消费多个 partion 数据；
- 一个组里的消费者应小于等于 **topic** 的 **partition** 数量，这是因为一个 partition 最多只能与一个 consumer 连接，那么如果 partition 数量大于 consumer 数量，则必定有 consumer 是空闲的，因此尽量避免这种情况；

### 生产者往kafka发送数据的流程

![image-20231025120538386](https://gitee.com/Transmigration_zhou/pic/raw/master/image-20231025120538386.png)

1. 生产者从 Kafka 集群获取分区 leader 信息
2. 生产者将消息发送给 leader
3. leader 将消息写入本地磁盘
4. follower 从 leader 拉取消息数据
5. follower 将消息写入本地磁盘后向 leader 发送 ACK
6. leader 收到所有的 follower 的 ACK 之后向生产者发送 ACK

### Kafka如何保证消息有序性

两种方案：

- 方案一：kafka topic 只设置一个 partition 分区
- 方案二：producer 将消息发送到指定同一个 partition 分区

方案一：kafka 默认保证同一个 partition 分区内的消息是有序的，则可以设置 topic 只使用一个分区，这样消息就是全局有序，缺点是只能被 consumer group 里的一个消费者消费，降低了性能，不适用高并发的情况。

方案二：既然 kafka 默认保证同一个 partition 分区内的消息是有序的，则 producer 可以在发送消息时可以指定需要保证顺序的几条消息发送到同一个分区，这样消费者消费时，消息就是有序。

![方案二](https://img2018.cnblogs.com/i-beta/727602/202001/727602-20200113155600857-1227819328.png)

**总结：每个分区内的消息是有序的，而消费者组确保每个分区只由一个消费者处理，从而保证了消费整体上的顺序性。**

