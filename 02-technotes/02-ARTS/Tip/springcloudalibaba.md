[TOC]

# spring cloud alibaba

spring cloud alibaba 是对 spring cloud 的标准实现，以微服务为核心的整体解决方案。
主流框架，[spring cloud alibaba版本说明](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)

**spring cloud 各套实现对比**：

![img](https://github.com/lission/markdownPics/blob/main/spring/springcloud%E7%BB%84%E4%BB%B6.jpeg?raw=true)

![img](https://github.com/lission/markdownPics/blob/main/spring/springcloudalibaba%E6%9E%B6%E6%9E%84.jpeg?raw=true)

## 1.1、nacos

是一个动态**服务发现(Nacos Discovery)**、**服务配置(Nacos Config)**和服务管理平台，支持几乎所有主流类型的**“服务”的发现、配置和管理：**

[Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)

[gRPC](https://grpc.io/docs/guides/concepts.html#service-definition) & [Dubbo RPC Service](https://dubbo.incubator.apache.org/)

[Spring Cloud RESTful Service](https://spring.io/projects/spring-restdocs)

集注册中心+配置中心+服务管理的平台



### 1.1.1、ncos discovery 注册中心

注册中心管理所有微服务，解决服务之间调用关系复杂，难以维护的问题。

**核心功能**：

- 服务注册，nacos client通过发送rest请求方式向nacos server注册自己的服务，提供自身的元数据，比如ip、端口等，nacos server接收到注册请求后，把元数据信息存储在双层内存map中
- 服务心跳，服务注册后，nacos client维护一个定时心跳持续通知nacos server，默认5s发生一次心跳
- 服务同步，集群之间服务实例同步
- 服务发现，服务消费者(nacos client)在调用服务提供者的服务时，发送一个rest请求给nacos server，获取注册的服务清单，缓存在nacos client本地。同时在nacos client本地开启一个定时任务定时拉取服务端最新的注册表信息更新到本地缓存
- 服务健康检查，nacos server开启一个定时任务检查注册服务实例健康状况，对于超过15s没有收到客户端心跳的实例将它的healthy属性置为false，如果某个实例超过30s没有收到心跳，直接剔除该实例(被剔除的实例如果恢复发送心跳会重新注册)

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

**主流注册中心区别**

CAP,C一致性、A可用性、P分区容错性

CP或AP

![img](https://github.com/lission/markdownPics/blob/main/spring/springcloud%E6%B3%A8%E5%86%8C%E4%B8%AD%E5%BF%83%E5%AF%B9%E6%AF%94.jpeg?raw=true)

### 1.1.1、ncos config 配置中心

提供用于存储配置和其他元数据的 key/value 存储

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

## 1.2、Ribbon

主流负载均衡方案：

- **集中式(服务端)负载均衡**，**在消费者和服务端中间使用独立的代理方式进行负载**，**硬件（F5），软件（nginx）**

- **客户端负载均衡**，客户端根据自己的请求情况做负载均衡，**ribbon属于客户端自己做的负载均衡**

ribbon，微服务负载均衡器。通过load balance 拉取服务列表，基于某种规则调用这些实例。

常见负载均衡算法：

- 轮询
- 随机
- 加权，服务提供者权重越高分配给它的流量越多
- 地址哈希，根据服务消费者ip进行hash取模进行服务器调度
- 最小连接数

使用ribbon在RestTemplate上添加@LoadBalanced注解。

**可以为所有服务提供一个负载均衡策略，也可以为指定服务提供负载均衡策略**。所有的负载均衡策略均实现了IRule接口。

Ribbon默认负载均衡策略ZoneAvoidanceRule(默认规则)，复合判断server所在区域的性能和server的可用性选择服务器。**区域：云原生中地域部署。如果没有区域概念，采用轮询规则**。



## 1.3、spring cloud loadBalancer

spring cloud官方自己提供的客户端负载均衡器，用来替换ribbon。目前只有轮询、随机的策略

spring cloud 提供了两种负载均衡的客户端：

- RestTemplate，**spring提供的用于访问rest服务的客户端**，RestTemplate提供多种便捷远程访问http服务的方法，默认情况下，RestTemplate默认依赖jdk的http工具
- WebClient，spring webflux 5.0开始提供的一个非阻塞的基于响应式编程的进行http请求的工具。响应式编程基于Reactor。WebClient提供了标准http请求方式对应的get、post、put、delete等方法

## 1.4、feign

netflix开发的声明式、模板化的http客户端，声明在客户端，整合了负载均衡器，可以帮助我们更加便捷、优雅地调用http api，优化服务间的调用。

spring cloud openfeign对feign进行了增强，使其支持spring MVC，整合了ribbon和nacos，使feign使用更加方便。

feign可以做到**使用http请求远程服务时就像调用本地方法一样**。

@FenginClient 动态代理

feign自定义拦截器

## 1.5、nacos config 配置中心

https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config

维护性、实效性、安全性

通过@Value注解可以获取到配置中心的值，但是无法动态感知修改后的值，需要利用@RefreshScope注解

## 1.6、sentinel 服务熔断

**解决微服务的可用性问题，实现高可用**

解决服务雪崩问题，服务雪崩效应：因服务提供者不可用导致服务调用者不可用，并将不可用过程逐渐扩大的过程，叫做服务雪崩效应。

常见容错机制：

- 超时机制，一旦超时，释放资源
- 限流机制，访问量超过限制，进行限流 ==服务提供端==
- 隔离，根据线程隔离、根据信号隔离（为服务设置并发限制）
- 服务熔断，==服务消费端==设置熔断机制，调用某个服务持续得不到响应，对该服务停止访问
- 服务降级，==服务消费端==服务不可用时，区分强依赖、弱依赖（如秒杀中的积分服务等），对弱依赖服务进行降级处理（后续在进行积分处理等）



sentinel 是阿里巴巴开源的，面向分布式服务架构的高可用防护组件。具备多维度的流控降级能力，**整合openfeign进行服务降级**

https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D

https://github.com/alibaba/Sentinel/wiki/%E5%9C%A8%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E4%B8%AD%E4%BD%BF%E7%94%A8-Sentinel

持久化模式：原始、pull（文件）、push(配置中心如nacos)



## 1.7、Seata 分布式事务

阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。

https://seata.io/zh-cn/docs/overview/what-is-seata.html



## 1.8、 spring cloud gateway

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/



## 1.9、 skywalking

官网网址：https://skywalking.apache.org/



中文文档：https://skyapm.github.io/document-cn-translation-of-skywalking/

链路追踪