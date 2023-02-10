[TOC]

# kafka笔记

## 简述kafka架构设计

### 架构图

![img](https://github.com/lission/markdownPics/blob/main/kafka/kafka%E6%9E%B6%E6%9E%84%E5%9B%BE.png?raw=true)

### 核心概念

- **代理(Broker)**，kafka服务**代理节点**，一个kafka实例就是一个Broker，一个Broker可以容纳多个topic。

- **主题(Topic)**，可以理解为一个队列，**topic将消息分类**，**生产者和消费者面向同一个topic**

- **分区(Partition)**，***为了实现扩展性，提高并发能力***

  - 一个topic可以在**创建时指定分区**，并将多个Partition分布到多个Broker上

  - 每个Partition是一个**有序队列**。

  - **一个topic的每个Partition都有若干个副本(Replica)，一个Leader和若干个Follower**。

  - **生产者发送数据的对象，以及消费者消费的对象，都是Leader**。

  - Follower负责实时从Leader中**同步数据**，保持和Leader数据的同步。

  - Leader发生故障时，某个**Follower会成为新的Leader**。

- **消息(Message)**，消息是 Kafka**最基本的数据单元**，`key-value` 格式。

  - Key-value格式

    > ```java
    > //producer
    > public ProducerRecord(String topic, K key, V value) {
    >   this(topic, (Integer)null, (Long)null, key, value, (Iterable)null);
    > }
    > public ProducerRecord(String topic, V value) {
    >   this(topic, (Integer)null, (Long)null, (Object)null, value, (Iterable)null);
    > }
    > //Consumer
    > public ConsumerRecord(String topic, int partition, long offset, long timestamp, TimestampType timestampType, int serializedKeySize, int serializedValueSize, K key, V value, Headers headers, Optional<Integer> leaderEpoch) {
    >   if (topic == null) {
    >     throw new IllegalArgumentException("Topic cannot be null");
    >   } else if (headers == null) {
    >     throw new IllegalArgumentException("Headers cannot be null");
    >   } else {
    >     this.topic = topic;
    >     this.partition = partition;
    >     this.offset = offset;
    >     this.timestamp = timestamp;
    >     this.timestampType = timestampType;
    >     this.serializedKeySize = serializedKeySize;
    >     this.serializedValueSize = serializedValueSize;
    >     this.key = key;
    >     this.value = value;
    >     this.headers = headers;
    >     this.leaderEpoch = leaderEpoch;
    >   }
    > }
    > ```

  - **key 的主要作用是根据一定的策略，将消息路由到指定的分区中**，这样就可以保证**同一key的消息全部写入同一分区中**。

  - key **可以是null**，这种情况下消息会**均衡地分布到不同的分区**。

- **偏移量(Offset)**，消费者**消费的位置信息**，监控数据消费到什么位置。

- **生产者(Producer)**，生产者负责**创建Message**，并将Message发送到特定的Topic。会按照一定**规则推送到指定分区**，比如：轮询或根据消息的 key 哈希值。

- **消费者(Consumer)**，消费者**订阅Topic或 Topic的某些（或某一个）分区**，接收 Message进而执行相应的业务逻辑处理。消费者自己维护下一个要消费的**消息偏移量（offset）**。

- **消费者组(Consumer Group)**，消费者组内每个消费者负责**消费不同分区的数据，提高消费能力**。逻辑上是一个订阅者。同一个分组的消费者不会重复消费同一条消息。

- zookeeper，kafka集群能够正常工作，需要依赖于zookeeper，zookeeper帮助kafka**存储和管理集群信息**

- **日志(Log)**，消息被**写入分区**(Partition)时，实际是写入了**分区对应的Log中**。

  - 面对海量数据，为避免出现超大文件，每个日志文件大小都有限制，**默认为1G**（log.segment.bytes）。超出限制后，创建新的segment。
  - **kafka采用顺序I/O，只需向最新的segment追加数据**。

  > ![img](https://raw.githubusercontent.com/lission/markdownPics/33489de656f7fe3917a0376ddde8ed857faefc4f/kafka/kafkaWritePartition.jpg)

## kafka应用配置

```properties
# Kafka config
kafka.servers=kafka.local1.group:9092,kafka.local2.group:9092,kafka.local3.group:9092
kafka.consumer.group-id=warning
# 为了区分不同环境的topic，在原有topic上加后缀
kafka.topic.suffix=_develop
# Kafka配置
spring.kafka.bootstrap-servers=${kafka.servers}
# 生产者配置
## 消息发送重试次数
spring.kafka.producer.retries=0
## 生产者要求leader在考虑完成请求之前收到的确认数量，0无需等待确认 1确认写入本地log并立即确认 -1所有备份完成后确认
spring.kafka.producer.acks=1
## 批量发送的容量大小
spring.kafka.producer.batch-size=10684
## 缓冲区内存缓冲大小
spring.kafka.producer.buffer-memory=33554432
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
# 消费者配置
spring.kafka.consumer.group-id=${kafka.consumer.group-id}
spring.kafka.consumer.enable-auto-commit=true
spring.kafka.consumer.auto-commit-interval=100
spring.kafka.consumer.auto-offset-reset=latest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer

#Kafka Topic配置
# topic名称
kafka.topics[0].name=ORIGINAL
## 分区数
kafka.topics[0].num-partitions=3
## 副本因子
kafka.topics[0].replication-factor=3
```

# kafka面试

## kafka常见应用场景

- **官方定位**：是一个**分布式流处理平台**，在大数据场景下应用广泛。

- **kafka功能**：提供了**发布订阅及Topic支持**，常用作**分布式消息队列**使用。

- **适用场景**：适用于需要**高吞吐量但对顺序没有要求**的场景。

- **应用举例**：比如：**日志收集**、**流式系统**（流式处理数据传递给下游）、**分布式消息队列**（对顺序没有要求）、**用户活动跟踪**（分析用户行为数据，用于推荐系统或其他)、**运营指标监控**。

## kafka为何吞吐量大？速度为什么快？

1. **日志顺序读写**和**快速检索**

   - 日志以**分区(Partition)为单位保存**
   - **日志分段**，日志太大读写会变慢，日志超过限制会创建新的segment
   - 每个partition只支持**顺序读写**，顺序读写的情况下**磁盘I/O的速度可能比内存更快**
   - 每一个 segment 分 index 和 log 两个文件，**log 存储消息**，**index 存放消息的索引**
   - 日志读操作
     - 首先需要在存储的数据中找出 segment 文件
     - 然后通过全局 offset 计算出 segment 中的 offset
     - 通过 index 中的 offset 寻找具体数据内容
   - 日志写操作
     - 允许**串行的追加消息到文件最后**
     - 当日志达到阈值则**滚动到新文件**上

2. **Partition分区机制，一个 Topic 可以进行分区**，比如分区数量为 200，那么用 200 个消费者订阅 Topic，就可实现**消息并行处理**

3. **批量发送接收及数据压缩机制**，kafka生产者能够将多条消息**按照分区进行分组**，并采用批量的方式一次发送多条消息，并且对消息进行压缩。**牺牲一点额外延迟，尽可能多地收集消息后批量压缩发送，以此换来更高吞吐量**

4. 通过**系统函数sendfile实现零拷贝机制**，指消费者进行消息的消费时，数据不需要每次都经历 `从磁盘 -> 内核读取缓冲区 -> 用户缓冲区 -> socket 缓冲区 -> 网卡接口` 的过程，**数据与用户进程零交互**。

   > ![img](https://github.com/lission/markdownPics/blob/main/notes/%E9%9B%B6%E6%8B%B7%E8%B4%9D%E6%8A%80%E6%9C%AF.png?raw=true)

## kafka消费者和消费者组

单个 Partition 只能由消费者组中的某个消费者消费；

消费者组中的一个消费者可以消费多个 Partition，但**建议一个消费者消费一个 Partition**。

## kafka如何保证有序性

**kafka只支持单Partition有序，不支持全局有序**。一个Topic多个Partition是无序的。但是单Partition效率很低。

**使用key+offset可以做到业务有序**。

## 消息的传递保证

传递保证包含三个语义：

- **at most once**(最多一次)，消息**可能丢失**，但绝**不会重复传递**
- **at least once**(最少一次)，消息**绝不会丢失**，但**可能重复传递**（适用于多次消费也不影响结果的场景，**天然幂等性**）
- **exactly once**，每个消息**只会传递一次**
  - 生产者保证**不产生重复的消息**
  - 消费者**不能拉取重复的消息**

### 如何保证生产者不产生重复消息

**方案一**：

**每个分区只有一个生产者投递消息**，当出现异常或超时，生产者要查询此分区最后一个消息，用来决定后续操作是消息重传还是继续发下一个消息。

- **生产者重试次数配置为0**

```properties
# 生产者配置
# 重试次数,必须设置为 0（默认就是 0），防止因网络波动（消息已经发送到 broker 但没收到响应）导致重复发送同一条消息
## 消息发送重试次数
spring.kafka.producer.retries=0
```

- **拦截器实现逻辑**，前提**用于每个分区只有 1 个生产者投递消息**，消息**投递失败回调时**，**查询==分区的最后一个消息的偏移量==，判断是否大于投递失败的消息的偏移量**，如果大于，表示消息已经成功投递到 kafka broker，可能是因为网络波动没有收到响应才失败的，这种情况应该被认定为消息成功投递。

**方案二(推荐)**:

为每个消息添加一个**全局唯一的主键**，生产者不做特殊处理，由**消费者对消息进行去重**，实现exactly once。

> 如果业务数据产生消息可以找到合适的字段作为主键，或是有一个**全局ID生成器**，可以优先考虑选用方案二。

### 如何确保消费者不会重复拉取相同信息

**方案一**：

- 不使用kafka的自动提交功能，**自动提交配置关闭**

```properties
# 消费者配置
# 确保自动提交 offset 关闭
spring.kafka.consumer.enable-auto-commit=false
```

- **使用事务，将提交offset和消息处理的逻辑放在一个事务里**，offset的提交使用关系型数据库或其他NoSQL数据库，**当消费者出现宕机或rebalance时，消费者可以从数据库中找到对应的offset，然后调用KafkaConsumer.seek()方法==手动设置消费位置==，从宕机前的offset开始继续消费**。
- **难点-rebalance**，消费者并不知道 Consumer Group 什么时候会发生 Rebalance 操作，哪个分区分配给了哪个消费者消费?可以通过订阅主题时添加自定义的 ConsumerRebalanceListener 解决

## kafka批量发送和批量拉取如何保证消息不丢失

### Producer 批量发送如何避免消息丢失

**方案一**：

事务，kafka从0.11版本开始提供了事务支持，**事务型Producer能保证消息原子性写入多个分区。批量消息要么全部成功，要么全部失败**。

### Consumer 批量拉取如何避免消息丢失

consumer支持**批量拉取**，可以提高消费效率。

第1种、消费端业务支持幂等性，**禁止kafka自动提交，批量消费完成后，手动提交**
第2种、将自动提交关闭使用事务，**将offset和消息处理逻辑放在一个事务中**，offset的提交可以使用关系型数据库，当消费者发生宕机时可以从数据库找到对应offset，然后可以手动设置消费位置

## kafka选举机制

Leader挂了就从**ISR(当前可用的副本集合)**中选择一个作为leader，直接抢**先到先得**。如果ISR中的副本全部都宕机了。kafka会进行 **unclean leader选举（脏节点选举，数据丢失）**。应该要尽量避免脏节点选举，监控系统应该要及时发现宕机的kafka并及时处理。

选举配置建议：

- 禁用unlean leader选举
- 手动指定最小ISR，比如ISR小于2就不接收消息，让业务系统去处理异常。

## kafka是pull还是push，优劣分析

kafka中使用**pull模式，consumer主动pull**。

**pull模式**：

- **根据consumer消费能力进行数据拉取**，可以**控制速率**
- 可以**批量拉取也可以单条拉取**
- 可以设置**不同的提交方式**，实现不同的传递语义

缺点：如果kafka没有数据，会导致**consumer空循环，消耗cpu资源**

解决：通过参数设置，consumer拉取**数据为空**或**没有达到一定数量**时**阻塞**

**push模式**：

不会导致consumer循环等待

缺点：速率固定，忽略了consumer的消费能力，可能导致拒绝服务或者网络阻塞等情况。

kafka producer客户端执行流程





## 消息中间件中RocketMQ与Kafka的存储区别

**存储**

说到存储，其实效率才是最主要的，容量不是我们关心的，但是说到存储，不只是mq，所有需要高效率的存储其实最后利用的核心都是一样的。

1. 随机写转换成顺序写
2. 集中刷盘

**为什么随机写要转换为顺序写？**

1.现在主流的硬盘是机械硬盘

2.机械硬盘的机械结构一次读写时间 = 寻道时间 + 旋转延迟 + 读取数据时间

那么寻道时间比较长，如果是顺序写，只需要一次寻道时间，关于机械硬盘整个过程，读者可自行google。

**为什么集中刷盘？**

因为每次刷盘都会进行系统调用，第二还是跟硬盘的本身属性有关，无论是机械硬盘还是ssd按照一定块刷盘会比小数据刷盘效率更好

**kafka**

为什么先说kafka的存储，因为kafka是第一个高性能的消息中间件，其中rocketmq也是借鉴于它，所以我们先说

​    ![0](https://note.youdao.com/yws/res/1056/EFBEA5B15E964516AAEDD6F1E47C45E3)

模型变化图

​    为什么引入消费组概念？

​    上一次模型图我们还没有消费组，那么引入消费组，是因为现在一个服务都有很多实例在运行，消费组是对这群一群机器的一个划分，他还是一个概念而已。

​    mq内部也发生了变化，一个topic后面又对应了很多partition，partition也是一个概念，他只不过是把一个topic分成了很多份，每一份叫一个partition，你高兴也可以叫他xxx,那么我们来说说为什么要分成很多份，一份不行吗？

​    因为现在一个服务有很多实例在运行，如果topic只有一份的话，那么所有的实例都会来消费消息，并且都是抢占我们一个topic，这不可避免引入了多实例竞争，以及他们之间怎么协调，一堆问题需要关注解决，现在我把topic分成了很多份，每一份只给一个实例，那么就不会引入各实例之间的竞争问题了，简化了mq的问题。

​    生产组的引入也是一样的，只不过是一组机器的一个概念，一个逻辑的划分，生产者发送消息原先是发往topic，那么现在topic分成了很多份，生产者发送消息，需要说明发往哪个partition或者随意分配都可以，只不过最终发送的消息，会到一个topic下的一份里面。无论使用哪种映射方式都可以。

那么模型出来了，我们说说存储的问题。

对于kafka，一个partition对应一个文件，每次消息来都是顺序写这个文件。并且是定时刷盘，而不是每次写都刷盘，所以kafka的写非常高效。

**RocketMQ**

​    ![0](https://note.youdao.com/yws/res/1054/4B1EF02E508F473EA043A2E63ACE13CA)

**rocketmq借鉴于kafka，所以存储借鉴了kafka，但是rocketmq不是仅仅把partition改成了ConsumeQueue，原先kafka，里面partition存储的是整个消息，但是现在ConsumeQueue里面是存储消息的存储地址，但是不存储消息了；**

​    ![0](https://note.youdao.com/yws/res/1057/6FDB5768ED344F6880BEA952CB44FFFF)

**现在每个ConsumeQueue存储的是每个消息在commitlog这个文件的地址，但是消息存在于commitlog中**

**也就是所有的消息体都写在了一个文件里面，每个ConsumeQueue只是存储这个消息在commitlog中地址**

**存储对比**

消息体存储的变化

那么我们先来看看kafka，假设partition有1000个，一个partition是顺序写一个文件，总体上就是1000个文件的顺序写，是不是就变成了随机写，所以当partition增加到一定数目后，kafka性能就会下降。而rocketmq是把消息都写到一个CommitLog文件中，所以相当于一个文件的顺序写;

为什么索引文件(ConsumeQueue)的增加对性能影响没有那么partition大？

(kafka也有索引文件，在这里只是想说明索引文件的增加跟partition增加的区别)

虽然rocketmq是把消息都写到一个CommitLog文件中，但是按照上面的实例会有1000个ConsumeQueue，也就是一千个文件，那么为什么就没有把顺序写变成随机写，带来性能的下降呢？首先就要介绍linux的pagecache

​    ![0](https://note.youdao.com/yws/res/1055/5DC71EF20825498EA76E8C4A9C9E3D56)

我们平常调用write或者fwrite的时候，数据还没有写到磁盘上，只是写到一个内核的缓存(pagecache)，只有当我们主动调用flush的时候才会写到硬盘中。或者需要回写的pagecache占总内存一定比例的时候或者一个应该回写的page超过一定时间还没有写磁盘的时候，内核会将这些数据通过后台进程写到磁盘中(总结就是达到一定比例，或者多长时间还没有回写，会被内核自动回写)。

然后我们现在来看看为什么大量索引文件的顺序写没有像partition一样导致性能明显下降。ConsumeQueue只存储了（CommitLog Offet + Size + Message Tag Hashcode），一共20个字节，那么当commitlog定时任务刷盘之后，应该回写的pagecache的比例就会下降很多，那么ConsumeQueue的部分可以不用刷盘，就相当于ConsumeQueue的内容会等待比较长的时间聚合批量写入，而kafka每个partition都是存储的消息体，因为消息体都相对较大，基本在kb之上。

当一个partition刷盘的时候，应该回写的pagecache的比例降低的并不多，不能阻止其他partition的刷盘，所以会大量存在多个partition同时刷盘的场景，变成随机写。但是rocketmq消息都会写入一个commitLog，也就是顺序写。

**总结**：

1、consumerQueue消息格式大小固定（20字节），写入pagecache之后被触发刷盘频率相对较低。就是因为每次写入的消息小，造成他占用的pagecache少，主要占用方一旦被清理，那么他就可以不用清理了;

2、kafka中多partition会存在随机写的可能性，partition之间刷盘的冲撞率会高，但是rocketmq中commitLog都是顺序写。