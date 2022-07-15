[TOC]

# kafka笔记

## 简述kafka架构设计

### 架构图

![img](https://github.com/lission/markdownPics/blob/main/kafka/kafka%E6%9E%B6%E6%9E%84%E5%9B%BE.png?raw=true)

### 核心概念

- **代理(Broker)**，kafka服务代理节点，一个kafka实例就是一个Broker，一个Broker可以容纳多个topic。

- **主题(Topic)**，可以理解为一个队列，topic将消息分类，生产者和消费者面向同一个topic

- **分区(Partition)**，***为了实现扩展性，提高并发能力***，一个topic可以在创建时指定分区，并将多个Partition分不到多个Broker上，每个Partition是一个有序队列。一个topic的每个Partition都有若干个副本(Replica)，一个Leader和若干个Follower。**生产者发送数据的对象，以及消费者消费的对象，都是Leader**。Follower负责实时从Leader中同步数据，保持和Leader数据的同步。Leader发生故障时，某个Follower会成为新的Leader。

- **消息(Message)**，消息是 Kafka 最基本的数据单元，`key-value` 格式。key 的主要作用是根据一定的策略，将消息路由到指定的分区中，这样就可以保证**同一 key 的消息全部写入同一分区中**。key 可以是 null，这种情况下消息会均衡地分布到不同的分区。

- **偏移量(Offset)**，消费者消费的位置信息，监控数据消费到什么位置。

- **生产者(Producer)**，生产者负责创建 Message，并将 Message 发送到特定的 Topic。会按照一定规则推送到指定分区，比如：轮询或根据消息的 key 哈希值。

- **消费者(Consumer)**，消费者订阅 Topic 或 Topic 的某些（或某一个）分区，接收 Message 进而执行相应的业务逻辑处理。消费者自己维护下一个要消费的消息偏移量（offset）。

- **消费者组(Consumer Group)**，消费者组内每个消费者负责消费不同分区的数据，提高消费能力。逻辑上是一个订阅者。同一个分组的消费者不会重复消费同一条消息。



# kafka面试

## kafka常见应用场景

kafka 官方定位是一个分布式流处理平台，在大数据场景下应用广泛。

kafka 还提供了发布订阅及Topic支持，常用作**分布式消息队列**使用。

kafka 适用于需要**高吞吐量但对顺序没有要求**的场景。

比如：日志收集、流式系统（流式处理数据传递给下游）、分布式消息队列（对顺序没有要求）、用户活动跟踪（分析用户行为数据，用于推荐系统或其他)、运营指标监控。