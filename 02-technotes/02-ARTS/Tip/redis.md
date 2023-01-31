[TOC]

[参考](https://juejin.im/post/5b7d226a6fb9a01a1e01ff64)

[完善](https://gitee.com/pingWurth/study-notes/blob/master/redis/Redis%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0.md#lists)

# 一、Redis基础

## 1.1、Redis是什么

==面试官：你先来说下redis是什么吧==

redis是C语言开发的**一个开源的高性能key-value键值对内存数据库**。可以用作数据库、缓存、消息中间件等。是**一种非关系型数据库**。

redis内存数据库的特点：

- 性能优秀，**数据在内存中，读写非常快，支持10WQPS**
- 单进程单线程，线程安全，采用**IO多路复用机制**
- 丰富的数据类型，支持**字符串(string)、散列(hash)、列表(list)、集合(set)、有序集合(zSet)、stream**
- **支持数据持久化**，可以将内存中数据存储在磁盘，重启时加载
- **所有操作都是原子性，还支持对几个操作合并后的原子性**
- 主从复制，哨兵机制，高可用
- 可以用作**分布式锁**
- 可以用作消息中间件，支持发布订阅



## 1.2、Redis各版本特性解读

### 1.2.1、Redis 2.6.0

Redis2.6 在 2012 年正是发布，经历了 17 个版本，到 2.6.17 版本，相对于 Redis2.4，主要特性如下：

1）服务端支持 Lua 脚本。

2）去掉虚拟内存相关功能。

3）放开对客户端连接数的硬编码限制。

4）键的过期时间支持毫秒。

5）从节点支持只读功能。

6）两个新的位图命令：bitcount 和 bitop。

7）增强了 redis-benchmark 的功能：支持定制化的压测，CSV 输出等功能。

8）基于浮点数自增命令：incrbyfloat 和 hincrbyfloat。

9）redis-cli 可以使用 –eval 参数实现 Lua 脚本执行。

10）shutdown 命令增强。

11）重构了大量的核心代码，所有集群相关的代码都去掉了，cluster 功能将会是3.0版本最大的亮点。

12）info 可以按照 section 输出，并且添加了一些统计项

13）sort 命令优化

### 1.2.2、Redis 2.8

Redis2.8 在 2013 年 11 月 22 日正式发布，经历了 24 个版本，到 2.8.24 版本，相比于 Redis2.6，主要特性如下：

1）添加部分主从复制的功能，在一定程度上降低了由于网络问题，造成频繁全量复制生成 RDB 对系统造成的压力。

2）尝试性的支持 IPv6.

3）可以通过 config set 命令设置 maxclients。

4）可以用 bind 命令绑定多个 IP 地址。

5）Redis 设置了明显的进程名，方便使用 ps 命令查看系统进程。

6）config rewrite 命令可以将 config set 持久化到 Redis 配置文件中。

7）发布订阅添加了 pubsub。

8）Redis Sentinel 第二版，相比于 Redis2.6 的 Redis Sentinel，此版本已经变成生产可用。

### 1.2.3、Redis 3.0 (里程碑)

Redis3.0 在 2015 年 4 月 1 日正式发布，相比于 Redis2.8 主要特性如下：

Redis 最大的改动就是添加 Redis 的分布式实现 Redis Cluster。

1）Redis Cluster：Redis 的官方分布式实现。

2）全新的 embedded string 对象编码结果，优化小对象内存访问，在特定的工作负载下载速度大幅提升。

3）Iru 算法大幅提升。

4）migrate 连接缓存，大幅提升键迁移的速度。

5）migrate 命令两个新的参数 copy 和 replace。

6）新的 client pause 命令，在指定时间内停止处理客户端请求。

7）bitcount 命令性能提升。

8）cinfig set 设置 maxmemory 时候可以设置不同的单位（之前只能是字节）。

9）Redis 日志小做调整：日志中会反应当前实例的角色（master 或者 slave）。

10）incr 命令性能提升。

### 1.2.4、Redis 3.2

Redis3.2 在 2016 年 5 月 6 日正式发布，相比于 Redis3.0 主要特征如下：

1）添加 GEO 相关功能。

2）SDS 在速度和节省空间上都做了优化。

3）支持用 upstart 或者 systemd 管理 Redis 进程。

4）新的 List 编码类型：quicklist。

5）从节点读取过期数据保证一致性。

6）添加了 hstrlen 命令。

7）增强了 debug 命令，支持了更多的参数。

8）Lua 脚本功能增强。

9）添加了 Lua Debugger。

10）config set 支持更多的配置参数。

11）优化了 Redis 崩溃后的相关报告。

12）新的 RDB 格式，但是仍然兼容旧的 RDB。

13）加速 RDB 的加载速度。

14）spop 命令支持个数参数。

15）cluster nodes 命令得到加速。

16）Jemalloc 更新到 4.0.3 版本。

### 1.2.5、Redis 4.0

Redis4.0 的新特性：

1）提供了模块系统，方便第三方开发者拓展 Redis 的功能。

2）PSYNC2.0：优化了之前版本中，主从节点切换必然引起全量复制的问题。

3）提供了新的缓存剔除算法：LFU（Last Frequently Used），并对已有算法进行了优化。

4）提供了非阻塞 del 和 flushall/flushdb 功能，有效解决删除了 bigkey 可能造成的 Redis 阻塞。

5）提供了 memory 命令，实现对内存更为全面的监控统计。

6）提供了交互数据库功能，实现 Redis 内部数据库的数据置换。

7）提供了 RDB-AOF 混合持久化格式，充分利用了 AOF 和 RDB 各自优势。

8）Redis Cluster **兼容 NAT 和 Docker**。

### 1.2.6、Redis 5.0

1）新的 Stream 数据类型。

2）新的 Redis 模块API：Timers and Cluster API。

3）RDB 现在存储 LFU 和 LRU 信息。

4）集群管理器从 Ruby（redis-trib.rb）移植到 C 代码。可以在 redis-cli 中。查看`redis-cli —cluster help`了解更多信息。

5）新 sorted set 命令：ZPOPMIN / MAX 和阻塞变量。

6）主动碎片整理 V2。

7）增强 HyperLogLog 实现。

8）更好的内存统计报告。

9）许多带有子命令的命令现在都有一个 HELP 子命令。

10）客户经常连接和断开连接时性能更好。

11）错误修复和改进。

12）Jemalloc 升级到 5.1 版

### 1.2.7、Redis 6.0

1）**多线程IO**。Redis 6 引入多线程 IO。但**多线程部分只是用来处理网络数据的读写和协议解析**，**执行命令仍然是单线程**。之所以这么设计是不想因为多线程而变得复杂，需要去控制 key、lua、事务，LPUSH/LPOP 等等的并发问题。

2）重新设计了客户端缓存功能。实现了 Client-side-caching（客户端缓存）功能。放弃了caching slot，而只使用 key names。

*Redis server-assisted client side caching*

3）RESP3 协议。RESP（Redis Serialization Protocol）是 Redis 服务端与客户端之间通信的协议。Redis 5 使用的是 RESP2，而 Redis 6 开始在兼容 RESP2 的基础上，开始支持 RESP3。

推出 RESP3 的目的：一是因为希望能为客户端提供更多的语义化响应，以开发使用旧协议难以实现的功能；另一个原因是实现 Client-side-caching（客户端缓存）功能。

4）支持 SSL。连接支持 SSL，更加安全。

5）ACL 权限控制

- 支持对客户端的权限控制，实现对不同的 key 授予不同的操作权限。

- 有一个新的 ACL 日志命令，允许查看所有违反 ACL 的客户机、访问不应该访问的命令、访问不应该访问的密钥，或者验证尝试失败。这对于调试 ACL 问题非常有用。

  6）提升了 RDB 日志加载速度。根据文件的实际组成（较大或较小的值），可以预期 20/30% 的改进。当有很多客户机连接时，信息也更快了，这是一个老问题，现在终于解决了。

  7）发布官方的 Redis 集群代理模块 Redis Cluster proxy。在 Redis 集群中，客户端会非常分散，现在为此引入了一个集群代理，可以为客户端抽象 Redis 群集，使其像正在与单个实例进行对话一样。同时在简单且客户端仅使用简单命令和功能时执行多路复用。

  8）提供了众多的新模块（modules）API

### 1.2.8、Redis 7.0

…… https://raw.githubusercontent.com/redis/redis/7.0/00-RELEASENOTES



## 1.3、Redis 客户端使用(java)

- Jedis 在实现上是直接连接的 redis server，如果在多线程环境下是非线程安全的，这个时候只有使用连接池，为每个 Jedis 实例增加物理连接。
- Lettuce 的连接是基于Netty的，连接实例（StatefulRedisConnection）可以在多个线程间并发访问，因为 StatefulRedisConnection 是线程安全的，所以一个连接实例（StatefulRedisConnection）就可以满足多线程环境下的并发访问，当然这个也是可伸缩的设计，一个连接实例不够的情况也可以按需增加连接实例。
- 在 SpringBoot Data Redis 1.X 之前默认使用的是 Jedis，但目前最新版的修改成了 Lettuce。
- 之前公司使用 Jedis 居多，Lettuce 近两年在逐步上升，总的来讲 Jedis 的性能会优于 Lettuce（因为它是直接操作 Redis）。

### 1.3.1、Redis 常用客户端—Jedis

- **第1步：导入 Jedis 依赖**

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.3.0</version>
</dependency>
```

- **第2步：连接测试**

```java
public class ConnectionToRedis {
    /**
     * 连接Redis
     */
    public static void main() {
        // 创建jedis对象，连接redis服务，其中Jedis(host, port)
        Jedis jedis = new Jedis("127.0.0.1", 6379);

        // 设置认证密码，如果没有可以设置为空
        jedis.auth("jasdklfj%*@#&$EHkfjsdha");

        // 指定数据库 默认是0
        jedis.select(1);

        // 使用 ping 命令，测试连接是否成功
        String result = jedis.ping();
        System.out.println(result);// 返回PONG

        // 添加一条数据
        jedis.set("username", "zhangsan");

        // 获取一条数据
        String username = jedis.get("username");
        System.out.println(username);

        // 释放资源
        if (jedis != null)
            jedis.close();
    }
}
```

- **第3步：使用连接池**

```java
public class JedisPoolConnectRedis {

    private static JedisPool jedisPool;

    static {
        // 创建连接池配置对象
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        // 设置最大连接数
        jedisPoolConfig.setMaxTotal(5);
        // 设置等待时间ms(当资源池连接用尽后，调用者最大等待时间)
        jedisPoolConfig.setMaxWaitMillis(100);
        // 其中JedisPool(jedisPoolConfig, host, port, connectionTimeout, password, db)
        jedisPool = new JedisPool(jedisPoolConfig, "127.0.0.1", 6379, 100, "root", 0);
    }

    /*获取jedis*/
    public static Jedis getJedis() {
        return jedisPool.getResource();
    }
    /**
     * 连接Redis
     */
    public static void main() {
        Jedis jedis = getJedis();

        // 使用ping命令，测试连接是否成功
        String result = jedis.ping();
        System.out.println(result);// 返回PONG

        // 添加一条数据
        jedis.set("username", "zhangsan");

        // 获取一条数据
        String username = jedis.get("username");
        System.out.println(username);

        // 释放资源
        if (jedis != null)
            jedis.close();
    }
}
```

### 1.3.2、Redis 常用客户端—RedisTemplate

Spring 封装了 RedisTemplate 来操作 Redis，它支持所有的 Redis 原生的 API。在RedisTemplate 中定义了对 5 种数据结构的操作方法。

- opsForValue()
- opsForHash()
- opsForList()
- opsForSet()
- opsForZSet()

**还可以使用 `execute` 方法，`opsForXXX` 底层就是通过调用 execute 来实现的。**

- **第一步：Spring Boot 配置 Redis**

**添加 Spring Data Redis 依赖**

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

- **第二步：添加配置**

```yaml
spring:
  redis:
    host: 127.0.0.1     # redis 服务器地址
    port: 6379          # 端口
    timeout: 6000       # 连接超时时间（毫秒）
    pool:
      max-active: 8     # 连接池的配置，最大连接激活数
      max-idle: 8       # 连接池配置，最大空闲数
      max-wait: -1      # 连接池配置，最大等待时间
      min-idle: 0       # 连接池配置，最小空闲活动连接数
```

- **第三步：注入使用的方式**

```java
@Resource(name = "redisTemplate")
private RedisTemplate<String, String> template;

@Resource(name = "redisTemplate")
private ValueOperations<String, Object> vOps;

@Resource(name = "redisTemplate")
private HashOperations<String, String, Object> hashOps;

@Resource(name = "redisTemplate")
private RedisTemplate<String, String> template;
```



### 1.3.3、Redis 常用客户端—Redission

wiki 文档：[https://github.com/redisson/redisson/wiki](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Fredisson%2Fredisson%2Fwiki)

 springboot starter https://github.com/redisson/redisson/tree/master/redisson-spring-boot-starter#spring-boot-starter

- **第一步：添加依赖**

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-data-21</artifactId>
    <version>3.13.6</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.13.6</version>
    <exclusions>
        <exclusion>
            <groupId>org.redisson</groupId>
            <artifactId>redisson-spring-data-23</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

- **第二步：添加配置 application.yml**

```yaml
spring: 
  redis:
    database: 0
    host: localhost
    port: 6379
    password: 123456
    redisson:
      file: classpath:redisson.yml
    jedis:
      max-active: 20     # 连接池最大连接数（使用负值表示没有限制）
      max-wait: 20000ms  # 连接池最大阻塞等待时间（使用负值表示没有限制）
      max-idle: 20       # 连接池中的最大空闲连接
      min-idle: 20       # 连接池中的最小空闲连接
```

- **第三步：添加配置 redisson.yml**

```yaml
# 单节点配置
singleServerConfig:
  # 连接空闲超时，单位：毫秒
  idleConnectionTimeout: 10000
  # 连接超时，单位：毫秒
  connectTimeout: 10000
  # 命令等待超时，单位：毫秒
  timeout: 3000
  # 命令失败重试次数,如果尝试达到 retryAttempts（命令失败重试次数） 仍然不能将命令发送至某个指定的节点时，将抛出错误。
  # 如果尝试在此限制之内发送成功，则开始启用 timeout（命令等待超时） 计时。
  retryAttempts: 3
  # 命令重试发送时间间隔，单位：毫秒
  retryInterval: 1500
  # 密码
  password: 123456
  # 单个连接最大订阅数量
  subscriptionsPerConnection: 5
  # 客户端名称
  clientName: axin
  # 节点地址
  address: redis://localhost:6379
  # 发布和订阅连接的最小空闲连接数
  subscriptionConnectionMinimumIdleSize: 1
  # 发布和订阅连接池大小
  subscriptionConnectionPoolSize: 50
  # 最小空闲连接数
  connectionMinimumIdleSize: 32
  # 连接池大小
  connectionPoolSize: 64
  # 数据库编号
  database: 0
  # DNS监测时间间隔，单位：毫秒
  dnsMonitoringInterval: 5000


# 线程池数量,默认值: 当前处理核数量 * 2
#threads: 0

# Netty线程池数量,默认值: 当前处理核数量 * 2
#nettyThreads: 0

# 编码
codec: !<org.redisson.codec.JsonJacksonCodec> {}
# 传输模式
transportMode: "NIO"
```

- **第四步：根据 profile 加载 `redisson-${profile}.yaml`**

```java
/**
 * Redisson 配置.
 * <p>
 *
 * @author Ping Wurth
 * @date 2021/6/6 22:48
 */
@Component
public class RedissonConfig {
    @Autowired
    private Environment env;

    /**
     * 加载 redisson.yaml 配置，创建 RedissonClient 实例
     *
     * @return
     * @throws IOException
     */
    @ConditionalOnMissingBean(RedissonClient.class)
    @Bean(destroyMethod = "shutdown")
    public RedissonClient redissonClient() throws IOException {
        String[] profiles = env.getActiveProfiles();
        ClassPathResource yamlConfigFile;

        // 设置了 profile，加载 redisson-${profile}.yml
        if (profiles.length > 0) {
            yamlConfigFile = new ClassPathResource("redisson-" + profiles[0] + ".yaml");
            if (yamlConfigFile.exists()) {
                return Redisson.create(Config.fromYAML(yamlConfigFile.getInputStream()));
            }
        }

        // 加载 redisson.yaml 创建 Redisson 实例
        yamlConfigFile = new ClassPathResource("redisson.yaml");
        if (yamlConfigFile.exists()) {
            return Redisson.create(Config.fromYAML(yamlConfigFile.getInputStream()));
        }

        // 没有 redisson.yaml 配置文件
        return null;
    }
}
```

- **第五步：启动类添加注解 `@EnableCaching`**



### 1.3.4、SpringBoot集成Redis

- **第1步：引入依赖**

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
```

- **第2步：配置Redis**

  在 **application.properties** 中添加 Redis 相关配置

  - **单实例 Redis**
  
    ```properties
    # 主机
    spring.redis.host=127.0.0.1
    # 端口
    spring.redis.port=6379
    # 密码
    spring.redis.password=jsklajfoI^&(%Y#hsakfakfyq)
    # 数据库，默认第 0 个
    spring.redis.database=0
    
    # 最大连接数量 = maxTotal
    spring.redis.jedis.pool.max-active=8
    # 资源池允许最大空闲数
    spring.redis.jedis.pool.max-idle=8
    # 资源池确保最少空闲连接数
    spring.redis.jedis.pool.min-idle=2
    # 连接超时时间
    spring.redis.jedis.pool.max-wait=1000
    ```
  
  - **哨兵模式（一主二从）**
  
    ```yaml
    spring:
      redis:
        database: 1
        password: pingwurth
        sentinel:
          master: pingwurth-master
          nodes: 192.168.1.191:26379,192.168.1.192:26379,192.168.1.193:26379
    ```
  
  - **集群模式（三主三从）**
  
    ```yaml
    spring:
      password: pingwurth
      cluster:
        nodes: 192.168.1.201:6379,192.168.1.202:6379,192.168.1.203:6379,192.168.1.204:6379,192.168.1.205:6379,192.168.1.206:6379
    ```
  
- ** 第3步：添加 Redis 序列化方法**

```java
/**
     * redisTemplate 序列化使用的jdkSerializeable, 存储二进制字节码, 所以自定义序列化类
     * @param redisConnectionFactory
     * @return
     */
    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);

        // 使用Jackson2JsonRedisSerialize 替换默认序列化
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        // 设置key和value的序列化规则
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);

        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);

        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
```

- ** 第4步：测试 Redis**

```java
@SpringBootTest
public class ConnectionRedisTest {

    @Resource
    private RedisTemplate redisTemplate;

    @Test
    void testConnection() {
        String result = redisTemplate.getConnectionFactory().getConnection().ping();
        // 验证
        assertEquals("PONG", result);
    }

    @Test
    void testOptString() {
        // 写入数据
        redisTemplate.opsForValue().set("username", "Hello Redis!");
        // 读取数据
        String result = (String)redisTemplate.opsForValue().get("username");
        // 验证
        assertEquals("Hello Redis!", result);
    }
}
```




# 2、五种数据类型+redis 5.0新增的stream类型

==面试官：总结的不错，看来是早有准备啊。刚来听你提到redis支持五种数据类型，那你能简单说下这五种数据类型吗？==

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redisObject.png)

redis内部使用redisObject来表示所有的key和value，redisObject主要信息如上，**type表示value对象具体是何种数据类型**，encoding是不同数据类型在redis内部的存储方式。比如：type=string表示value存储的是一个普通字符串，那么encoding可以是raw或者int。

简单说一下5种数据类型：

- **string是redis基本数据类型**，一个key对应一个value，**==value不仅可以是字符串也以是数字==**。string类型是二进制全的，==redis的string可以包含任何数据，比如jpg图片或者序列化的对象==。string类型的值最大能存储512M。

- **hash是一个键值（key-value）的集合**。redis的hash是一个string的key和value的映射表，hash特别适合存储对象。

  - 常用命令hget,hset,hgetall等

- **list列表是简单的字符串列表，按照插入顺序排序**，可以添加一个元素到列表的头部(左边)或尾部(右边)，常用命令：lpush、rpush、lpop、rpop、lrange(获取列表片段等)
  - 应用场景：list应用场景非常多，比如twitter的关注列表，粉丝列表等都可以用list结构实现
  - 数据结构：list就是链表，redis提供了List的push和pop操作
  - 实现方式：redis list的实现是一个**双向链表**，支持反向查找和遍历，方便操作，但是带来了额外内存开销

- **set是string类型的无序集合**，集合通过hashtable实现，set中元素无序，且没有重复的

  - 常用命令：sdd、spop、smembers、sunion等
  - 应用场景：redis set对外提供功能与list一样是一个列表，特殊在于set自动去重，set提供判断某个成员是否在一个set集合中

- **zset和set一样是string类型元素的集合，不允许重复元素**。

  - 常用命令：zadd，zrange、zrem、zcard
  - 使用场景：sorted set可以通过用户额外提供一个优先级(score)的参数来为成员排序，且插入是有序的。和set相比，sorted set**关联了一个double类型权重的参数score**，使集合中的元素能够按照score进行有序排序，redis是通过分数为集合中的成员进行从小到大的排序
  - 实现方式：sorted set内部使用hashMap和跳跃表skipList保证数据存储和有序，HashMap里存放的是成员到score的映射，而跳跃表存放的是所有成员，排序依据HashMap里存的score，跳跃表的结构可以或得较高的查找效率

  数据类型应用场景总结

  | 类型       | 简介                                                    | 特性                                                         | 场景                     |
  | ---------- | ------------------------------------------------------- | ------------------------------------------------------------ | ------------------------ |
  | string     | 为二进制安全，最大能存储512M。支持自增自减操作          | 可以包含任何数据，比如jpg图片或序列化对象                    | ……                       |
  | Hash       | 键值对集合，编程语言中的map类型                         | 适合存储对象，可以像数据库中的update一样，只修改某一项属性值 | 存储、读取修改用户属性   |
  | List       | 链表，双向链表                                          | 增删快，提供了操作某一元素的api                              | 最新消息排行，消息队列   |
  | set        | hash表实现，元素不重复                                  | 添加、删除、查找的复杂度都是O(1)，提供求交际、冰机】差集操作 | 共同好友；利用唯一性     |
  | sorted set | 将set中的元素增加一个权重参数score，元素按score有序排列 | 数据插入集合时，进行了天然排序                               | 排行榜，带权重的消息队列 |

- stream，redis 5.0添加的一种新的数据类型，它是一个新的强大的支持多播的**可持久化的消息队列**。[参考](https://www.zhihu.com/question/279540635/answer/409746087)，stream借鉴了kafka设计。它有一个**消息链表，将所有加入的消息都串起来，每个消息都有一个唯一的ID和对应的内容**。消息是持久化的，Redis重启后，内容还在

# 3、redis缓存

==面试官：说一下redis缓存怎么用的==

结合springboot使用的，一般两种方式：

- 直接通过RedisTemplate使用
- 使用spring cache集成Redis

# 4、缓存问题

==面试官：实际项目中使用缓存有遇到什么问题或会遇到什么问题吗？==

**缓存和数据库数据一致性**问题，分布式环境下非常容易出现缓存和数据数据一致性问题，如果项目对缓存要求强一致性，那就不要使用缓存。只能采取合适的策略降低缓存和数据库间数据不一致的概率，而无法保证两者间的强一致性。合适的策略包括**缓存更新策略**，更新数据库后及时更新缓存、**缓存失败时增加重试机制**

==面试官：redis雪崩了解吗==

目前电商首页和热点数据都会去做缓存，一般缓存都是定时任务去刷新，或者查不到之后去更新缓存。**如果Redis缓存同一时间大面积失效，大数量级的请求直接到数据库，导致数据库服务崩溃，这就是缓存雪崩**。

==面试官：如何处理缓存雪崩==

- **在批量往redis存数据的时候，把每个key的失效时间加个随机值**，保证数据不会再同一时间大面积失效。

- 如果redis是集群部署，将热点数据均匀分布在不同的redis库中
- **定时更新热点数据，刷新缓存，避免自动失效**
- **服务限流、接口限流**，如果服务和接口都有限流机制，就算缓存全部失效了，但是请求的总量是有限制的，可以在承受范围之类，这样短时间内系统响应慢点，但不至于挂掉，影响整个系统

==面试官：了解缓存穿透和击穿吗，他们和缓存雪崩的区别是什么？==

- 缓存穿透：指缓存和数据库中都没有的数据，用户不断发起请求，导致数据库压力很大，严重时会击垮数据库
- 缓存击穿：与缓存雪崩相似，缓存雪崩是因为大面积缓存失效，打崩了DB，**缓存击穿是指一个key非常热点，不停的扛着大量请求，大并发集中对一个点进行访问，在这个key失效的瞬间，持续的大并发直接落到了数据库上**，就在这个key的点上击穿了缓存

==面试官：如何解决缓存穿透和击穿呢==

缓存穿透：

- 在接口层增加校验，比如用户鉴权，参数校验，不合法参数校验直接return。
- 或者使用**布隆过滤器**，利用高效的数据结构和算法快速判断这个key是否存在库中，不存在直接return。

缓存击穿，可以**对热点数据设置永不过期，或者加上互斥锁**

# 5、redis为什么这么快

==面试官：redis是单线程的，为什么还能这么快==

redis是**单进程单线程模型**

- redis**完全基于内存**，绝大部分请求是纯粹的内存操作
- 数据结构简单，对数据操作也简单，数据存在内存中类似于HashMap，**查找和操作的时间复杂度是O(1)**
- 采用单线程，**避免了不必要的上下文切换和竞争条件**，不存在多线程导致的**cpu切换**，无需考虑各种锁的问题，**不存在加锁释放锁操作，没有死锁问题导致的性能消耗**
- 采用**IO多路复用模型，非阻塞IO**，(**一个线程，通过记录I/O流的状态来同时管理多个I/O，可以提高服务器的吞吐能力**)[参考](https://mp.weixin.qq.com/s?__biz=MzU2MTI4MjI0MQ==&mid=2247488306&idx=1&sn=8497ae941b5a13de03e9b94451770a0d&chksm=fc7a7e9ccb0df78a265c3fddc417bba21a76da913e45d1173a6a670156e868f96022935f3d9c&scene=21#wechat_redirect)

# 6、redis和Memcached区别

==面试官：为何选择redis而不用memcached==

- 存储方式上，memcached会把数据全部放在内存中，不会持久化到磁盘，断电或重启后数据小时，数据不能超过内存大小。**redis会支持数据持久化**
- 数据支持类型上：memcached对数据类型支持简单，只支持简单的key-value，**redis支持五(6)种数据类型**
- 底层模型不同，他们之间底层实现方式以及客户端之间的通信应用协议不一样，redis直接自己构建了VM机制，因为一般的系统调用系统函数，会浪费一定时间去移动和请求
- value的上限，redis可以达到1GB，memcache只有1MB

# 7、淘汰策略

==面试官：redis的淘汰策略有哪些==

redis有六**（8）**种淘汰策略

| 策略            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| volatile-lru    | 从**已设置过期时间**的KV集中优先对==最近最少使用(less recently used)==的数据淘汰 |
| volatile-ttl    | 从**已设置过期时间**的KV集中优先对==剩余时间短(time to live)==的数据淘汰 |
| volatile-random | 从**已设置过期时间**的KV集中==随机选择数据淘汰==             |
| allkeys-lru     | 从所有KV集中优先对==最近最少使用(less recently used)==的数据淘汰 |
| allkeys-random  | 从所有KV集中==随机选择数据淘汰==                             |
| noeviction      | 不淘汰策略，若超过最大内存，返回错误信息                     |

redis4.0加入了**LFU(least frequency use)淘汰策略**，包括volitile-lfu和allkeys-lfu,通过统计访问频率，**将访问频率最少**，即最不经常使用的KV淘汰

# 8、持久化

==面试官：你对redis的持久化机制了解吗，能讲一下吗==

redis为了保证效率，数据缓存在了内存中，但是会周期性的把更新的数据写入磁盘或把修改操作写入追加的记录文件，以保证数据的持久化。redis的**持久化策略**有两种：

- RDB：快照形式是直接把内存中的数据保存到一个dump文件中，定时保存，默认保存策略
- AOF：**把所有对redis的服务器进行修改的命令都存到一个文件里**，命令的集合。
- 混合持久化，RDB和AOF相结合
- redis默认是快照rdb的持久化方式

redis重启时，会优先使用AOF文件来还原数据集，因为AOF文件保存的数据集通常比RDB文件保存的数据集更完整。[redis持久化机制](https://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247489359&idx=1&sn=30367a57e57aa107511be22cf41e8c18&chksm=ebd62863dca1a175197409337f22b532177a392b4fe9f289fdf416617a462f376e44663c5290&scene=21#wechat_redirect)

==面试官：RDB是怎么工作的？==

默认redis会以快照RDB的形式将数据持久化到磁盘的一个**二进制文件dump.rdb**。

工作原理简单说一下：**当redis需要做持久化时，redis主进程会fork一个子进程，子进程将数据写到磁盘上一个临时RDB文件中。当子进程完成写临时文件后，==将原来的RDB替换掉==，好处是可以copy-on-write**。

**BGSAVE**命令，调用glib(linux系统底层api)的fork函数，启动一个子进程执行快照持久化。

**SAVE**命令，阻塞所有客户端，生成RDB文件保存所有数据当前时间点的快照。（持久化期间redis不可用）

RDB的优点是，这种文件非常适合备份，可以在最近的24小时内，每小时备份一次，并且每个月的每天备份一个RDB文件，这样即使遇到问题，也可以随时将数据集还原到不同版本。RDB非常**适合灾难恢复**

RDB工作原理:

![img](https://github.com/lission/markdownPics/blob/main/redisPic/RDB%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.png?raw=true)

**RDB的缺点**是，**恢复时会丢失部分数据**；RDB每次在fork子进程执行RDB快照数据文件生成时，如果数据文件特别大，可能会导致对客户端提供的服务暂停数毫秒，甚至数秒

==面试官：请再说一下AOF==

使用AOF做持久化，**每一个写命令都通过write函数追加到appendonly.aof中**，配置方式如下

```
appendfsync yes   
appendfsync always     #每次有数据修改发生时都会写入AOF文件。
appendfsync everysec   #每秒钟同步一次，该策略为AOF的缺省策略。
```

AOF可以做到全程持久化，只需要在配置中开启appendonly yes。这样redis每执行一个修改数据的命令，都会把它添加到AOF文件中，**当redis重启时，将会读取aof文件进行重放，恢复到redis关闭前的最后时刻。**

使用aof的优点是会让redis变得非常耐久。可以设置不同的fsync策略，**aof默认策略是每秒钟fsync一次**。

AOF日志文件以append-only模式写入，所以没有任何磁盘寻址的开销，写入性能非常高。

**==AOF重写==**

aof文件越来越大，(同一个键可能存在多个写命令，已删除或已过期的数据命令仍然保留)，**redis会对AOF文件重写，将当前数据内容保存一份到文件中，替换原来的AOF文件**。

1. fork一个子进程，称为AOF进程，**AOF进程负责将当前内存数据保存到一个新文件中**
2. AOF进程将**步骤1执行期间主进程进行的增量命令写到新文件中**，最后结束AOF进程
3. 主进程进行收尾，将**步骤2执行期间主进程执行的增量命令也写到新文件中**，替换AOF文件，AOF文件重写完成。

**缺点**：

- 对于相同的数据集来说，**aof的文件体积通常要大于rdb文件体积**。
- **根据所使用的sync策略，aof的速度可能慢于rdb**

==面试官：如何选择持久化机制==

如果非常关心数据，但仍然可以承受数分钟的数据丢失，那么可以只使用RDF持久。AOF将redis执行的每一条写命令追加到磁盘内，处理巨大的写入会降低redis性能。

数据备份和灾难恢复：定时生成rdb快照非常便于进行数据库备份，并且rdb恢复数据集的速度也要比aof恢复速度快。

**redis支持同时开启rdb和aof**，系统重启后，**redis会优先使用aof来恢复数据，这样丢失的数据最少**。

# 9、主从复制

==面试官：redis单节点存在**单点故障问题**，为解决单点问题，一般都需要对redis配置从节点，然后使用哨兵来监听主节点存活状态，如果主节点挂掉，从节点能继续提供缓存功能，你能说说redis主从复制的过程和原理吗==

**主从复制作用**

- 数据冗余，将数据热备份到从节点，即使主节点故障，从节点依然可以工作
- 读/写分离，由主节点提供写服务，从节点提供读服务，提高redis整体吞吐量
- 故障恢复，主节点故障，可以手动将从节点切换为主节点或结合哨兵模式，实现主从切换

主从复制过程：

- **从节点**执行slaveof(masterIP)(masterPort)，**保存主节点信息**
- **从节点中的定时任务发现主节点信息，建立和主节点的socket连接**
- 从节点发送ping信号，主节点返回pong门两边能互相通信
- 连接建立后，**主节点将所有数据发送给从节点(数据同步)**
- 主节点把当前的数据同步给从节点后，便完成了复制的建立过程。接下来，**主节点就会持续的把写命令发送给从节点，保证主从数据一致性**

==面试官：你能详细说一下数据同步过程吗==

![img](https://github.com/lission/markdownPics/blob/main/redisPic/Redis%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6%E7%AE%80%E6%98%93%E6%B5%81%E7%A8%8B%E5%9B%BE.png?raw=true)

redis2.8之前使用sync(runId)(offset)同步命令，reidis2.8之后使用psync(runId)(offset)命令。[redis集群和mysql主从同步](http://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247490339&idx=1&sn=355246186ce9cad69a8a4fe578141ffb&chksm=ebd6240fdca1ad19a363b3c0a917352f78ca4cdb20c79f639298ef731295bde3753fd55cf539&scene=21#wechat_redirect)

两者不同在于**sync命令仅支持全量复制过程**，**psync支持全量和部分复制**。介绍同步之前，先介绍几个概念：

- runId：每个redis节点启动都会生成唯一的uuid，每次redis重启后，runId都会发生变化
- offset：主节点和从节点都各自维护自己的**==主从复制偏移量offset==**，当主节点有写入命令时，offset=offset+命令的字节长度。从节点在收到主节点发送的命令后，会增加自己的offset，并把自己的offset发送给主节点。这样，**主节点同时保存自己的offset和从节点的offset，通过比对offset来判断主从节点数据是否一致**。
- repl_backlog_size：保存在主节点上的一个固定长度的先进先出队列，默认大小是1MB。
- 主节点发送数据给从节点过程中，主节点还会进行一些写操作，这时数据存储在**复制缓冲区**中。从节点同步主节点数据完成后，**主节点将缓冲区的数据发送给从节点，用于部分复制**。
- 主节点响应写命令时，不但会把命令发送给从节点，还会写入复制积压缓冲区，用于复制命令丢失的数据补救。

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-psync.png)

从节点发送psync(runId)(offset)命令，**主节点有三种响应**ß：

- FULLRESYNC：第一次连接，进行全量复制
- CONTINUE：进行部分复制
- ERR：不支持psync命令，进行全量复制

==面试官：能具体说一下全量复制和部分复制的过程吗==

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-allcopy.png)

全量复制过程：

- 从节点发送psync ? -1 命令（因为第一次发送，不知道主节点runid，所以为?，因为是第一次复制，所以offset=-1）。
- 从节点接收主节点信息后，保存到info中
- 主节点在发送FULLRESYNC后，**启动bgsave命令，生成RDB文件**
- 主节点发送RDB文件给从节点。到从节点加载数据完成这段期间主节点的写命令放入**复制缓冲区**。
- 从节点清理自己的数据库数据
- 从节点加载RDB文件，将数据保存到自己的数据库中
- 如果从节点开启了AOF，从节点会异步重写AOF文件

部分复制：

- **==部分复制主要是redis针对全量复制的过高开销做出的一种优化措施==**，使用psync(runId)(offset)命令实现。当从节点正在复制主节点时，如果出现网络闪断或命令丢失等异常情况，从节点会向主节点要求补发丢失的命令数据，主节点的**复制积压缓冲区**将这部分数据直接发送给从节点，保持主从节点复制的一致性。补发的这部分数据一搬远远小于全量数据。
- 主从连接中断期间主节点依然响应命令，但因复制连接中断命令无法发送给从节点，不过***主节点内的复制积压缓冲区依然可以保存最近一段时间的写命令数据***。
- 当主从连接恢复后，由于从节点之前保存了自身已复制的偏移量和主节点的运行id，因此会把它们当做psync参数发送给主节点，要求进行部分复制。
- 主节点接收到psync命令后首先对参数runId是否与自身一致，如果一致，说明之前复制的是当前主节点；之后根据参数offset在复制积压缓冲区中查找，如果offset之后的数据存在，则对从节点发送CONTINUE命令，表示可以进行部分复制。因为**缓存区大小固定，若发生缓冲溢出，则进行全量复制**。
- 主节点根据偏移量把复制积压缓冲区里的数据发送给从节点，保证主从复制进行正常状态 [主从复制](https://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247485675&idx=1&sn=237472e5433310ae18892c8237a7ac4d&chksm=ebd637c7dca1bed1f8d8a61ca5afeac8e5e904e381e394fa4b6c53a4f8466503186cd1ca2a86&scene=21#wechat_redirect)

# 10、哨兵

==**面试官：主从复制会存在哪些问题呢？**==

- 一旦主节点宕机，从节点晋升为主节点，同时需要修改应用方的主节点地址，还需要命令所有从节点去复制新的主节点，整个过程需要人工干预
- 主节点的写能力收到单机限制
- 主节点存储能力收到单机限制
- 原生复制的弊端在早期版本比较突出，如：redis复制中断后，从节点会发起psync。如果同步不成功，会进行全量同步，**主库执行全量备份的同时，可能会造成毫秒或秒级的卡顿**

==**面试官：比较主流的解决方案是什么呢？**==

哨兵机制

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel.png)

redis sentinel 哨兵主要功能包括主节点存活检测、主从运行情况检测、自动故障转移、主从切换。redis sentinel最小配置一主一从。

redis 的sentinel可以用来管理多个redis服务器，该系统可以执行以下4个任务：

- 监控：不断检查主服务器和从服务器是否正常运行
- 通知：当被监控的某个redis服务器出现问题，sentinel通过api脚本向管理员或其他应用程序发出通知
- **自动故障转移**：当主节点不能正常工作时，sentinel会自动开始一次自动的故障转移操作，它会将与失效主节点是主从关系的其中一个从节点升级为新的主节点，并且将其他的从节点指向新的主节点，避免人工干预
- 配置提供者：在redis sentinel模式下，客户端应用在初始化时连接是sentinel节点集合，从中获取主节点的信息

==**面试官：哨兵的工作原理**==

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work.png)

- 1、每个sentinel节点都需要定期执行以下任务：**每个sentinel以每秒一次的频率**，向它所知的主服务器、从服务器以及其他的sentinel实例发送一个**PING命令**。如上图

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-1.png)

- 2、如果一个实例距离最后一次有效回复PING命令的时间超过**down-after-milliseconds**所指定的值，那么这个实例会被**sentinel标记为主观下线**。如上图

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-2.png)

- 3、如果一个主服务器被标记为主观下线，那么正在监视这个服务器的所有sentinel节点，要以**每秒一次的频率**确认主服务器的确进入了主观下线状态，如上图

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-3.png)

- 4、如果一个主服务器被标记为主观下线，并且有**足够数量的sentinel(至少要达到配置文件指定的数量)**在指定时间范围内同意这一判断，那么这个主服务器被标记为**客观下线**。如上图

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-4.png)

- 5、一般情况下，每个sentinel会以每10秒一次的频率向它已知的所有主服务器和从服务器发送INFO命令，当一个主服务器被标记为**客观下线**时，sentinel向下线主服务器所有服务器发送INFO命令的频率，会从10秒一次改为每秒一次

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-5.png)

- 6、sentinel和其他sentinel协商客观下线的主节点状态，如果处于SDOWN状态，则投票自动选出新的主节点，将剩余从节点指向新的主节点进行数据复制。

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-6.png)

- 7、当没有足够数量的sentinel同意主服务器下线时，主服务器的客观下线状态就会被移除。**当主服务器重新向sentinel的PING命令返回有效回复时**，主服务器的主观下线状态就会被移除。

## 哨兵机制优缺点

优点：

- 主从自动切换，系统更健壮，可用性更高
- sentinel会不断检查主服务器和从服务器是否运作正常，当被监控的某个redis服务器出现问题时，sentinel可以通过api向管理员或其他应用程序发送通知

缺点：

- 主从切换需要时间，会丢失数据
- 没有解决主节点写压力
- 动态扩容困难

# 11、redis 为什么会读到过期数据

由redis的删除策略导致：

- **惰性删除**：master节点每次读取命令时检查是否超时，如果超时则执行del命令删除键对象，之后异步把del命令发送给slave节点，保证数据复制的一致性，**slave节点永远不会主动删除超时数据**
- **定时删除**：redis的master节点内部定时任务，循环采样一定数量的建，当发现采样的键过期时，执行del命令，之后同步给slave节点，**采样速度跟不上数据过期速度**
- **主动删除**：当前已用内存超过maxMemory限定时，才会触发主动清理策略。主动删除的前提是设置了maxMemory值

> 根本原因是slave节点不能主动删除过期key。**redis 3.2**已经解决了redis删除策略导致的过期数据（slave节点读取数据之前会检查过期时间来决定是否返回数据）

# 12、redis缓存和数据库双写一致性问题

- **通过代码保证redis和关系型数据库的数据一致性是不可靠的**
- 可以使用**canal监控数据库的binlog将数据的变化实时同步到redis**

# 13、Redis 为什么之前一直不使用多线程，6.0 为什么又使用了？

使用 Redis 的**瓶颈主要是内存和网络，CPU 不会成为瓶颈**，使用多线程会增加线程切换的开销、还会带来一些并发读写的问题、增加了复杂度。

对于 80% 的公司来说，8w 到 10w 的 QPS 已经足够了，不过随着业务场景越来越复杂，有些公司动不动就上亿的交易量，因此需要更大的 QPS，采用分布式的 Redis 解决方案，增加了运维成本、服务器投入成本。所以 6.0 版本为了提高网络 IO 性能**使用了多线程来处理网络数据的读写和协议解析，==执行命令仍然是单线程的==**，而且默认是禁用多线程的，需要将 `io-threads-do-reads` 参数设置为 yes 才会启用多线程。

官方推荐至少要 4 核的机器才开启多线程，线程数的设置不能高于 cpu 核心数，而且超过 8 个基本上没有意义。

## 13.1、 Redis6.0多线程的实现机制？

主线程获取可读的 socket 会先放入一个等待队列，等待队列满了再交给一个 IO 线程组处理，**读取 socket 中的数据和协议解析由 IO 线程组完成，这里是并发执行的，提升了 IO 效率，期间主线程阻塞，IO 线程组处理完后，主线程才开始执行所有请求命令**。

# 14、为什么 bgsave 和 aof 重写的时候 rehash 的负载因子变成 5 了

因为 bgsave 和 aof 重写是 fork 一个子进程去做的，内核把父进程中所有内存页的权限都设置为 read-only，当父进程需要修改数据时需要拷贝一份数据页，**在 bgsave 和 aof 重写期间 rehash 可能造成大量的内存拷贝，性能低下**，所以这个期间触发 rehash 的条件从【负载因子 = 1】变成了 【负载因子 = 5】。

# redis cluster

Redis Cluster 采用**无中心结构**，**每个节点都可以保存数据**和整个集群状态，每个节点都和其他所有节点连接。

Cluster 一般由多个节点组成，节点数量至少为 6 个才能保证组成完整高可用的集群，其中三个为主节点，三个为从节点。

**三个主节点会分配槽，处理客户端的命令请求**，而从节点可用在主节点故障后，顶替主节点。

**优点**

- 去中心化；
- 可扩展性，数据按照 Slot 存储分布在多个节点，节点间数据共享，节点可动态添加或删除，可动态调整数据分布；
- 高可用性，部分节点不可用时，集群仍可用。通过增加 Slave 做备份数据副本；
- 实现故障自动迁移，节点之间通过 gossip 协议交换状态信息，用投票机制完成 Slave 到 Master 的角色提升。

**缺点**

- 数据通过异步复制，无法保证**数据强一致性**；
- 集群环境搭建略微复杂。

