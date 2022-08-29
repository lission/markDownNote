[TOC]

# spring cloud alibaba

spring cloud alibaba 是对 spring cloud 的标准实现，**以微服务为核心的整体解决方案**。
主流框架，[spring cloud alibaba版本说明](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)

当前项目使用版本：2021.0.1.0

| Spring Cloud Alibaba Version | Spring Cloud Version  | Spring Boot Version |
| ---------------------------- | --------------------- | ------------------- |
| 2021.0.1.0*                  | Spring Cloud 2021.0.1 | 2.6.3               |



**spring cloud 各套实现对比**：

![img](https://github.com/lission/markdownPics/blob/main/spring/springcloud%E7%BB%84%E4%BB%B6.jpeg?raw=true)

![img](https://github.com/lission/markdownPics/blob/main/spring/spring%20cloud%20Alibaba%E4%BD%93%E7%B3%BB.jpg?raw=true)

## 1.1、nacos

是一个动态**服务发现(Nacos Discovery)**、**服务配置(Nacos Config)**和服务管理平台，支持几乎所有主流类型的**“服务”的发现、配置和管理：**

[Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)

[gRPC](https://grpc.io/docs/guides/concepts.html#service-definition) & [Dubbo RPC Service](https://dubbo.incubator.apache.org/)

[Spring Cloud RESTful Service](https://spring.io/projects/spring-restdocs)

集注册中心+配置中心+服务管理的平台



### 1.1.1、ncos discovery 注册中心

注册中心**管理所有微服务**，**解决服务之间调用关系复杂，难以维护**的问题。

**核心功能**：

- **服务注册**，nacos client通过**发送rest请求方式向nacos server注册自己的服务**，提供自身的元数据，比如ip、端口等，nacos server接收到注册请求后，把元数据信息存储在**双层内存map**中
- **服务心跳**，服务注册后，**nacos client维护一个定时心跳持续通知nacos server**，**默认5s发生一次心跳**
- **服务同步**，集群之间服务实例同步
- **服务发现**，服务消费者(nacos client)在**调用服务提供者的服务时**，发送一个**rest请求给nacos server**，获取**注册的服务清单**，**缓存在nacos client本地**。同时在**nacos client本地开启一个定时任务定时拉取服务端最新的注册表信息**更新到本地缓存
- **服务健康检查**，nacos server开启一个**定时任务检查注册服务实例健康状况**，对于超过**15s**没有收到客户端心跳的实例将它的**healthy属性置为false**，如果某个实例**超过30s没有收到心跳，直接剔除该实例**(被剔除的实例如果恢复发送心跳会重新注册)

当前项目版本：2021.0.1.0

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

#### CAP理论

CAP,C一致性、A可用性、P分区容错性

[CAP理论](https://zhuanlan.zhihu.com/p/50990721?utm_medium=social&utm_oi=990587482462937088)，分布式系统最大难点，**就是各个节点状态如何保持数据一致**。CAP理论是在设计分布式系统过程中，处理**数据一致性问题**时必须要考虑的理论。

- Consistency（一致性）

  > 一致性，站在分布式系统角度，对访问本系统的客户端的一种承诺：要么返回一个错误，要么返回**绝对一致的数据**。其强调**数据正确**。

- Availability（可用性）

  > 可用性，站在分布式系统角度，对访问本系统的客户端的一种承诺：一定会返回数据，不会返回错误，但不保证数据最新。其强调**不出错**。

- Partition tolerance（分区容错性）

  > 分区容错性，站在分布式系统角度，对访问本系统的客户端的一种承诺：我会一直运行，不管内部出现何种数据同步问题。其强调**不挂掉**。

CAP理论是：**一个分布式系统，不可能同时做到这三点**。


![img](https://github.com/lission/markdownPics/blob/main/architecture/capTheory.jpeg?raw=true)

**权衡C、A**

对于一个分布式系统而言，**P是前提，必须保证系统不挂掉**。只要有网络交互就一定会存在**延迟和数据丢失**，在C和A之间选择，要么保证数据一致性**（保证数据绝对正确）**，要么保证可用性**（保证系统不出错）**。

> **note：**其实这里有个关于CAP理论理解的误区。不要以为在所有时候都只能选择两个特性。在不存在网络失败的情况下（分布式系统正常运行时），C和A能够同时保证。只有**当网络发生分区或失败时**，才会在C和A之间做出选择。

**主流注册中心区别**

CP或AP

Nacos支持CP和AP两种一致性协议，***服务发现AP，服务配置CP***。

> AP协议：阿里自己实现的distro协议，是一种最终一致性协议。

![img](https://github.com/lission/markdownPics/blob/main/spring/springcloud%E6%B3%A8%E5%86%8C%E4%B8%AD%E5%BF%83%E5%AF%B9%E6%AF%94.jpeg?raw=true)

### 1.1.2、ncos config 配置中心

提供用于存储配置和其他元数据的 **key/value 存储**

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

```properties
config.nacos.host=127.0.0.1
config.nacos.port=8848
# 切记配置为命名空间id
config.nacos.namespace=dev-namespace-id
config.nacos.group=DEFAULT_GROUP
config.nacos.username=nacos
config.nacos.password=nacos

# nacos config
spring.cloud.nacos.config.server-addr=${config.nacos.host}:${config.nacos.port}
spring.cloud.nacos.config.namespace=${config.nacos.namespace}
spring.cloud.nacos.config.group=${config.nacos.group}
spring.cloud.nacos.config.username=${config.nacos.username}
spring.cloud.nacos.config.password=${config.nacos.password}
## nacos discovery
spring.cloud.nacos.discovery.server-addr=${config.nacos.host}:${config.nacos.port}
spring.cloud.nacos.discovery.namespace=${config.nacos.namespace}
spring.cloud.nacos.discovery.group=${config.nacos.group}
spring.cloud.nacos.discovery.username=${config.nacos.username}
spring.cloud.nacos.discovery.password=${config.nacos.password}
spring.cloud.nacos.discovery.metadata.service.source=${config.nacos.service.source:local}

```



## 1.2、Ribbon

主流负载均衡方案：

- **集中式(服务端)负载均衡**，**在消费者和服务端中间使用独立的代理方式进行负载**，**硬件（F5），软件（nginx）**

- **客户端负载均衡**，客户端**根据自己的请求情况**做负载均衡，**ribbon属于客户端自己做的负载均衡**

ribbon，**微服务负载均衡器**。通过load balance 拉取服务列表，基于某种规则调用这些实例。

常见负载均衡算法：

- 轮询
- 随机
- 加权，服务提供者权重越高分配给它的流量越多
- 地址哈希，根据服务消费者ip进行hash取模进行服务器调度
- 最小连接数

**使用ribbon在RestTemplate上添加@LoadBalanced注解**。

**可以为所有服务提供一个负载均衡策略，也可以为指定服务提供负载均衡策略**。所有的负载均衡策略均实现了**IRule接口**。

Ribbon默认负载均衡策略**ZoneAvoidanceRule**(默认规则)，复合判断server所在区域的性能和server的可用性选择服务器。**区域：云原生中地域部署。如果没有区域概念，采用轮询规则**。



## 1.3、spring cloud loadBalancer

spring cloud官方自己提供的客户端负载均衡器，用来替换ribbon。**目前只有轮询、随机的策略**

spring cloud 提供了两种负载均衡的客户端：

- **RestTemplate**，**spring提供的用于访问rest服务的客户端**，RestTemplate提供多种便捷远程访问http服务的方法，默认情况下，RestTemplate默认依赖jdk的http工具
- **WebClient**，spring webflux 5.0开始提供的一个**非阻塞的基于响应式编程的进行http请求的工具**。响应式编程基于Reactor。WebClient提供了标准http请求方式对应的get、post、put、delete等方法

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

阿里巴巴开源产品，一个易于使用的高性能微服务**分布式事务解决方案**。

https://seata.io/zh-cn/docs/overview/what-is-seata.html

seataw为用户提供了AT、TCC、SAGA、XA事务模式。阿里首推AT模式。

常见分布式事务解决方案：

- seata阿里分布式事务框架
- 消息队列
- saga
- XA

他们都有一个共同点，**“两阶段提交(2PC)”**。

**分布式事务理论**，分布式事务相关协议2PC,3PC。3PC协议实现非常难，目前市场主流采用2PC

### 1.7.1 2PC两阶段提交协议

Two-Phase Commit，(**无法百分之百保证事务成功率，只能尽量保证事务成功率**)。分为两个阶段，prepare(预处理阶段)和commit(提交/回滚阶段)

- prepare提交事务请求：询问——》执行——》响应
  1. 事务协调者向所有参与者发送事务请求，询问是否可执行事务操作，等待参与者响应
  2. 各参与者接收到询问后，执行事务操作，并将redo undo信息写入事务日志
  3. 如果参与者成功执行事务并写入redo undo信息，向事务协调者发送yes，否则返回no。参与者可能宕机不返回

- Commit 执行事务提交：commit请求——》事务提交——》反馈结果——》完成事务
  1. 事务协调者向所有参与者提交commit请求
  2. 参与者收到commit请求后，执行事务提交，提交完成后释放事务占用资源
  3. 参与者执行事务提交成功后向事务协调者发送ack响应
  4. 事务协调者接收到所有参与者ack响应后，完成事务提交

**2pc问题**：

1. 同步阻塞，参与者等待协调者指令时处于阻塞状态，无法进行其他操作
2. 单点2pc中，一切请求来自协调者，如果协调者宕机会导致事务不可用
3. 数据不一致，commit阶段，commit请求或rollback请求，如果有参与者宕机或网络问题，会导致数据不一致，只能人为干预或者补偿机制处理

### 1.7.2 分布式事务实现的4种模式

**AT模式**：（auto transaction），是一种无侵入式的分布式事务解决方案，seata框架实现了该模式。AT模式下，用户只需关注自己业务，用户的业务sql作为一阶段，seata框架自动生成事务的二阶段提交和回滚操作。

- 一阶段：seata拦截业务sql，首先解析sql语义，找到业务sql要更新的业务数据，在业务数据被更新前，将其保存成 before image，然后执行业务sql更新数据，在业务数据更新后，将其保存成 after image，最后生成行锁。上述操作在同一个事务中完成，保证一阶段操作的原子性
- 二阶段
  - 如果是提交的话，业务sql在一阶段已经提交至数据库，seata框架只需将一阶段保存的快照和行锁删除掉，完成数据清理即可
  - 如果是回滚的话，用before image还原业务数据，在还原前首先要校验脏写，对比数据库当前数据和after image，如果两份数据完全一致说明没有脏写，可以还原业务数据；如果不一致说明发生脏写，需要转人工处理

**TCC模式**：需要用户根据自己业务场景实现try、confirm、cancel三个操作；事务发起方在一阶段执行try方式，在二阶段提交执行confirm方法，二阶段回滚执行cancel方法

> 缺点：侵入性比较强，且需要用户自己实现事务控制逻辑
>
> 优点：整个过程没有锁，性能更强
>
> 第三方TCC框架：
>
> BeyeTCC
>
> TCC-transaction
>
> himly

**SAGA模式**：

**XA模式**：



### 1.7.3 可靠消息最终一致性方案

![img](https://github.com/lission/markdownPics/blob/main/architecture/RocketMQ%E4%BA%8B%E5%8A%A1%E6%B6%88%E6%81%AF%E5%AE%9E%E7%8E%B0%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%E6%95%B0%E6%8D%AE%E6%9C%80%E7%BB%88%E4%B8%80%E8%87%B4%E6%80%A7.png?raw=true)

![img](https://github.com/lission/markdownPics/blob/main/architecture/%E5%8F%AF%E9%9D%A0%E6%B6%88%E6%81%AF%E6%9C%80%E7%BB%88%E4%B8%80%E8%87%B4%E6%80%A7%E6%96%B9%E6%A1%88.jpg?raw=true)



结合TCC理念通过消息队列MQ，实现可靠消息最终一致性方案。

预备消息 try

删除消息 cancel

确认消息 confirm

回查状态

消息投递 commit

失败重试

### 1.7.4 seata的三大角色

seata架构中三个重要角色：

- TC（Transaction coordinator），事务协调者。维护全局和分支事务状态，驱动全局事务回滚或提交
- TM（Transaction Manager），事务管理器。定义全局事务范围，开始全局事务回滚或提交
- RM（Resource Manager），资源管理器。管理分支事务处理的资源，与TC交互以注册分支事务和报告分支事务状态，并驱动分支事务回滚或提交

> TC为单独部署的server服务端，TM和RM为嵌入到应用的client客户端



### 1.7.5 seata应用

使用@GlobalTransaction开启全局事务

server端存储方式支持三种：

- file，默认，单机模式，全局事务会话信息内存中读写并持久化本地文件root.data，性能较高
- db，高可用模式，全局事务会话信息通过db共享，相应性能差些
- redis，seata-server 1.3及以上版本支持，性能较高，存在事务信息丢失风险，要提前配置适合场景的redis持久化配置

**db+nacos高可用模式**

使用nacos作为seata配置中心和注册中心，将seata注册到nacos中，可以和参与者应用进行通信。



seata pom依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```



## 1.8、 spring cloud gateway

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/

网关作为流量的入口，常用功能包括路由转发、权限校验、限流等

spring cloud gateway 是由WebFlux+Netty+Reactor实现的响应式API网关（非阻塞），旨在为微服务架构提供一种简单有效的API路由的管理方式，并基于Filter方式提供网关基本功能。如安全验证、监控、限流等。

**spring cloud gateway功能特征**：

- 动态路由，能够匹配任何请求属性
- 支持路径重写
- 集成spring cloud 服务发现功能（nacos、Eureka）
- 可集成流控降级功能（sentinel、hystrix）
- 可以对路由指定易于编写的Predicate（断言）和Filter（过滤器）

**example**

```yaml
spring:
  cloud:
    gateway:
      default-filters: 
      - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Headers Access-Control-Allow-Methods Access-Control-Allow-Origin Access-Control-Expose-Headers Access-Control-Max-Age, RETAIN_FIRST
      # 路由规则
      routes:
        - id: swan-ucenter-ignore # 路由的唯一标识
          order: 0 # 
          uri: lb://swan-ucenter # 需要转发的地址  lb:使用nacos中的本地负载均衡策略loadBalance
          #断言规则，用于路由规则的匹配
          predicates:
          - Path=/swan-ucenter/sso/login,/swan-ucenter/sso/checkToken,/swan-ucenter/sso/logout,/swan-ucenter/captcha,/swan-ucenter/admin/login,/swan-ucenter/auth/**
          #过滤器
          filters:
          - IgnoreTokenGlobalFilter
          - StripPrefix=1 # 转发之前去掉第一层的路径
```

### 1.8.1、spring cloud gateway核心概念

- route，路由，网关中最基础的一部分，路由信息包括一个id、一个目的地uri、一组断言工厂、一组filter。如果断言为真，说明请求的url和配置的路由匹配
- predicate，断言，spring cloud gateway中的**断言函数类型是spring 5.0框架中的ServerWebExchange**。断言函数允许开发者定义匹配http request中的任何信息，比如请求头和参数
- filter，过滤器，spring cloud gateway中的filter分为gateway filter和gloable filter。filter可以对请求和响应进行处理

gateway依赖

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```



### 1.8.2、spring cloud gateway内置断言工厂（Route Predicates Factories）配置

**作用**：当我们请求gateway服务时，网关使用断言对请求进行匹配，如果成功则路由转发，否则返回404

**类型**：内置、自定义

- 基于DateTime类型的断言工厂，根据时间判断，主要有3个：

  - AfterRoutePredicateFactory，接收一个日期参数，判断请求日期是否晚于指定日期
  - BeforeRoutePredicateFactory，接收一个日期参数，判断请求日期是否早于指定日期
  - BetweenRoutePredicateFactory，接收一个日期参数，判断请求日期是否在指定时间段内

  ```yaml
  # 示例
  -After=2022-08-16T13:43:07.040
  ```

- 基于远程地址的断言工厂，RemoteAddrRoutePredicateFactory，接收一个ip地址段，判断请求主机地址是否在地址段中

  ```yaml
  - RemoteAddr=192.168.1.1/24
  ```

- 基于cookie的断言工厂，CookieRoutePredicateFactory，接收两个参数，cookie和一个正则表达式，判断请求cookie是否具有给定名称且值与正则表达式匹配

  ```yaml
  - Cookie=chocolate,ch.
  ```

- 基于Header的断言工厂，HeaderRoutePredicateFactory，接收两个参数，标题名称和正则表达式，判断请求Header是否具有给定名称且值与正则表达式匹配

  ```yaml
  - Header=-X-Request-ID,\d+ #只允许为数字
  ```

- 基于Host的断言工厂，HostRoutePredicateFactory，接收一个参数，主机名名称，判断请求host是否满足匹配规则

  ```yaml
  - Host=**.test.com
  ```

- 基于Method请求方法的断言工厂，MethodRoutePredicateFactory，接收一个参数，判断请求类型是否跟指定类型匹配

  ```yaml
  - Method=GET
  ```

- 基于Path请求路径的断言工厂，PathRoutePredicateFactory，接收一个参数，判断请求的URL部分是否满足路径规则

  ```yaml
  - Path=/foo/{segment}
  ```

- 基于Query请求参数的断言工厂，QueryRoutePredicateFactory，接收两个参数，请求param和正则表达式，判断请求参数是否具有给定名称且值与正则表达式匹配

  ```yaml
  - Query=baz,baz.
  ```

- 基于路由权重的断言工厂，WeightRoutePredicateFactory，接收一个[组名，权重]，然后对于同一个组内的路由按照权重转发

  ```yaml
  
  ```

### 1.8.3、自定义断言工厂

自定义路由断言工厂需要**继承AbstractRoutePredicateFactory**，重写apply方法逻辑，在apply方法中可以通过exchange.getRequest()拿到ServerHttpRequest对象，从而可以获取到请求的参数、请求方式、请求头等信息

> 1、命名必须以RoutePredicateFactory结尾
>
> 2、必须继承AbstractRoutePredicateFactory
>
> 3、必须在内部创建一个Config静态内部类，用于接收配置中的断言配置
>
> 4、需要结合shortcutFiledOrder进行绑定
>
> 5、通过apply进行逻辑判断，true就是匹配成功，否则失败



### 1.8.4、spring cloud gateway 过滤器工厂（GatewayFilter Factories）配置

gateway内置了很多过滤器工厂，通过过滤器工厂可以进行一些业务逻辑处理，比如添加\剔除响应头，添加\去除参数等。

- 添加请求头

  ```yaml
  - AddRequestHeader=X-Request-color,red #添加请求头
  ```

- 添加请求参数

  ```yaml
  - AddRequestParamer=X-Request-color,red #添加请求头
  ```

- 为请求路径添加前缀

  ```yaml
  - PrefixPath=/mall-order #添加前缀
  ```

- 重定向(RedirectTo) 等……

**自定义过滤器工厂**

继承AbstractNameValueGatewayFilterFactory，定义的名称必须以GatewayFilterFactory结尾。

### 1.8.5、全局过滤器（Global Filters）配置

局部过滤器：针对某个路由，需要在路由中进行配置。

全局过滤器：针对所有路由请求进行过滤，无需显示配置。



自定义全局过滤器，继承GlobalFilter，重写filter方法。

 

### 1.8.6、gateway整合sentinel流控降级

网关层的限流可以简单地**针对不同路由进行限流**，也可以**针对业务接口进行限流**，或者**根据接口特征分组限流**

**1、添加依赖**

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

**2、添加配置**

```yaml
sentinel:
	transport:
		#添加sentinel的控制台地址
		dashboard:127.0.0.1:8080
```

**3、sentinel配置方式**

- 控制台配置方式
- 代码实现配置方式



网关高可用，可以多部署几台网关服务，通过nginx或f5进行负载均衡



## 1.9、 skywalking

链路追踪

官网网址：https://skywalking.apache.org/

中文文档：https://skyapm.github.io/document-cn-translation-of-skywalking/

