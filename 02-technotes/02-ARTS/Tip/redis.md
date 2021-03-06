[TOC]

# 1、Redis是什么

==面试官：你先来说下redis是什么吧==

redis是C语言开发的一个开源的高性能key-value键值对内存数据库。可以用作数据库、缓存、消息中间件等。是一种非关系型数据库。

redis内存数据库的特点：

- 性能优秀，数据在内存中，读写非常快，支持10WQPS
- 单进程单线程，线程安全，采用IO多路复用机制
- 丰富的数据类型，支持字符串(string)、散列(hash)、列表(list)、集合(set)、有序集合(zSet)
- 支持数据持久化，可以将内存中数据存储在磁盘，重启时加载
- 主从复制，哨兵机制，高可用
- 可以用作分布式锁
- 可以用作消息中间件，支持发布订阅

# 2、五种数据类型

==面试官：总结的不错，看来是早有准备啊。刚来听你提到redis支持五种数据类型，那你能简单说下这五种数据类型吗？==

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redisObject.png)

redis内部使用redisObject来表示所有的key和value，redisObject主要信息如上，type表示value对象具体是何种数据类型，encoding是不同数据类型在redis内部的存储方式。比如：type=string表示value存储的是一个普通字符串，那么encoding可以是raw或者int。

简单说一下5种数据类型：

- **string是redis基本数据类型**，一个key对应一个value，==value不仅可以是字符串也以是数字==。string类型是二进制全的，==redis的string可以包含任何数据，比如jpg图片或者序列化的对象==。string类型的值最大能存储512M。

- **hash是一个键值（key-value）的集合**。redis的hash是一个string的key和value的映射表，hash特别适合存储对象。

  - 常用命令hget,hset,hgetall等

- **list列表是简单的字符串列表，按照插入顺序排序**，可以添加一个元素到列表的头部(左边)或尾部(右边)，常用命令：lpush、rpush、lpop、rpop、lrange(获取列表片段等)
  - 应用场景:list应用场景非常多，比如twitter的关注列表，粉丝列表等都可以用list结构实现
  - 数据结构：list就是链表，redis提供了List的push和pop操作
  - 实现方式：redis list的实现是一个双向链表，支持反向查找和遍历，方便操作，但是带来了额外内存开销

- **set是string类型的无需集合**，集合通过hashtable实现，set中元素无序，且没有重复的

  - 常用命令：sdd、spop、smembers、sunion等
  - 应用场景：redis set对外提供功能与list一样是一个列表，特殊在于set自动去重，set提供判断某个成员是否在一个set集合中

- **zset和set一样是string类型元素的集合，不允许重复元素**。

  - 常用命令：zadd，zrange、zrem、zcard
  - 使用场景：sorted set可以通过用户额外提供一个优先级(score)的参数来为成员排序，且插入是有序的。和set相比，sorted set关联了一个double类型权重的参数score，使集合中的元素能够按照scord进行有序排序，redis是通过分数为集合中的成员进行从小到大的排序
  - 实现方式：sorted set内部使用hashMap和跳跃表skipList保证数据存储和有序，HashMap里存放的是成员到score的映射，而跳跃表存放的是所有成员，排序依据HashMap里存的score，跳跃表的结构可以或得较高的查找效率

  数据类型应用场景总结

  | 类型       | 简介                                                    | 特性                                                         | 场景                     |
  | ---------- | ------------------------------------------------------- | ------------------------------------------------------------ | ------------------------ |
  | string     | 为禁止安全                                              | 可以包含任何数据，比如jpg图片或序列化对象                    | ……                       |
  | Hash       | 键值对集合，编程语言中的map类型                         | 适合存储对象，可以像数据库中的update一样，只修改某一项属性值 | 存储、读取修改用户属性   |
  | List       | 链表，双向链表                                          | 增删快，提供了操作某一元素的api                              | 最新消息排行，消息队列   |
  | set        | hashTable实现，元素不重复                               | 添加、删除、查找的复杂度都是O(1)，提供求交际、冰机】差集操作 | 共同好友；利用唯一性     |
  | sorted set | 将set中的元素增加一个权重参数score，元素按score有序排列 | 数据插入集合石，进行了天然排序                               | 排行榜，带权重的消息队列 |

  

# 3、redis缓存

==面试官：说一下redis缓存怎么用的==

结合springboot使用的，一搬两种方式：

- 直接通过RedisTemplate使用
- 使用spring cache集成Redis



# 4、缓存问题

==面试官：实际项目中使用缓存有遇到什么问题或会遇到什么问题吗？==

缓存和数据库数据一致性问题，分布式环境下非常容易出现缓存和数据数据一致性问题，如果项目对缓存要求强一致性，那就不要使用缓存。只能采取合适的策略降低缓存和数据库间数据不一致的概率，而无法保证两者间的强一致性。合适的策略包括缓存更新策略，更新数据库后及时更新缓存、缓存失败时增加重试机制

==面试官：redis雪崩了解吗==

目前电商首页和热点数据都会去做缓存，一版缓存都是定时任务去刷新，或者查不到之后去更新缓存。如果Redis缓存同一时间大面积失效，大数量级的请求直接到数据库，导致数据库服务崩溃，这就是缓存雪崩。

==面试官：如何处理缓存雪崩==

在批量往redis存数据的时候，把每个key的失效时间加个随机值，保证数据不会再同一时间大面积失效。

如果redis是集群部署，将热点数据均匀分布在不同的redis库中

==面试官：了解缓存穿透和击穿吗，他们和缓存雪崩的区别是什么？==

- 缓存穿透：指缓存和数据库中都没有的数据，用户不断发起请求，导致数据库压力很大，严重时会击垮数据库
- 缓存击穿：与缓存雪崩相似，缓存雪崩是因为大面积缓存失效，打崩了DB，缓存击穿是指一个key非常热点，不停的扛着大量请求，大并发集中对一个点进行访问，在这个key失效的瞬间，持续的大并发直接落到了数据库上，就在这个key的点上击穿了缓存

==面试官：如何解决缓存穿透和击穿呢==

缓存穿透在接口层增加校验，比如用户鉴权，参数校验，不合法参数校验直接return。或者使用布隆过滤器，利用高效的数据结构和算法快速判断这个key是否存在书库中，不存在直接return。

缓存击穿，可以对热点数据设置永不过期，或者加上互斥锁

# 5、redis为什么这么快

==面试官：redis是单线程的，为什么还能这么快==

redis是单进程单线程模型

- redis完全基于内存，绝大部分请求是纯粹的内存操作
- 数据结构简单，对数据操作也简单，数据存在内存中类似于HashMap，查找和操作的时间复杂度是O(1)
- 采用单线程，避免了不必要的上下文切换和竞争条件，不存在多线程导致的cpu切换，无需考虑各种锁的问题，不存在加锁释放锁操作，没有死锁问题导致的性能消耗
- 采用多路复用IO模型，非阻塞IO，[参考](https://mp.weixin.qq.com/s?__biz=MzU2MTI4MjI0MQ==&mid=2247488306&idx=1&sn=8497ae941b5a13de03e9b94451770a0d&chksm=fc7a7e9ccb0df78a265c3fddc417bba21a76da913e45d1173a6a670156e868f96022935f3d9c&scene=21#wechat_redirect)

# 6、redis和Memcached区别

==面试官：为何选择redis而不用memcached==

- 存储方式上，memcached会把数据全部放在内存中，不会持久化到磁盘，断电或重启后数据小时，数据不能超过内存大小。redis会支持数据持久化
- 数据支持类型上：memcached对数据类型支持简单，只支持简单的key-value，redis支持五种数据类型
- 底层模型不同，他们之间底层实现方式以及客户端之间的通信应用协议不一样，redis直接自己构建了VM机制，因为一版的系统调用系统函数，会浪费一定时间去移动和请求
- value的肖肖，redis可以达到1GB，memcache只有1MB

# 7、淘汰策略

==面试官：redis的淘汰策略有哪些==

redis有六种淘汰策略

| 策略             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| volitile-lru     | 从已设置过期时间的KV集中优先对==最近最少使用(less recently used)==的数据淘汰 |
| volitile-ttl     | 从已设置过期时间的KV集中优先对==剩余时间短(time to live)==的数据淘汰 |
| voliitile-random | 从已设置过期时间的KV集中==随机选择数据淘汰==                 |
| allkeys-lru      | 从所有KV集中优先对==最近最少使用(less recently used)==的数据淘汰 |
| allkeys-random   | 从所有KV集中==随机选择数据淘汰==                             |
| noeviction       | 不淘汰策略，若超过最大内存，返回错误信息                     |

redis4.0加入了LFU(least frequency use)淘汰策略，包括volitile-lfu和allkeys-lfu,通过统计访问频率，将访问频率最少，即最不经常使用的KV淘汰

# 8、持久化

==面试官：你对redis的持久化机制了解吗，能讲一下吗==

redis为了保证效率，数据缓存在了内存中，但是会周期性的把更新的数据写入磁盘或把修改操作写入追加的记录文件，以保证数据的持久化。redis的持久化策略有两种：

- RDB：快照形式是直接把内存中的数据保存到一个dump文件中，定时保存，保存策略
- AOF：把所有的对redis的服务器进行修改的敏力都存到一个文件里，命令的集合。redis默认是快照rdb的持久化方式

redis重启时，会优先使用AOF文件来还原数据集，因为AOF文件保存的数据集通常比RDB文件保存的数据集更完整。[redis持久化机制](https://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247489359&idx=1&sn=30367a57e57aa107511be22cf41e8c18&chksm=ebd62863dca1a175197409337f22b532177a392b4fe9f289fdf416617a462f376e44663c5290&scene=21#wechat_redirect)

==面试官：RDB是怎么工作的？==

默认redis会议快照RDB的形式将数据持久化到磁盘的一个二进制文件dump.rdb。工作原理简单说一下：当redis需要做持久化时，redis会fork一个子进程，子进程将数据写到磁盘上一个临时RDB文件中。当子进程完成写临时文件后，将原来的RDB替换掉，好处是可以copy-on-write。

RDB的优点是，这种文件非常适合备份，可以在最近的24小时内，每小时备份一次，并且每个月的媒体天备份一个RDB文件，这样即使遇到问题，也可以随时将数据集还原到不同版本。RDB非常适合灾难恢复

RDB的缺点是，恢复时会丢失部分数据

==面试官：请再说一下AOF==

使用AOF做持久化，每一个写命令都通过write函数追加到appendonly.aof中，配置方式如下

```
appendfsync yes   
appendfsync always     #每次有数据修改发生时都会写入AOF文件。
appendfsync everysec   #每秒钟同步一次，该策略为AOF的缺省策略。
```

AOF可以做到全程持久化，只需要在配置中开启appendonly yes。这样redis每执行一个修改数据的命令，都会把它添加到AOF文件中，当redis重启时，将会读取aof文件进行重放，恢复到redis关闭前的最后时刻。

使用aof的优点是会让redis变得非常耐久。可以设置不同的fsync策略，aof默认策略是每秒钟fsync一次。

缺点是对于相同的数据集来说，aof的文件体积按通常要大于rdb文件体积。根据所使用的sync策略，aof的速度可能慢于rdb

==面试官：如何选择持久化机制==

如果非常关心数据，单仍然可以承受数分钟的数据丢失，那么可以只使用RDF持久。AOF将redis执行的每一条写命令追加到磁盘内，处理巨大的写入会降低redis性能。

数据备份和灾难恢复：定时生成rdb快照非常便于进行数据库备份，并且rdb恢复数据集的速度也要比aof恢复速度快。

redis支持同时开启rdb和aof，系统重启后，redis会优先使用aof来恢复数据，这样丢失的数据最少。

# 9、主从复制

==面试官：redis单节点存在单点故障问题，为解决单点问题，一版都需要对redis配置从节点，然后使用哨兵来监听主节点存活状态，如果主节点挂掉，从节点能继续提供缓存功能，你能说说redis主从复制的过程和原理吗==

主从配置结合哨兵机制能解决单点故障问题，提高redis可用性。从节点仅提供读操作，主节点提供写操作。对于读多写少的状况，可以给主节点配置多个从节点，从而提高响应效率。

主从复制过程：

- 从节点执行slaveof(masterIP)(masterPort),保存主节点信息
- 从节点中的定时任务发现主节点信息，建立和主节点的socket连接
- 从节点发送ping信号，主节点返回pong门两边能互相通信
- 连接建立后，主节点将所有数据发送给从节点(数据同步)
- 主节点把当前的数据同步给从节点后，便完成了复制的建立过程。接下来，从节点就会持续的把写命令发送给从节点，保证主从数据一致性

==面试官：你能详细说一下数据同步过程吗==

redis2.8之前使用sync(runId)(offset)同步命令，reidis2.8之后使用psync(runId)(offset)命令。[redis集群和mysql主从同步](http://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247490339&idx=1&sn=355246186ce9cad69a8a4fe578141ffb&chksm=ebd6240fdca1ad19a363b3c0a917352f78ca4cdb20c79f639298ef731295bde3753fd55cf539&scene=21#wechat_redirect)

两者不同在于sync命令仅支持全量复制过程，psync支持全量和部分复制。介绍同步之前，先介绍几个概念：

- runId：每个redis节点启动都会生成唯一的uuid没每次redis重启后，runId都会发生变化
- offset：主节点和从节点都各自委会自己的==主从复制偏移量offset==，当主节点有写入命令时，offset=offset+命令的字节长度。从节点在收到主节点发送的命令后，会增加自己的offset，并把自己的offset发送给主节点。这样，主节点同时保存自己的offset和从节点的offset，通过比对offset来判断主从节点数据是否一致。
- repl_backlog_size：保存在主节点上的一个固定长度的先进先出队列，默认大小是1MB。
- 主节点发送数据给从节点过程中，主节点还会进行一些写操作，这是数据存储在复制缓冲区中。从节点同步主节点数据完成后，主节点将缓冲区的数据发送给从节点，用于部分复制。
- 主节点响应写命令时，不但会把命名发送给从节点，还会写入复制积压缓冲区，用于赋值命令丢失的数据补救。

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-psync.png)

从节点发送psync(runId)(offset)命令，主节点有三种响应：

- FULLRESYNC：第一次连接，进行全量复制
- CONTINUE：进行部分复制
- ERR：不支持psync命令，进行全量复制

==面试官：能具体说一下全量复制和部分复制的过程吗==

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-allcopy.png)

全量复制过程：

- 从节点发送psync ? -1 命令（因为第一次发送，不知道主节点runid，所以为?，因为是第一次复制，所以offset=-1）。
- 从节点接收主节点信息后，保存到info中
- 主节点在发送FULLRESYNC后，启动bgsave命令，生成RDB文件
- 主节点发送RDB文件给从节点。到从节点加载数据完成这段期间主节点的写命令放入缓冲区。
- 从节点清理自己的数据库数据
- 从节点加载RDB文件，将数据保存到自己的数据库中
- 如果从节点开启了AOF，从节点会异步重写AOF文件

部分复制：

- ==部分复制主要是redis针对全量复制的过高开销做出的一种优化措施==，使用psync(runId)(offset)命令实现。当从节点正在复制主节点时，如果出现网络闪断或命令丢失等异常情况，从节点会向主节点要求补发丢失的命令数据，主节点的**复制积压缓冲区**将这部分数据直接发送给从节点，保持主从节点复制的一致性。补发的这部分数据一搬远远小于全量数据。
- 主从连接中断期间主节点依然响应命令，但因复制连接中断命令无法发送给从节点，不过***主节点内的复制积压缓冲区依然可以保存最近一段时间的写命令数据***。
- 当主从连接恢复后，由于从节点之前保存了自身已复制的偏移量和主节点的运行id，因此会把它们当做psync参数发送给主节点，要求进行部分复制。
- 主节点接收到psync命令后首先对参数runId是否与自身一致，如果一致，说明之前复制的是当前主节点；之后根据参数offset在复制积压缓冲区中查找，如果offset之后的数据存在，则对从节点发送+CONTINUE命令，表示可以进行部分复制。因为缓存区大小固定，若发生缓冲溢出，则进行全量复制。
- 主节点根据偏移量把复制积压缓冲区里的数据发送给从节点，保证主从复制进行正常状态 [主从复制](https://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247485675&idx=1&sn=237472e5433310ae18892c8237a7ac4d&chksm=ebd637c7dca1bed1f8d8a61ca5afeac8e5e904e381e394fa4b6c53a4f8466503186cd1ca2a86&scene=21#wechat_redirect)

# 10、哨兵

==**面试官：主从复制会存在哪些问题呢？**==

- 一旦主节点宕机，从节点晋升为主节点，同时需要修改应用方的主节点地址，还需要命令所有从节点去复制新的主节点，整个过程需要人工干预
- 主节点的写能力收到单机限制
- 主节点存储能力收到单机限制
- 原生复制的弊端在早期版本比较突出，如：redis复制中断后，从节点会发起psync。如果同步不成功，会进行全量同步，主库执行全量备份的同时，可能会造成毫秒或秒级的卡顿

==**面试官：比较主流的解决方案是什么呢？**==

哨兵机制

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel.png)

redis sentinel 哨兵主要功能包括主节点存活检测、主从运行情况检测、自动故障转移、主从切换。redis sentinel最小配置一主一从。

redis 的sentinel可以用来管理多个redis服务器，该系统可以执行以下4个任务：

- 监控：不断检查主服务器和从服务器是否正常运行
- 通知：当被监控的某个redis服务器出现问题，sentinel通过api脚本向管理员或其他应用程序发出通知
- 自动故障转移：当主节点不能正常工作时，sentinel会自动开始一次自动的故障转移操作，它会将与失效主节点是主从关系的其中一个从节点升级为新的主节点，并且将其他的从节点指向新的主节点，避免人工干预
- 配置提供者：在redis sentinel模式下，客户端应用在初始化时连接是sentinel节点集合，从中获取主节点的信息

==**面试官：哨兵的工作原理**==

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work.png)

- 1、每个sentinel节点都需要定期执行以下任务：每个sentinel以每秒一次的频率，向它所知的主服务器、从服务器以及其他的sentinel实例发送一个PING命令。如上图

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-1.png)

- 2、如果一个实例距离最后一次有效回复PING命令的时间超过down-after-milliseconds所指定的值，那么这个实例会被sentinel标记为主观下线。如上图

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-2.png)

- 3、如果一个主服务器被标记为主观下线，那么正在监视这个服务器的所有sentinel节点，要以**每秒一次的频率**确认主服务器的确进入了主观下线状态，如上图

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-3.png)

- 4、如果一个主服务器被标记为主观下线，并且有足够数量的sentinel(至少要达到配置文件指定的数量)在指定时间范围内同意这一判断，那么这个主服务器被标记为客观下线。如上图

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-4.png)

- 5、一般情况下，每个sentinel会以每10秒一次的频率向它已知的所有主服务器和从服务器发送INFO命令，当一个主服务器被标记为客观下线时，sentinel向下线主服务器所有服务器发送INFO命令的频率，会从10秒一次改为每秒一次

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-5.png)

- 6、sentinel和其他sentinel协商客观下线的主节点状态，如果处于SDOWN状态，则投票自动选出新的主节点，将剩余从节点指向新的主节点进行数据复制。

![img](https://raw.githubusercontent.com/lission/markdownPics/main/redisPic/redis-sentinel-work-6.png)

- 7、当没有足够数量的sentinel同意主服务器下线时，主服务器的客观下线状态就会被移除。当主服务器重新向sentinel的PING命令返回有效回复时，主服务器的主观下线状态就会被移除。

# 11、参考

https://juejin.im/post/5b7d226a6fb9a01a1e01ff64
