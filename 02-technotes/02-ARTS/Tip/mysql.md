[TOC]

[极客时间mysql45讲](https://funnylog.gitee.io/mysql45/)

# mysql架构

## mysql的执行流程

**查询流程**

![img](https://github.com/lission/markdownPics/blob/main/mysql/MySQL%E6%89%A7%E8%A1%8C%E6%9F%A5%E8%AF%A2%E8%BF%87%E7%A8%8B.png?raw=true)

- 连接

- 查询缓存，mysql内部自带一个缓存模块，MySQL 的缓存默认是关闭的。**在 MySQL8.0 中，查询缓存已经被移除了。**

  > MySql自带缓存应用场景有限：
  >
  > - SQL 语句必须一模一样，中间多一个空格，字母大小写不同，都会被认为是不同的 SQL 
  > - 表中任意一条数据发生变化的时候，这张表的缓存都会失效。所以，对于有大量数据更新的应用，也不合适
  >
  > **所以缓存这一块，我们还是交给 ORM 框架（比如 MyBatis 默认开启了一级缓存），或者独立的缓存服务，比如 Redis 来处理更为合适。**

- 解析器，Parser 解析器，对语句基于 SQL 语法进行词法和语法分析，以及语义的解析。语法解析是根据 SQL 语句生成一个数据结构。**这个结构我们把它叫做解析树**
- 预处理器，Preprocessor，**它会检查生成的解析树，解决解析器无法分析的语义。比如，它会检查表和列名是否存在，检查名字和别名，保证没有歧义。**预处理之后得到一个新的解析树。
- 查询优化器，Optimizer，查询优化器的目的就是根据解析树生成不同的`执行计划（Execution Plan）`，然后选择一种最优的执行计划，MySQL 里面使用的是基于开销（cost）的优化器，哪种执行计划开销最小，就用哪种。
- 执行计划。Execution Plan，查询优化器基于解析树以最小开销原则生成选择执行计划
- 执行引擎和存储引擎，执行引擎利用存储引擎提供的相应的 API 来完成操作，最后把数据返回给客户端



## mysql架构分层

总体上，我们可以把 MySQL 分成三层。

- 跟客户端对接的连接层
- 执行操作的服务层
- 和硬件打交道的存储引擎层

![img](https://github.com/lission/markdownPics/blob/main/mysql/MySQL%20%E6%9E%B6%E6%9E%84%E5%88%86%E5%B1%82.png?raw=true)

## mysql中一条更新sql是如何执行的

update操作其实包括更新、插入和删除。更新流程与查询流程基本一致，也要经过解析器、优化器的处理最后交给执行器，区别在于拿到符合条件的数据之后的操作。

> MyBatis源码Executor里也只有doQuery()和doUpdate()方法，没有doDelete() 和 doInsert()

简化的更新操作流程：

- 1、事务开始，从内存（Buffer pool）或磁盘（data file）取到包含这条数据的数据页，返回给Server执行器
- 2、server执行器修改数据页的数据值
- 3、记录原数据至undo log
- 4、记录更新数据至redo log
- 5、调用存储引擎接口，记录数据页到Buffer pool
- 6、事务提交

**redo log 和 undo log与事务密切相关，统称为事务日志**



## mysql 主从复制过程

![img](https://github.com/lission/markdownPics/blob/main/mysql/mysql%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6.png?raw=true)

1. 当Master节点进行insert、update、delete操作时，会按顺序写入到binlog中。
2. slave从节点连接master主节点，Master有多少个slave就会创建多少个binlog dump线程。
3. 当Master节点的binlog发生变化时，binlog dump 线程会通知所有的slave节点，并将相应的binlog内容**推送给slave节点**。
4. slave节点的I/O线程接收到 binlog 内容后，将内容写入到本地的 relay-log。
5. slave节点的SQL线程读取I/O线程写入的relay-log，并且根据 relay-log 的内容对从节点做对应的操作。

**缺点**

尽管主从复制、读写分离能很大程度保证MySQL服务的高可用和提高整体性能，但是问题也不少：

- **从机是通过binlog日志从master同步数据的，如果在网络延迟的情况，从机就会出现数据延迟。那么就有可能出现master写入数据后，slave读取数据不一定能马上读出来**

**事务问题**：**同一线程且同一数据库连接内，如有写入操作，以后的读操作均从主库读取**，保证数据一致性。

## 主从复制的几种方式

[参考](https://www.zhihu.com/search?type=content&q=mysql+%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6)

mysql默认的复制是异步的

- 异步复制，主库写数据到BINLOG后，执行commit，返回客户端，主从binlog异步线程同步。

  > 有一个主（source）并且有一或多个从（replicas）。主数据库execute事务，将其commit，然后将它们稍后（异步）发送给从数据库

- 多线程复制，增加并发的binlog复制，提升异步复制的效率问题

  > 并行是指从库多线程apply binlog，**库级别并行应用binlog**，同一个库数据更改还是串行的(5.7版并行复制基于事务组)设置
  >
  > mysql 5.6之后为多线程复制

- 增强半同步复制

  - 传统半同步复制，**为了保证主库上的每一个BINLOG事务都能够被可靠地复制到从库上**，主库在每次事务成功提交时，并不及时反馈给前端应用用户，而是**等待至少一个从库也接收到BINLOG事务并成功写入中继日志relay log后**，主库才返回Commit操作成功给客户端

    > Mysql 5.5添加了半同步复制插件

  - 增强半同步复制，主库写数据到BINLOG后，就开始等待从库的应答ACK，直到**至少一个从库写入Relay Log后，并将数据落盘**，然后返回给主库消息，通知主库可以执行Commit操作，然后主库开始提交到事务引擎层



# 存储引擎

## MyISAM

> 不支持事务，性能更好，数据占用空间小。锁级别为表级锁，并发性差。

**数据表底层结构**

每张表对应生成三个文件

- frm文件：表结构定义文件，存储与表相关的元数据(meta)信息
- MYD文件：MyISAM存储引擎专用，存放MyISAM表的数据(data)
- MYI文件：MyISAM存储引擎专用，存放MyISAM表的索引相关信息

**mysql8.0从元数据中移除了frm文件**

**适用场景**

- 非事务型应用
- 只读类应用(可以压缩表)，存档数据

## InnoDB

> Mysql 5.7中的默认存储引擎，InnoDB是一个事务安全存储引擎。支持行级锁和表锁，支持读写并发，写不阻塞(MVCC)。InnoDB 将用户数据存储在`聚集索引`中(索引和记录在一起存储)，以减少基于主键的常见查询 I/O 。

InnoDB 对应两个文件： 表结构文件 `.frm `、数据文件 `.ibd`；表最大为 64TB。

**适用场景**

- 经常更新表，存在并发读写或者有事务处理的业务系统
- 并发业务多，需要考虑读写互不干扰，保证比较高的数据一致性的场景

## MyISAM和InnoDB对比

- 事务和外键
  - InnoDB支持事务，具有安全性和完整性，适合大量insert和update操作
  - MyISAM不支持事务，提供高速检索，适合大量select操作
- 锁机制
  - InnoDB支持行级锁和表级锁，基于索引实现
  - MyISAM只支持表级锁
- 索引结构
  - InnoDB使用**聚集索引(聚簇索引)，索引和记录在一起存储**
  - MyISAM使用**非聚集索引(非聚簇索引)，索引和记录分开存储**
- 并发处理能力
  - InnoDB读写阻塞与隔离级别有关，可以采用多版本并发控制MVCC支持高并发
  - MyISAM使用表锁，写操作并发率低，读写阻塞
- 存储文件
  - InnoDB 对应两个文件： 表结构文件 `.frm `、数据文件 `.ibd`；表最大为 64TB。
  - MyISAM 对应三个文件：表结构文件 `.frm`、表数据文件`.MYD`、索引文件`.MYI`；从 MySQL5.0 开始默认限制是 256TB
  - **mysql8.0从元数据中移除了frm文件**
- 适用场景
  - InnoDB读写阻塞与隔离级别有关，可以采用多版本并发控制MVCC支持高并发
    - 经常更新表，存在并发读写或者有事务处理的业务系统
    - 并发业务多，需要考虑读写互不干扰，保证比较高的数据一致性的场景
  - MyISAM
    - 非事务型应用
    - 只读类应用(可以压缩表)，存档数据

## InnoDB核心概念

### 缓冲池 Buffer Pool

对InnoDB存储引擎来说，数据都是放在磁盘上的，存储引擎要操作数据必须先把磁盘里面的数据加载到内存里面才可以操作。磁盘I/O的读写相对于内存操作很慢，操作系统、内存引擎都有一个**预读取**的概念。

buffer pool是一个**数据页的链表结构**

> 预读取：依据局部性原理（当磁盘上一块数据被读取时，很可能它附近的位置也会被读取。）每次多读取一点，而不是用多少读多少。***可以分为线性预读、随机预读***
>
> **InnoDB设定了存储引擎从磁盘读取数据到内存的最小单位，页page**。操作系统的默认page大小4kb，InnoDB默认page大小为16kb，如果修改值，需要清空数据重新初始化服务。

**InnoDB设计缓冲区Buffer Pool默认大小128M，作用就是来提升读写效率**。

- 读取数据时，先判断是不是在这个buffer pool，如果是直接读取；如果不是，再从磁盘加载。读取到的数据写到这个内存缓冲区。

- 修改数据时，也是先写到这个buffer pool，而不是直接写进磁盘。**内存数据页未同步至磁盘前，称之为脏页**。InnoDB有专门的后台线程把buffer pool数据写入磁盘，往磁盘同步时称之为**刷脏**。

#### 缓冲池管理，LRU

InnoDB按照page页的方式管理数据，使用LRU(最近最少使用)算法进行page管理。

**【普通 LRU 算法】**

LRU 列表头部为使用最频繁的 Page，尾部为最少使用的 Page。当内存不足继续从磁盘读取 Page 时，释放尾部的 Page。

**【存在的问题】**

有一些操作需要访问表中的许多页，甚至全部页（如：索引 or 数据的扫描操作），**这些页仅在这次查询操作中需要，并非热点数据**；

**如果直接把这些页放到 LRU 列表头部，那么就会有很多热点数据页被刷新出去，影响缓冲池效率，而且下次读取得重新访问磁盘。**

**优化行为**：

 [Section 15.8.3.3, “Making the Buffer Pool Scan Resistant”](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-midpoint_insertion.html)

 [Section 15.8.3.4, “Configuring InnoDB Buffer Pool Prefetching (Read-Ahead)”](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-read_ahead.html)

【引入改良版 LRU 算法】

- 使用 **中点插入策略（midpoint insertion strategy）**，把LRU list分成两部分，靠近head的叫做new sublist，存放热数据，叫它热区；靠近tail部分的叫做old sublist，存放冷数据，叫它冷区。中割线称为midpoint，**最新访问的页放入 LRU 列表的 midpoint 位置**（the head of the old sublist）。
- midpoint 可以通过参数 `innodb_old_blocks_pct` 设置，默认为 37，即 LRU 列表从尾部开始 37% 的位置（约 3/8）。热区占5/8，冷区占3/8

> [buffer bool](https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool.html)
>
> By default, the algorithm operates as follows:
>
> - 3/8 of the buffer pool is devoted to the old sublist.
> - The midpoint of the list is the boundary where the tail of the new sublist meets the head of the old sublist.
> - **When `InnoDB` reads a page into the buffer pool, it initially inserts it at the midpoint (the head of the old sublist)**. A page can be read because it is required for a user-initiated operation such as an SQL query, or as part of a [read-ahead](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_read_ahead) **(预读)** operation performed automatically by `InnoDB`.
> - Accessing a page in the old sublist makes it “young”, moving it to the head of the new sublist. If the page was read because it was required by a user-initiated operation, the first access occurs immediately and the page is made young. If the page was read due to a read-ahead operation, the first access does not occur immediately and might not occur at all before the page is evicted.
> - As the database operates, pages in the buffer pool that are not accessed “age” by moving toward the tail of the list. Pages in both the new and old sublists age as other pages are made new. Pages in the old sublist also age as pages are inserted at the midpoint. Eventually, a page that remains unused reaches the tail of the old sublist and is evicted.

### redo log

**问题**：刷脏不是实时的，如果buffer pool里的脏页还没有刷入磁盘，数据库宕机或重启，这些数据会丢失。

为了避免这个问题**（刷脏不及时导致的数据丢失）**，InnoDB把所有对页page的**修改操作**专门写入一个日志文件即**磁盘的redo log(重做日志)**，如果有未同步到磁盘的数据数据库在启动时，会从这redo log进行恢复操作(crash-safe)。事务中ACID中的D(持久性)就是用redo log实现的。

redo log位于/var/lib/mysql目录下，**ib_logfile0和ib_logfile1，默认2个文件，每个48M**

redo log特点：

- redo log是InnoDB存储引擎实现的，**支持崩溃恢复**是InnoDB的一个特性
- redo log不是记录数据页更新之后的状态，而是记录**在这个数据页上做了什么修改**。属于物理日志
- redo log **大小固定，前面的内容会被覆盖，一旦写满，就会触发buffer pool到磁盘的同步**，以便腾出空间记录后面的修改

> **同样是写磁盘，为什么不直接写到db file里面去？为什么先写日志再写磁盘？写日志文件和写到数据文件有什么区别？**
>
> - 刷盘是随机I/O，数据可能存储在磁盘不同的扇区中，找到对应的数据需要磁臂旋转到指定页，然后盘片找到对应扇区，找到需要的一块数据，直至找到所有数据，读写速度慢。
> - 记录日志是顺序I/O，找到了第一块数据，其他数据就在这块数据后边，不需要重新寻址。顺序I/O效率更高。
>
> **二者本质上是数据集中存储和分散存储的区别，先把修改写入日志文件，保证了内存数据的安全性的情况下，延迟刷盘时机，提升吞吐性。**

### undo log

又叫撤销日志或回滚日志，记录了事务发生之前的数据状态，分别为insert undo log和update undo log。修改数据时出现异常，**可以用undo log实现回滚操作（保持原子性）**

updo log可以理解为记录的是反向操作，比如insert 会记录delete，称为逻辑格式日志

 [Section 15.6.6, “Undo Logs”](https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-logs.html).

## InnoDB 内存结构

[官网地址](https://dev.mysql.com/doc/refman/8.0/en/innodb-architecture.html)

![innodb-architecture.png](https://github.com/lission/markdownPics/blob/main/mysql/innodb-architecture.png?raw=true)

- Buffer pool，**InnoDB设计缓冲区Buffer Pool默认大小128M，作用就是来提升读写效率**。管理存储预读取的数据页

  - Change Buffer，写缓冲，是Buffer Pool的一部分，如果一个数据页不是唯一索引，不存在数据重复情况，也就不需要从磁盘加载索引页判断数据是否重复(唯一性检查)。这种情况可以先把修改记录在内存的缓冲池中，提升更新语句的执行速度

- Log Buffer，日志缓存，日志信息会先放到缓冲区，然后按照一定频率刷新到日志文件。日志包含：

  - InnoDB 引擎日志
  - 数据库操作时产生的 redo 和 undo 日志

  > Log Buffer 写入磁盘的时机，由一个参数控制，默认是 1 。
  >
  > | 值                            | 含义                                                         |
  > | ----------------------------- | ------------------------------------------------------------ |
  > | 0（延迟写）                   | log buffer 将每秒执行一次的写入 log file 中，并且 log file 的 flush 操作同时进行。 该模式下，在事物提交的时候，不会触发主动写入磁盘的操作。 |
  > | ==1（默认，实时写，实时刷）== | ==每次事务提交时 MySQL 都会把 log buffer 的数据写入 log file。并且刷新到磁盘中去。== |
  > | 2 （延时写，延时刷）          | 每次事务提交时 MySQL 都会把 log buffer 的数据写入 log file，但是 flush 操作并不会同时进行。该模式下，MySQL 会每秒执行一次 flush 操作。 |

- Adaptive Hash Index，自适应hash索引，用户不可以显示的创建 Hash Index ，只能由 InnoDB 自己维护，一般用在 Buffer Pool 。**InnoDB 自动为 Buffer Pool 中的热点页创建的索引**）

## InnoDB 后台线程

后台线程的主要作用是**负责刷新内存池中的数据和把修改的数据页刷新到磁盘**。后台线程分为：master thread，IO thread，purge thread，page cleaner thread。

- **master thread** 负责刷新缓存数据到磁盘，并协调调度其他后台线程。
- **IO thread** 分为 insert buffer、log 、read 、write 进程。分别用来处理 insert buffer、重做日志、读写请求的 IO 回调。
- **purge thread** 用来回收 undo 页。
- **page cleaner thread** 用来刷新脏页。

## bin log

MySQL 的 Server 层也有一个日志文件，叫做 **binlog** ，它**可以被所有的存储引擎使用**。binlog 以事件的形式记录了所有的 DDL 和 DML 语句（因为**记录的是操作而不是数据值，数据逻辑日志**），可以用来做**主从复制和数据恢复**。
跟 redo log 不一样，它的内容是可以追加的，没有固定大小限制。在开启了 binlog 功能情况下，我们可以吧 binlog 导出成 SQL 语句，把所有操作重放一遍，来**实现数据恢复**

## bin log与redo log

二进制日志（binlog）是用来进行 POINT-IN-TIME（PIT）**特定时间点的恢复以及主从复制环境的建立**。

从表面上看，binlog 和 redo log 都是记录对数据库操作的日志，但两者有很大的不同：

- 产生位置不同：
  - redo log 只是 InnoDB 存储引擎层产生的
  - binlog 在任何存储引擎下都会产生
- 内容不同：
  - binlog 是逻辑日志，其记录的是对应的 SQL 语句
  - redo log 是物理日志，其记录的是对于每个页的修改
- 记录时间点不同：
  - binlog 只在事务提交完成后写入一次
  - redo log 在事务进行期间不断写入



# 索引

## 索引的分类

索引，数据库管理系统(DBMS)的一个**排序**的数据结构，以**协助快速查询、更新数据库表中的数据**。

### 应用层面

- 普通索引，也叫做非唯一索引，是普通索引，没有任何限制
- 唯一索引，要求键值不重复
- 主键索引，是一种特殊的唯一索引，它还多了一个限制条件，要求**键值不能为空**。主键索引用 **Primay key**创建
- 复合索引，又称为联合索引，是在多个列上创建的索引。创建复合索引最重要的是**列顺序**的选择，这关系到索引能否使用上，或者影响多少个谓词条件能使用上索引。**复合索引的使用遵循最左匹配原则**，只有索引左边的列匹配到，后面的列才能继续匹配

### 数据与键值逻辑

- 聚集索引，**(聚簇索引)，索引和记录在一起存储**，一个索引值对应一行记录。
  - 如果一张表创建了主键索引，那么主键索引就是聚簇索引
  - 如果没有主键，则为第一个非空唯一索引
  - 没有非空唯一索引，则为 InnoDB 内置的 ROWID
- 非聚集索引，**(非聚簇索引)，索引和记录分开存储**。二级索引/辅助索引，主键索引之外的索引叶子节点上存储的是这条**记录对应的主键的值**。

### 数据结构

- B树，是一种自平衡的m阶树，能够保持数据有序，B表示平衡balance。[数据结构说明](./Tip/数据结构.md)

- B+树，在B树基础上，为叶子节点增加链表指针（B树+叶子有序链表），所有关键字都在叶子节点上，非叶子节点作为叶子节点的索引。InnoDB 就是使用 B+树。[数据结构说明](./Tip/数据结构.md)

- Hash索引，查询效率高，可以一次性定位。在 InnoDB 中，不能显示地创建一个哈希索引（**所谓支持哈希索引指的 自适应哈希索引（Adaptive Hash Index），它是 InnoDB 自动为 Buffer Pool 中的热点页创建的索引**）。

  **Memory 存储引擎可以使用哈希索引**

  > 缺陷：
  >
  > - 不能用于范围查询
  > - 不能避免表扫描
  > - 大量hash重复时，性能欠佳

- FullText索引，全文索引，目前仅用于char、vachar、text这三种类型
- R树索引，比较少见，主要用于空间数据索引

## 索引的使用原则

- **列的离散度**，离散度公式：count(distinct(column_name)):count(1)，**列的全部不同值和所有数据行的比例。** **列的重复值越多，离散度就越低**，重复值越少，离散度就越高。**不建议在离散度低的字段上建立索引**
- **联合索引最左匹配原则**，联合索引在B+Tree中是复合的数据结构，按照从左到右的顺序建立搜索树
- **覆盖索引**，覆盖索引减少了IO次数，大大提升查询效率
  - 回表：非主键索引，先通过索引找到主键索引的键值，再通过主键值查出索引没有的数据，比基于主键索引的查询多扫描了一颗索引树
  - 覆盖索引：在**二级索引**里面，不管是单列索引还是联合索引，如果**select数据只用从索引中就能取得，不必从数据区中读取，避免回表，这时候的索引就叫覆盖索引**
- **索引条件下推**，index condition pushdown ，默认开启，只适用于二级索引。ICP目标是**减少访问表的完整行的读数量从而减少IO操作**。下推，指把过滤动作在存储引擎做完，不需要到server层过滤

## 索引失效

1. 索引列上使用函数(replace\SUBSTR\CONCAT\sum count avg)、表达式计算(+ - * /)

2. 字符串不加引号，**出现隐式转换**

3. like 前面带%
4. 负向查询，NOT LIKE 不能。!=、<>、和 NOT IN 在某些情况下可以。

**其实用不用索引最终都是优化器说了算**。优化器是**基于 cost 开销（Cost Base Optimizer），它不是基于规则（Rule-Based Optimizer），也不是基于语义。怎么开销小怎么来。**

## 索引创建

1. 在用于 where 判断、order 排序 、 join 的（on）和 group by 的字段上创建索引。


2. 索引的个数不要过多。浪费空间，更新变慢。


3. 过长的字段应使用前缀索引。

   ```sql
   CREATE TABLE `pre_test` (
   'content' varchar(20) DEFAULT NULL,
   KEY `pre_index` (`content`(6))
   )ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
   ```


4. 区分度低的字段，例如性别，不要建立索引。离散度太低，导致扫描行数过多。


5. 频繁更新的值，不要作为主键索或索引。导致频繁的**页分裂**。


6. 随机无规律的值，不建议作为索引，例如身份证号、UUID。无需，导致频繁的**页分裂**。


7. 组合索引把离散度高（区分度高）的值放在前面。


8. 创建复合索引，而不是创建单列索引



# 事务

## 事务特性

事务的四大特性ACID

- 原子性，Atomicity，一个事务内的操作要么全部成功，要么全部失败，即事务是不可分割的单位

  > 原子性，在 innoDB 里面是通过**undo log**来实现的，它记录了数据修改之前的值(是反向操作，比如insert 会记录delete)（逻辑日志），一旦发生异常，就可以用 **undo log** 来实现回滚操作。

- 一致性，Consistent，指的是**数据库的完整性约束没有被破坏，事务执行的前后都是合法的数据状态**。

  > 原子性，隔离性，持久性，最后都是为了实现一致性

- 隔离性，Isolation，**要求一个事务未提交前，别的事务不能感知到它的修改**。

  > 事务隔离两大实现方案：
  >
  > - 基于锁的并发控制(LBCC)，Lock Based Concurrence Control，为保证两次读取数据一致，那么读数据时，锁定要操作的数据，不允许其他事务修改。这意味着不支持并发读写，极大影响效率
  > - 多版本并发控制(MVCC)，Multi Version Concurrence Control，如果要让一个事务前后两次读取的数据保持一致，可以在修改数据之前给它建立一个备份叫快照，后面读取这个快照
  >
  > - 一个事务写操作对另一个事务**写操作的隔离使用锁机制保证**
  > - 一个事务写操作对另一个事务**读操作的隔离使用MVCC保证**

- 持久性，Durability，**事务一旦提交，结果就是永久性的，即使宕机重启后也能恢复**。

  > 持久性是通过 **==redo log== **和 **==double write buffer（双写缓冲）== **来实现的，我们操作数据的时候，会先写到内存的 **==buffer pool==** 里面，同时记录 **==redo log==** ，如果在刷盘之前出现异常，在重启后就可以读取 ==redo log== 的内容，写入磁盘，保证数据的持久性。
  >
  > [Section 15.6.4, “Doublewrite Buffer”](https://dev.mysql.com/doc/refman/8.0/en/innodb-doublewrite-buffer.html).



## 事务并发带来的问题

- **脏读**，一个事务里面，由于其他的事务修改了数据并且没有提交，而导致了前后两次查询到的数据不一致的情况。
- **不可重复读**，一个事物读取到了其他事务已提交的数据造成前后两次读取到数据不一致的情况。
- **幻读**，同一个事务前后两次读取数据不一致，是由于其他事务插入数据造成的

> **不可重复读和幻读的区别:**
>
> ==**修改或者删除**造成的读一不一致叫做不可重复读==，==**插入**造成的读不一致叫做幻读==。

无论是脏读，还是不可重复读，还是幻读，它们都是数据库的**==读一致性==**的问题，都是在一个事务里面前后两次读取出现不一致的情况。**读一致性的问题，必须要由数据库提供一定的事务隔离机制来解决。**



## 事务隔离级别、分别解决什么问题

|          事务隔离级别           |    脏读    | 不可重复读 |          幻读           |
| :-----------------------------: | :--------: | :--------: | :---------------------: |
|  未提交读（Read Uncommitted）   |    可能    |    可能    |          可能           |
|   已提交读（Read Committed）    | ==不可能== |    可能    |          可能           |
| **可重复读（Repeatable Read）** | ==不可能== | ==不可能== | ==**对InnoDB 不可能**== |
|     串行化（Serializable）      | ==不可能== | ==不可能== |       ==不可能==        |

InnoDB 支持这四个隔离级别，InnoDB 默认使用的 RR 作为事务隔离级。隔离界别越高，事务的并发度就越低。唯一的区别就在于，InnoDB 在 RR 级别就解决了幻读的问题。**因此不需要使用串行化的隔离界别去解决所有的问题，既保证了数据的一致性，又支持较高的并发度。**



## MVCC为什么需要、怎么实现的、解决什么问题

MVCC，Multi Version Concurrence Control，多版本并发控制。**MVCC用来解决读写互不阻塞和不可重复读问题**。为了让一个事务前后两次读取的数据保持一致，在修改数据之前给它建立一个备份(快照)。只在Repeatable Read和Read Committed两个隔离级别下工作。

**MVCC原则：**

- 一个事务能看到的数据版本
  1. 第一次查询之前已提交的事务修改
  2. 本事务的修改
- 一个事务不能看到的数据版本
  1. 本事务第一次查询之后创建的事务(**事务ID比本事务ID大**)
  2. 活跃(**未提交事务**)事务的修改

**MVCC效果**：

可以查到在这个事务开始之前已经存在的数据，即使它在后面被修改或删除。因此称之为快照，**不管别的事务做任何增删改查操作，只能看到第一次查询时看到的数据版本**。

**MVCC如何快照实现？是否会占用额外存储空间？**

- InnoDB的事务都是有编号的，且会不断递增。
- InnoDB为每行记录都实现了两个隐藏字段：
  - **DB_TRX_ID**，6字节，事务ID，数据是在哪个事务插入或修改为新数据的，记录当前事物ID
  - **DB_ROLL_PTR**，7字节，回滚指针，把它理解为删除版本号，数据被删除或记录为旧数据时，记录当前事务ID，没有修改或删除时是空的。指向一条undo log中的记录

MVCC中通过一个**Read View(可见性视图)**，来实现事务id判断。**每个事务维护一个自己的Read View**。Read View中保存了本事务id、活跃事务id、当前系统最大事务id

- ==**m_ids**==：表示在生成 Read View 时当前系统中获得读写事务的事务 ID 列表。

- ==**min_trx_id**==：表示在生成 Read View 时当前系统中活跃的读写事务 ID 中最小的事务 ID 。也就是 m_ids 的中的最小值。

- ==**max_trx_id**==：表示生成 Read View 时系统中应该分配给下一个事务的 ID 值。

- ==**creator_trx_id**==：表示生成该 Read View 的事务的 ID 。

**通过Read View数据结构，事务判断可见性规则：**

0. 从数据的最早版本开始判断（undo log）。
1. 数据版本 ==trx_id = creator_trx_id== ，本事务修改，可以访问。
2. 数据版本 ==trx_id < min_trx_id（未提交事务的最小 ID）== ，说明这个版本在生成 Read View 时已经提交，可以访问。
3. 数据版本的 ==trx_id > max_trx_id（下一个事务 ID）== ，这个版本是生成 Read View 之后才开启的事务建立的，不能访问。
4. 数据版本的 ==trx_id 在 min_trx_id 和 max_trx_id 之间==。如果在 ==m_ids== 中，不可以访问，如果不在 ==m_ids== 中，可以访问。
5. 如果当前版本不可见，就找 undo log 版本链中的下一个版本。

注意：**RR 中的 Read View 是事务第一次查询的时候建立的。RC 的Read View 是事务每次查询的时候建立的。**Oracle、Postgres 等等其他数据库都有 MVCC 的实现。

# 锁

## 共享锁和排它锁

- **共享锁(读锁)**，读取数据时的锁，其他事务可以并发读取数据，不能写
- **排它锁(写锁)**，写数据时的锁，其他事务既不能读，也不能写

## 锁粒度(表级锁、行级锁)

- MyISAM 只支持表锁，用 lock table 的语法加锁。
- InnoDB 同时支持表锁和行锁

## 意向锁

**InnoDB意向锁是一种不与行级锁冲突的表级锁**，InnoDB意向锁是InnoDB引擎自动加的，是**一种标志性的表级锁**。

- 当我们**给一行数据加上共享锁之前**，数据库自动**在这张表上加一个意向共享锁**
- 当我们**给一行数据加上排他锁之前**，数据库自动**在这张表上加一个意向排他锁**

如果一张表上面**至少有一个意向共享锁\意向排他锁**，则说明**有其他的事务给其中的某些数据行加上了共享锁\排它锁**。

**意向锁与意向锁不冲突，意向锁与行锁也不冲突**。

如果没有意向锁的话，我们准备给一张表加上表锁时，首先要**判断有没有其他事务锁锁定了其中的某些行**，如果数据量特别大，**加表锁效率很低**。引入意向锁后，我们只需判断这张表上面有没有意向锁，如果有，直接返回失败。如果没有，可以加锁成功。**因此InnoDB的表级锁—意向锁，可以理解为一个标志，同来提高加锁的效率**。

## InnoDB行级锁算法

[参考](https://zhuanlan.zhihu.com/p/344541644)

首先解释一下范围概念：

- **Record Lock，记录锁**，**锁定一个index record(对索引进行加锁)**，当我们对唯一性索引（包含唯一索引和主键索引）使用等值查询，精准匹配到一条记录时，使用记录锁。即使表没有创建索引，InnoDB通过内置的Row ID作为聚簇索引来进行记录锁。
- **Gap Lock，间隙锁**，锁定一个索引记录区间，**查询未命中index record**时(无论是等值查询还是范围查询)，采用间隙锁。**间隙锁在Repeatable Read以上的隔离级别才可用。间隙锁为了解决同一事务中出现幻读的问题。**
- **Next-key Lock，临键锁，相当于记录锁+间隙锁**，InnoDB在Repeatable Repeat隔离级别下使用临键锁作为默认的行锁算法。**锁定下一个key的左开右闭区间(解决幻读问题)**

InnoDB的Repeatable Read隔离级别下的等值查询锁算法示例：

- 主键索引(聚簇索引)，**对主键索引记录加记录锁**
- 唯一索引，对辅助索引加记录锁，对主键索引也加记录锁
- 普通索引，对相关的辅助索引加临键锁，对对应的主键索引加记录锁
- 无索引，对全表加临键锁

**总结**：

1. **InnoDB** 中的`行锁`的实现依赖于`索引`，一旦某个加锁操作没有使用到索引，那么该锁就会退化为`表锁`。

2. **记录锁**存在于包括`主键索引`在内的`唯一索引`中，锁定单条索引记录。

   > 查询语句必须为`精准匹配`（`=`），不能为 `>`、`<`、`like`等，否则也会退化成间隙锁

3. **间隙锁**存在于`非唯一索引`中，锁定`开区间`范围内的一段间隔，它是基于**临键锁**实现的。

   > 查询未命中索引记录

4. **临键锁**存在于`非唯一索引`中，该类型的每条记录的索引上都存在这种锁，它是一种特殊的**间隙锁**，锁定一段`左开右闭`的索引区间。

   > 不仅命中记录还包含了间隙时使用临键锁。
   >
   > 唯一索引，等值匹配到一条记录时，退化为记录锁
   >
   > 没匹配到任何记录，退化成间隙锁



## 事务隔离级别选择

innodb 默认事务隔离级别 rr，ru、serializable肯定不能用。有些推荐使用RC。

**RC与RR主要区别：**

1. **RR的间隙锁会导致锁定范围变大**

2. **条件列未使用到索引，==RR锁表，RC锁行==**

3. RC的半一致性读可以增加update操作的并发性。

   > rc中，一个update语句，如果读到一行已经加锁的记录，此时InnoDB会返回记录最近提交的版本，又Mysql上层判断此版本师傅满足update where条件。若满足，则mysql重新发起一次读操作，此时会读取行的最新版本

实际上，如果能够正确使用锁**(避免不使用索引去加锁)**，只锁定需要数据，默认RR级别就可以了。



# mysql 调优

## 架构优化

**项目架构决定性能，优秀的架构胜过一万次的调优。**

> 一个单节点(一台应用服务器+一台数据库服务器)的系统架构，无论如何调优也不可能让系统达到百万级并发。在一台性能相对不错的物理机上，mysql最多能承载3500~4500的QPS。
>
> 通过架构优化可以轻易解决高并发问题。

**打到mysql上的请求无非是读和写**，我们可以分两种情况来处理

- 解决**读IO**问题，比如通过redis进行热点数据缓存，高并发时，读IO大部分都走redis了，减轻mysql负担
- 解决**写IO**问题，当大量写需求到达mysql时，可以通过在mysql前**加上消息队列相关组件**，让写请求先进队列，然后再慢慢一次对mysql进行写操作，减轻mysql写请求IO

进一步可以继续通过**增加集群**来进行优化，比如redis集群、mq集群等。

### 缓存

通过对热点数据的缓存，减轻数据库的压力，提升查询效率，可以使用redis等缓存工具。

### 集群、主从复制

增加mysql节点，mysql在server层会在binlog中记录所有 DDL 和 DML 语句，**mysql的主从节点之间通过binlog保持主从数据同步**。

主从同步涉及三个线程：

- I/O thread，从节点获取到master节点的binlog后，解析binlog写入relay log(中继日志)
- log dump thread，maste节点上的log dump thread 用来发送binlog给slave
- sql thread，从节点的sql thread用来读取relay log，把数据写入到slave数据库

做了主从配置方案后，我可以进行**读写分离**，只把数据写入master节点，而读的请求可以分担到slave节点

### 分库分表

在mysql集群架构中，所有节点存储的都是相同的数据。如果单表存储的数据过大，我们就可以进行分库分表，**把单个节点的数据分散到多个节点存储，减少存储和访问压力**

分库分表总体可以分两类：

- **垂直分库，减少并发压力**，把一个数据库按照业务拆分成不同数据库
- **水平分库分表，解决存储瓶颈**，把单张表数据按照一定规则分不到多个数据库

## 调优工具

### 慢日志查询

mysql_slow_log，打开慢日志开关，开启慢查询日志是有代价的，所以它默认关闭。

### explain执行计划

explain只适用于select语句

**字段详解：**

| 字段          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | id 越大越先执行，id 相同执行顺序由上至下，结果集的 id 为 null |
| select_type   | 查询类型：                                                   |
| table         | 这一步执行计划涉及到的表                                     |
| partitions    | 分区                                                         |
| type          | 表示关联类型或访问类型，即 MySQL 决定如何查找表中的行，null>system>const>eq_ref>ref>fulltext>range>index>all |
| possible_keys | 这一步执行计划可能使用哪些索引，如果为 NULL，可以看下 WHERE 子句能够创造一个适当的索引来提高查询性能 |
| key           | 实际使用的索引                                               |
| key_len       | 使用的索引的长度，不损失精确性的前提下，越小越好； 如果太长，MySQL 会做一个类似做前缀索引的处理，将前半部分的字符提取出来做索引 |
| ref           | 显示哪些列会与 key 列中的索引进行比较                        |
| rows          | MySQL 要读取并检测的行数【估计值】                           |
| filtered      | 经过服务器过滤后，实际返回给客户端的数据是多少（%, 百分比）  |
| Extra         | Useing index 等等                                            |



## 调优思路

- 对最需要优化的 SQL 进行调优（并发量高的）
- 从 EXPLAIN 执行计划入手
  - 永远用小结果集作为驱动表
  - 尽可能在索引中完成排序
  - 只取出需要的列，不要使用 SELECT *
  - 只选用最有效的过滤条件，过滤条件并不是越多越好
  - 尽可能避免复杂的 JOIN 和子查询（每个 SQL 不要超过 3 张表，表越多锁定的资源越多，对性能影响越大）
  - 特别是表数据量很大的情况下，尽量单表查询，在应用层对数据进行组装
  - 谨慎使用 ORDER BY、GROUP BY、DISTINCT 语句，它们都需要排序，尽量走索引
  - 合理设计并利用索引（组合索引何时生效）

## JOIN 调优

进行多表连接查询时，用小结果集作为驱动表，称为驱动表后，该表的结果集会被一条条地循环遍历，与被驱动表记录对比筛选，因此驱动表的数据越少，循环次数越少，I/O次数也越少。

- 尽可能减少 join 语句中 Nested Loop 嵌套循环的总次数；
- 优先优化 Nested Loop 嵌套循环的内层循环；
- 在被驱动表的关联字段上创建索引；
- 保证 join 语句中被驱动表上 join 连接条件字段是索引，如果无法保证，就不要吝啬 `join_buffer_size` 的设置（内存充足的前提下）;
- 复杂的 join 语句需要锁定的资源越多，所阻塞的线程也就越多，并发高的时候系统整体性能可能会急剧下降
  - 将复杂的 join 语句拆分成多个简单的语句分步执行
  - 使用 NoSQL 缓存

## ORDER BY 调优

- order by 的字段加索引（扫描索引，在内存中完成，属于逻辑 IO），不加索引可能会启用一个临时文件辅助排序（物理IO）；

- 不能利用索引进行排序时，MySQL 会使用`Using filesort`，这并不一定是文件排序，也可能是内存排序，主要由`sort_buffer_size`参数【默认 256K】与结果集大小决定；MySQL 内部实现排序主要有 3 中模式：
  - 快速排序
  - 归并排序
  - 堆排序

## GROUP BY调优

GROUP BY 本质上是先进行排序（MySQL 8 优化了，默认不排序）再分组。

GROUP BY 还经常搭配一些聚合函数使用。

**与 ORDER BY 一样，GROUP BY 也可以用索引优化，还可以考虑优化为子查询、添加查询条件**。

## LIMIT 调优

分页查询，偏移量越大查询越慢，如何优化？

- 利用索引覆盖，只查询索引列

- **引入子查询，通过 WHERE 子句利用主键过滤，外层查询 LIMIT 的偏移量就可以从 0 开始**

  > ```sql
  > SELECT * FROM test WHERE id>= (SELECT id FROM test LIMIT 10000,1) LIMIT 100;
  > ```

- 使用有索引的列或主键进行 ORDER BY

## Count 调优

- count(*)，统计结果集的行数
- count(col)，统计某个列值的数量，不统计 NULL

通常来说 COUNT() 都需要扫描大量的行才能获得精确的结果，因此是很难优化的，**唯一能做的就是索引覆盖扫描**，使用：

```
COUNT(主键列)
```

如果只要一个近似值，可以通过执行 EXPLAIN 来获得估算的 rows 行数。

