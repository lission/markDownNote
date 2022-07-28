[TOC]

# spring cloud alibaba

spring cloud alibaba 是对 spring cloud 的标准实现，以微服务为核心的整体解决方案。
主流框架，[spring cloud alibaba版本说明](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)

## 1.1、nacos

集 注册中心+配置中心+服务管理 平台，支持几乎所有主流类型的**“服务”的发现、配置和管理：**

[Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)

[gRPC](https://grpc.io/docs/guides/concepts.html#service-definition) & [Dubbo RPC Service](https://dubbo.incubator.apache.org/)

[Spring Cloud RESTful Service](https://spring.io/projects/spring-restdocs)

### 1.1.1、ncos discovery 注册中心

注册中心管理所有微服务，解决服务之间调用关系复杂，难以维护的问题。Nacos

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

### 1.1.1、ncos config 配置中心

提供用于存储配置和其他元数据的 key/value 存储

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

## 1.2、Ribbon

- **集中式负载均衡**，**在消费者和服务端中间使用独立的代理方式进行负载**，**硬件（F5），软件（nginx）**

- **客户端负载均衡**，客户端根据自己的请求情况做负载均衡，**ribbon属于客户端自己做的负载均衡**

ribbon通过load balance 拉取服务列表，基于某种规则调用这些实例

常见负载均衡算法：

- 轮询
- 随机
- 加权，服务提供者权重越高分配给它的流量越多
- 地址哈希，根据服务消费者ip进行hash取模进行服务器调度
- 最小连接数

Ribbon负载均衡策略：



## 1.3、spring cloud loadBalancer





## 1.4、feign

优化服务间的调用

neteflix开发的声明式、模板化的http客户端，声明在客户端，整合了负载均衡器

spring cloud openfeign对feign进行了增强，使其支持spring MVC，整合了ribbon和nacos。



@FenginClient 动态代理



feign自定义拦截器





## 1.5、nacos config 配置中心

https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config

维护性、实效性、安全性



## 1.6、sentinel 服务熔断

解决微服务的可用性问题，实现高可用

解决服务雪崩问题，服务雪崩效应：因服务提供者不可用导致服务调用者不可用，并将不可用过程逐渐扩大的过程，叫做服务雪崩效应。



常见容错机制：

- 超时机制，一旦超时，释放资源
- 限流机制，访问量超过限制，进行限流 ==服务提供端==
- 隔离，根据线程隔离、根据信号隔离（为服务设置并发限制）
- 服务熔断，==服务消费端==设置熔断机制，调用某个服务持续得不到响应，对该服务停止访问
- 服务降级，==服务消费端==服务不可用时，区分强依赖、弱依赖（如秒杀中的积分服务等），对弱依赖服务进行降级处理（后续在进行积分处理等）



sentinel 是阿里巴巴开源的，面向分布式服务架构的高可用防护组件。

https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D

具备多维度的流控降级能力，



整合openfeign进行服务降级



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