[TOC]

# 集合
## ArrayList与LinkedList的区别

1、底层数据结构不同，ArrayList底层是数组，LinkedList是双向链表

2、ArrayList容量有限制，可以指定容量，默认容量是10，当超过容量时需要扩容，每次扩容50%。LinkedList没有容量限制

3、ArrayList和LinkedList都实现了List接口，LinkedList还实现了Deque接口，可以当做双端队列或栈来使用。

4、根据使用场景，ArrayList通常更适合随机查找，时间复杂度O(1)，add()方法直接在后面添加，如果不涉及扩容那么时间复杂度是O(1)，如果指定下标添加add(index,E)，那么时间复杂度为O(n)，后面的元素需要移动，LinkedList更适合添加、删除操作。这有一些细节，LinkedList可以通过getFirst()和getLast()直接获取第一个或最后一个元素，时间复杂度O(1)。如果get(index)指定下标元素，那需要依次遍历，时间复杂度O(n)，LinkedList的add()方法，直接在链表尾端添加元素，时间复杂度O(1)，如果指定下标add(index,E)添加元素，那么需要先遍历到指定下边位置，再添加元素，时间复杂度O(n)。



## 怎么创建线程安全的List

- 可以使用Vector，通过synchronized实现同步方法。

- 可以使用Collections的静态方法synchronizedList(List<> list)，通过synchronized同步代码块实现。

- ==todo 待完善==CopyOnWriteArrayList容器：读不加锁，写入时加锁，先copy一个新数组，写入完成后用新数组替换旧数组。



## HashMap jdk1.7与jdk1.8版本的区别

1、1.7版本的HashMap底层数据结构是==数组+链表==，1.8版本HashMap底层数据结构是==数组+链表+红黑树==，加入红黑树的目的是**提高HashMap的插入和查询效率**。

2、1.7中链表使用的是头插法，1.8中链表使用的是尾插法，因为1.8中需要遍历链表统计节点个数，因此正好使用尾插法，插入后链表数量如果大于等于8，链表将转换为红黑树，此时先判断数组长度是否达到64，如果没达到，就需要扩容，如果达到64，链表转红黑树。

3、1.7中哈希算法比较复杂，存在各种右移与异或运算，1.8中进行了简化，把**key的hash值右移16位**与**hash值异或运算**。因为复杂的哈希运算可以提高散列性，提高HashMap的整体效率，而1.8引入了红黑树，可以适当简化哈希算法，节省CPU资源

4、HashMap有两个参数影响其性能：**初始容量**和**加载因子**。

- 初始容量，指HashMap在创建时的容量，默认初始容量为16，每次扩容扩大为原来容量2倍。
- 加载因子，指HashMap在其容量自动扩容前可以达到的最大比例，默认加载因子为0.75。

创建HashMap时，jdk1.8对传入的加载因子直接使用。对指定的初始容量，通过**右移1位与位移前异运算**，得到的值再**右移2位，4位，8位，16位都做异运算**，得到的结果如果大于2^30，就设置容量为 2^30，如果小于 2^30，就设置容量为**结果+1**，这样得到的结果一定是**==比入参大的最小的 2^n==**，可以减少hash碰撞。

5、加载因子初始值设置为0.75，是时间和空间上的一种折衷。

- 加载因子越低，HashMap所能容纳键值对变少，扩容时，重新将键值对存储新的桶数组里，**键之间产生的哈希碰撞会降低**，此时HashMap的增删改查等操作效率变高，这是典型的**拿空间换时间**。
- 加载因子越高，HashMap所能容纳的键值对数量变多，**空间利用率高，哈希碰撞率也高**，这意味着链表变长，效率降低，这是拿**时间换空间**。

## 简述HashMap的put方法

hashMap的put方法的大体流程：

1. 计算key的hash值（把key的hashCode右移16位与原值**异或**计算），然后判断hashMap是否已经初始化，如果没有需要先初始化，然后**用hashMap数组长度 -1 与hash值**与运算，得到数组下标

2. 如果数组下标位置元素为空，则将key和value封装为Node对象(jdk1.7中是Entry对象)，放入该位置

3. 如果数组下标位置元素不为空，则要分情况讨论

   - a. 如果是jdk1.7，则先判断是否需要扩容，如果需要扩容就扩容，如果不需要扩容就生成Entry对象，并使用头插法添加到当前位置的链表中

   - b. 如果是jdk1.8，则先判断**当前位置上的Node类型**，看是红黑树Node还是，链表Node、
     - i. 如果是红黑树Node，则将key和value封装为一个红黑树节点，并添加到红黑树中去，在这个过程中会判断红黑树**是否存在当前key**，如果存在则更新value
     - ii. 如果此位置上Node是链表节点，将key和value封装为一个链表Node，并通过尾插法插入到链表最后位置去，因为是尾插法，需要遍历链表，在遍历链表过程中会判断是否存在当前key，如果存在则更新value。当遍历完链表后，将新链表Node插入到链表中，插入到链表后，会看当前链表节点个数，如果大于等于8，将该链表转成红黑树，此时先判断数组长度是否达到64，如果没达到，就需要扩容，如果达到64，链表转红黑树
     - iii. 将key和value封装为Node插入到链表或红黑树中后，**判断是否需要进行扩容**，如果需要就扩容

​			

## HashMap与HashTable区别

- HashMap 方法没有synchronized修饰，是非线程安全的；HashTable是线程安全的
- HashMap允许key为null，HashTable 不允许key为null
- HashMap中key为null的键值对，存储在数组的下标为0的位置



## LinkedHashMap实现原理

LinkedHashMap继承自HashMap，在HashMap基础上维护了一条双向链表，解决了HashMap不能随时保持遍历顺序和插入顺序一致的问题。

可以通过继承LinkedHashMap实现一个简单的LRU策略缓存。当我们基于LinkedHashMap实现缓存时，通过覆写其removeEldestEntry 实现自定义策略的LRU缓存。



## HashMap、LinkedHashMap和TreeMap简介

HashMap、LinkedHashMap和TreeMap 三个映射类基于不同数据结构实现了不同功能。

- HashMap，底层基于拉链式的散列结构，在jdk1.8中引入红黑树优化过长链表问题，提供了高效的增删改查操作
- LinkedHashMap，继承HashMap，通过维护一条双向链表，实现了散列数据结构的**有序遍历**。
- TreeMap，底层基于红黑树实现，利用红黑树特性，实现了键值对排序功能。



## ConcurrentHashMap原理，jdk1.7与jdk1.8区别

### jdk1.7

1、数据结构，ReentrantLock+Segment+HashEntry，一个Segment中包含一个HashEntry数组，每个HashEntry又是一个链表结构。

元素查询：二次hash，第一次hash定位到segment，第二次hash定位到元素所在的链表头部（即HashEntry数组对应下标位置）

2、锁，Segment分段锁，Segment继承了ReentrantLocal，锁定操作的Segement，其他的Segment不受影响，并发度为Segment个数，可以通过构造函数指定，数组扩容不会影响其他的Segment

get方法无需加锁，volatile保证可见性。

**Segment数量确定，当数据量很大时不能有更好的效率**

3、扩容，每个Segment内部会进行扩容，先生成新的数组，然后转移原数到新数组中，扩容判断也是每个Segment内部单独判断，判断是否超过阈值。

### jdk 1.8

1、数据结构：synchronized+CAS+Node+红黑树，Node的val和next都用volatile修饰，保证可见性。

查找，替换，赋值操作都使用CAS，**不支持key=null的键值对**。存入数据时，对key进行哈希算法运算得到数组下标位置，如果计算位置上是空的，通过CAS更新，更新失败或者位置上有元素，先在这个节点加synchronized锁，然后更新。

2、锁，锁链表的head节点，不影响其他元素的读写，锁粒度更细，效率更高，扩容时，阻塞所有读写操作、并发扩容

读操作无锁:Node的val和next使用volatile修饰，读写线程对该变量互相可见。数组volatile Node<K,V>[]用volatile修饰，保证扩容时被读线程感知。

3、扩容，支持多个线程同时扩容，扩容之前先生成一个新的数组，在转移元素时，先将原数组分组，将每组分给不同的线程来进行元素转移，每个线程负责一组或多组的元素转移工作。



## Queue和Deque(比较少用，简单整理)

- 1、Queue，单端队列，(FIFO)先进先出队列。Deque，Queue的子接口，双端队列，可以在首尾都进行插入删除操作

- 2、Queue的常用子类，PriorityQueue底层数据结构**数组，无边界，自带扩容机制**。Deque常用子类，LinkedList和ArrayDeque；LinkedList是**双向链表**。ArrayDeque是无初始容量的双端队列，**数组实现**
- 3、PriorityQueue可以作为**堆（优先队列）**使用，而且可以根据传入的Comparator实现大小的调整，会是一个很好的选择。
  - ArrayDeque可以作为栈或队列使用，但是栈的效率不如LinkedList高，**通常作为队列使用**。
  - LinkedList可以作为栈或队列使用，但是队列的效率不如ArrayQueue高，**通常作为栈(FILO)使用**。



# 线程




## 什么是守护线程?

守护线程就是为所有非守护线程(即用户线程)提供服务的线程，如GC垃圾回收线程。

当用户线程全都结束执行，守护线程也立即结束执行。因此，不应该用守护线程来做一些IO、File、数据库更新等操作。

可以调用isDaemon()方法检查线程是否为守护线程。

在Thread start之前，调用Thread.setDaemon(true)，可以将用户线程设置为守护线程。如果在thread start之后调用thread.setDaemon(true)，会抛出IllegalThreadStateException，不可以将正在运行的线程设置为守护线程。

守护线程中新产生的线程也是守护线程。

在java自带多线程框架，如ExecutorService中执行守护线程，会将守护线程设置为非守护线程。所以如果要使用守护线程，就不能用java的多线程框架。

![img](https://github.com/lission/markdownPics/blob/main/java/%E5%AE%88%E6%8A%A4%E7%BA%BF%E7%A8%8B.png?raw=true)


## ThreadLocal

### 1、什么是ThreadLocal

ThreadLocal是**java线程本地存储工具**，实现线程间的数据隔离，缓存数据至线程内部，线程可以在任意时刻、任意方法获取到缓存的数据。

### 2、ThreadLocal底层原理

ThreadLocal底层通过ThreadLocalMap实现，每个Thread内部有一个ThreadLocalMap成员变量，ThreadLocalMap数据结构是Entry数组。Entry继承了WeakReference<ThreadLocal<?>>，由一个ThreadLocal对象和一个Object对象构成，Entry的key为ThreadLocal对象，并且是一个弱引用，value为需要缓存的对象。

####  存储数据

调用ThreadLocal的set()方法时，ThreadLocal首先获取当前Thread对象，然后通过Thread对象获取该线程的ThreadLocalMap，以当前ThreadLocal为key，存入value数据至ThreadLocalMap中。

#### 获取数据
调用ThreadLocal的get()方法时，ThreadLocal获取当前Thread对象，然后通过Thread对象获取该线程的ThreadLocalMap，再以当前ThreadLocal为key，获取value数据。

### 3、ThreadLocal 应用场景

通过ThreadLocal可以实现Thread之间的数据隔离，也可以使线程在任意时刻、任意方法获取缓存的数据

- 利用ThreadLocal的在线程内部缓存数据，可以在任意时刻，任意方法获取缓存数据的特性，可以作为**跨层传递参数**
- 利用ThreadLocal实现线程间的数据隔离特性，可以实现**事务隔离，session会话管理**等

### 4、ThreadLocal内存泄露风险

两种场景下会发生内存泄漏

![img](https://raw.githubusercontent.com/lission/markdownPics/main/java/threadLocal-entry.webp)

- 一个原因是ThreadLocal的内部类ThreadLocalMap中的Entry设计，Entry继承了WeakReference<ThreadLocal<?>>，Entry key为ThreadLocal，是弱引用，如果没有对key的强引用，key会被垃圾回收，这会导致一种现象，key为null，value有值。key为空的话，value是无效数据，随着无效value数据累积，会造成内存泄露。
- 线程池中使用ThreadLocal，也可能会造成内存泄露。线程池中的线程是不会回收的，线程对ThreadLocalMap是强引用，ThreadLocalMap对Entry也是强引用，线程不回收，Entry对象也不会被回收，从而出现内存泄露。

### 5、如何避免内存泄露

使用ThreadLocal后，手动调用ThreadLocal的remove()方法，该方法会清除当前key和value。

![img](https://raw.githubusercontent.com/lission/markdownPics/main/java/threadLocal-remove.webp)

其中的e.clear()方法会清除Entry的Key，它调用的是WeakReference的方法， this.referent = null。expungStateEntry(i)方法会清除Entry对应的value，同时会遍历下一个key == null的Entry，并将value赋值为null，等待GC回收。

### 6、JDK开发者如何处置内存泄露

ThreadLocal的设计者意识到了内存泄露的风险，他们在一些方法中埋了对key==null的Entry擦除的操作。

以get()方法为例，它调用了ThreadLocalMap的getEntry()方法，对key进行了校验，及对null key进行擦除。

同理set()方法也最终调用了expungStateEntry(i)方法。

这样做只能尽可能避免内存泄露，但不能完全避免内存泄露问题，极端情况下，只创建了ThreadLocal，但没有调用set、get、remove方法，因此应在使用完ThreadLocal后，手动调用remove方法。

### 7、弱引用导致内存泄露，那为什么Entry中key不设置为强引用

如果将key设置为强引用，threadLocal实例释放后，threadLocal == null，但是threadLocal对ThreadLocalMap存在强引用，ThreadLocalMap中Entry对threadLocal也存在强引用，形成引用循环，这会导致threadLocal不被GC回收。

弱引用虽然可能导致内存泄露，但是通过set、get、remove等方法中主动对key==null的Entry擦除，方案更优。

### 拓展

#### 1、spring如何保证数据库事务在同一个连接下执行的

 **todo 待完善**

ThreadLocal可以实现线程间的数据隔离，对spring的默认单例bean多线程访问是一个完美的解决方案。

要想实现jdbc事务，必须在同一个连接对象中操作，否则会事务不可控。

DataSourceTransactionManager是spring中的数据源事务管理器，它会在你调用getConnection()时从数据库连接池获取一个connection，让后将其与ThreadLocal绑定，事务结束后解绑，这样保证了事务在同一连接下执行。





## 并发三大特性：原子性、可见性、有序性



## 为什么使用线程池

- **降低资源消耗**。通过重复利用已创建的线程降低现场创建和销毁的消耗
- **提高响应速度**。当任务到达时，任务可以不需要等待线程创建立即执行
- **提高线程的可管理行**。线程是稀缺资源，如果无限制创建，不仅消耗系统资源，还会降低系统稳定性，使用线程池可以进行统一分配、调优和监控。

## 如何创建线程池

- 通过ThreadPoolExecutor构造方法实现
- 通过Executor框架的工具类Executors实现

《阿里巴巴java开发手册》强制线程池不允许使用Executors去创建，而是使用ThreadPoolExecutor的方式，规避资源耗尽的风险

Executors返回线程池对象的弊端：

- FixedThreadPool 和 SingleThreadExecutor：允许请求的队列长度为Integer.MAX_VALUE，可能堆积大量请求，导致OOM。
- CachedThreadPool 和 ScheduledThreadPool：允许创建的线程数量为Integer.MAX_VALUE，可能创建大量线程，导致OOM。

## ThreadPoolExcutor原理

线程池处理流程executor.execute(worker)来提交一个任务到线程池中去

![threadPoolExecutor说明图](https://github.com/lission/markdownPics/blob/main/java/%E5%9B%BE%E8%A7%A3%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86.png?raw=true)

### ThreadPoolExecutor构造函数重要参数

ThreadPoolExecutor最长构造函数共有**7**个参数

- corePoolSize，核心线程数，正常状况下运行的线程数量，这些线程创建后不会消除，是一种常驻线程
- maximumPoolSize，当队列中存放的任务达到容量时，可以运行的最大线程数
- keepAliveTime，超出核心线程数的空闲线程存活时间，超出核心线程数的线程在空闲超过一定时间会被消除。
- unit(TimeUnit)，keepAliveTime的单位
- workQueue，用来存放待执行的任务，假设当前核心线程都已被占用，新进来的任务被放入队列，直到队列被放满，还有任务进入，才会创建新的线程。
- threadFactor，线程工厂，用来生产线程执行任务。可以使用默认的线程工厂DefaultThreadFactory，产生的线程都在同一个组内，同样的优先级，都是非守护线程。也可以自定义线程工厂。
- handler，任务拒绝策略，有两种场景：
  - 调用shutdown()等方法关闭线程池后，此时即使线程内部还有没执行完的任务，由于线程池已关闭，我们想继续提交任务会被拒绝
  - 当达到最大线程数，线程池没有能力处理新提交的任务时，夜会执行拒绝策略

### ThreadPoolExecutor拒绝策略

四种拒绝策略

- **ThreadPoolExecutor.AbortPolicy**，抛出RejectedExecutionException，拒绝新任务

- **ThreadPoolExecutor.CallerRunsPolicy**，由**创建了线程池的线程**执行被拒绝的任务，该策略会降低对新任务提交速度，影响程序整体性能
- **ThreadPoolExecutor.DiscardPolicy**，不处理新任务，直接丢弃掉
- **ThreadPoolExecutor.DiscardOldestPolicy**，将最早的未处理的任务丢弃掉



## 线程池中阻塞队列的作用？为什么是先添加队列而不是先创建最大线程？

1、一般的队列只能作为一个有限长度的缓冲区，如果超过缓冲长度，无法保留当前任务。阻塞队列可以通过阻塞保留当前想要继续入队的任务。

阻塞队列可以在任务队列中没有任务时阻塞获取任务的线程，使线程进入wait状态，释放cpu资源。

阻塞队列自带阻塞和唤醒功能，不需要额外处理，无任务执行时，线程池利用阻塞队列的take方法挂起，维持核心线程的存活，不至于一直占用cpu资源。

2、在创建新线程的时候，是要获取全局锁的，这时其他线程就要阻塞，影响整体效率

## 线程池中线程复用的原理

线程池将线程和任务进行解耦，线程是线程，任务是任务，摆脱了之前通过Thread创建线程时的一个线程必须对应一个任务的限制。

在线程池中，同一个线程可以不断从阻塞队列获取任务执行，其核心原理在于，线程池对Thread进行了封装，并不会每次执行任务都调用Thread.start()创建新线程，而是让每个线程去执行一个”循环任务“，在这个循环任务中不停检查是否有任务需要被执行，如果有则执行，调用任务中的run()方法，将run方法当做一个普通方法执行，通过这种方式使用固定线程将所有任务run方法串联起来。



## Thread的生命周期

Thread start之后处于runnable状态，等待获取cpu使用权

阻塞状态是线程由于某种原因放弃cpu使用权，暂时停止运行。直到线程进入就绪runnable状态，才有机会转到运行状态

![img](https://github.com/lission/markdownPics/blob/main/java/thread_lifecycle.png?raw=true)

## 描述ForkJoinPool

ForkJoinPool是jdk7增加的一个线程池类，Fork/Join是分治算法的一个并行实现。它可以将一个大任务拆分为很多小任务来异步执行。

Fork/Join框架主要三个模块：

- 任务对象：ForkJoinTask，包括（RecursiveTask、RecursiveAction、CountedCompleter）
- 执行fork/join任务的线程：ForkJoinWorkerThread
- 线程池：ForkJoinPool

三者关系是，ForkJoinPool可以通过池中的ForkJoinWorkerThread来处理ForkJoinTask任务

### ForkJoinPool核心思想

- 分治算法（Divide and Conquer），把任务递归的拆分为子任务

- 工作窃取算法（work-stealing），除了线程池内部有队列，每个ForkJoinWorkerThread线程内部都有队列，线程执行时把任务拆分成小任务写入线程内部队列，当线程执行完自己的任务，会窃取其他线程的任务执行。

  

  工作窃取详细思路暂不作为本次重点，[参考地址](https://www.pdai.tech/md/java/thread/java-thread-x-juc-executor-ForkJoinPool.html#%E5%B8%A6%E7%9D%80bat%E5%A4%A7%E5%8E%82%E7%9A%84%E9%9D%A2%E8%AF%95%E9%97%AE%E9%A2%98%E5%8E%BB%E7%90%86%E8%A7%A3forkjoin%E6%A1%86%E6%9E%B6)

- 

  - 每个线程都有自己的一个WorkQueue，该工作队列是一个双端队列。
  - 队列支持三个功能push、pop、poll
  - push/pop只能被队列的所有者线程调用，而poll可以被其他线程调用。
  - 划分的子任务调用fork时，都会被push到自己的队列中。
  - 默认情况下，工作线程从自己的双端队列获出任务并执行。
  - 当自己的队列为空时，线程随机从另一个线程的队列末尾调用poll方法窃取任务。

### 哪些JDK源码中利用了Fork/Join思想

- 数组工具类 Arrays 在JDK 8之后新增的并行排序方法(parallelSort)就运用了 ForkJoinPool 的特性
- ConcurrentHashMap 在JDK 8之后添加的函数式方法(如forEach等)也有运用

### 实际应用

- 可以用来实现斐波那契数列



# 锁

## 死锁

~~两个或多个线程互相争抢资源，每个线程获取了其他线程的锁，自己需要的锁又被占用，导致都无法进行下去。~~

导致死锁的原因：

- 1、一个资源一次只能被一个资源使用
- 2、一个线程在阻塞等待某个资源时，不释放已占用资源
- 3、一个线程已获得的资源，在未使用完之前，不能被强制剥夺
- 4、若干线程形成头尾相接的循环等待资源关系

死锁必须要满足上述4个条件

开发过程中应注意：

- 1、注意加锁顺序，保证每个线程按同样顺序进行加锁
- 2、注意加锁时限，可以针对锁设置一个超时时间
- 3、注意死锁检查，这是一种预防机制，确保第一时间发现死锁并解决

## 如何查看线程死锁

1、可以通过jstack命令来进行查看，jstack命令中会显示发生了死锁的线程

Jstack [option] pid

| 选项 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| -F   | 当正常输出的请求不被响应时，强制输出线程堆栈                 |
| -m   | 如果调用到本地方法的话，可以显示C/C++的堆栈                  |
| -l   | 除堆栈外，显示关于锁的附加信息，在发生死锁时可以用jstack -l pid来观察锁持有情况 |



## synchronized、lock、cas的实现与区别，各自优缺点，描述volatile关键字

- synchronized，锁关键字，通过jvm实现，修饰的代码块是原子性的，依赖对象头的线程id和monitor锁，也可以禁止指令重排，非公平锁

- lock，通过jdk实现的锁，更加灵活，支持公平锁与非公平锁，内部通过双端队列AQS实现

- cas，轻量级锁，内部操作是原子操作，每一次都要判断拿到的值与当前值是否一致，如果一致则更新数据，可能会出现ABA问题

- volatile是修饰变量的，主要保证每次更新对所有线程可见，通过内存屏障防止指令重排序。使用happen-before描述两个操作之间的内存可见性。volatile实现：在volatile写操作的后面插入一个StoreLoad屏障。保证volatile写操作不会和之后的读操作重排序。

  如果A happen-before B，意味着A的执行结果必须对B可见，也就是保证跨线程的内存可见性。A happen before B不代表A一定在B之前执行。因为，对于多线程程序而言，两个操作的执行顺序是不确
  定的。happen-before只确保如果A在B之前执行，则A的执行结果必须对B可见。定义了内存可见性的约束，也就定义了一系列重排序的约束。happen-before具有传递性。



## volatile详解

作用：防重排序、实现可见性、保证原子性(单次读/写)

可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是**每个线程拥有自己的一个高速缓存区——线程工作内存**

volatile不能保证完全的原子性，只能保证单次的读/写操作具有原子性

### 问题1： i++为什么不能保证原子性?

i++其实是一个复合操作，包括三步骤：

- 读取i的值。
- 对i加1。
- 将i的值写回内存。

### 问题2： 共享的long和double变量的为什么要用volatile?

**还是不太明白**

因为long和double两种数据类型的操作可分为高32位和低32位两部分，因此普通的long或double类型读/写可能不是原子的，因此，鼓励大家将共享的long和double变量设置为volatile类型，这样能保证任何情况下对long和double的单次读/写操作都具有原子性。



# 零碎

## String使用final修饰的好处，介绍一下字符串常量池

String类型被final修饰，同时内部的字符数组也被private修饰，保证字符串是不可变的。

不可变的好处：

- 可以缓存hash值，String作为HashMap的key，不可变特性使得hash值也不可变，只需要进行一次计算
- 字符串常量池的需要，如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool
- 安全性，String作为参数可以保证参数不可变
- 线程安全，天然的线程安全

字符串常量池：多个引用(String的对象)指向相同的字符串时，可以不用创建多个，因为不可修改。如果某个对象修改了会指向别的字符串，不会导致一个修改别的引用的值也都变了。



## jdk动态代理和cglib的实现与区别

动态代理就是在程序运行期，创建目标对象的代理对象，并对目标对象中的方法进行功能性增强。

- jdk动态代理：对接口的代理，要实现InvocationHandler接口，并实现invoker()方法，接口与实现类和静态代理一样，创建代理对象需要过Proxy.newProxyInstance()方法，代理对象直接调用接口的方法就可以执行invoker()代码（对被代理方法的增强）
- CGlib动态代理：以子类的形式进行代理，不用非得实现接口。基于ASM字节码工具操作字节码。



## 描述jdk的反射机制

**todo 待完善**

运行时类型识别，通过类的全限定名，在运行时构造一个类的对象；判断一个类所具有的的成员变量和方法；调用一个对象的方法；生成动态代理。

反射最大的应用就是框架。

反射的缺点：

- 由于反射涉及动态解析的类型，因此无法执行某些JVM优化。因此，反射操作的性能要比非反射慢。
- 反射机制可以使private修饰的方法、属性在别的类中也可以使用(不安全)。



## java中常见异常及用法

运行时异常、编译异常

（1）ClassCastException，类型转换异常类
（2）NullPointException，空指针异常
（3）NoSuchMethodException，方法未找到抛出的异常
（4）IndexOutOfBoundsException下标越界
（5）IOException，操作输入流和输出流时可能出现的异常
（6）NumberFormatException，字符串转换为数字抛出的异常
（7）FileNotFoundException，文件未找到异常



## java怎么实现的序列化

序列化需要实现序列化接口Serializable，序列化接口没有方法，只是序列化的标识，实现序列化接口的类可以被序列化。

序列化是把对象转为字节序列的过程。如果不指定serialVersionUID，对象在序列化时会自动生成一个serialVersionUID，反序列化时可能会导致InvalidClassException，反序列化对class细节很敏感，serialVersionUID的值要一致。



## 集合类为什么没有实现克隆和序列化接口

克隆(cloning)或者是序列化(serialization)的语义和含义是跟具体的实现相关的。

集合类框架中的接口没有实现Cloneable和Serializable接口，但是集合具体的类是有实现这两个接口的，比如ArrayList。因为接口不是具体的容器，所以不需要实现这两个接口，也没有意义。
