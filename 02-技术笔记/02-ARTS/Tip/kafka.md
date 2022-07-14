[TOC]

# kafka笔记

## 简述kafka架构设计

![img](https://github.com/lission/markdownPics/blob/main/kafka/kafka.jpg?raw=true)



# kafka面试

## kafka常见应用场景

kafka 官方定位是一个分布式流处理平台，在大数据场景下应用广泛。

kafka 还提供了发布订阅及Topic支持，常用作**分布式消息队列**使用。

kafka 适用于需要**高吞吐量但对顺序没有要求**的场景。

比如：日志收集、流式系统（流式处理数据传递给下游）、分布式消息队列（对顺序没有要求）、用户活动跟踪（分析用户行为数据，用于推荐系统或其他)、运营指标监控。