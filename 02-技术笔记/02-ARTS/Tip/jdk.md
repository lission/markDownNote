[TOC]

# 集合
## ArrayList与LinkedList的区别

1、底层数据结构不同，ArrayList底层是数组，LinkedList是双向链表

2、ArrayList容量有限制，可以指定容量，默认容量是10，当超过容量时需要扩容，每次扩容50%。LinkedList没有容量限制

3、ArrayList和LinkedList都实现了List接口，LinkedList还实现了Deque接口，可以当做双端队列来使用。

4、根据使用场景，ArrayList通常更适合随机查找，时间复杂度O(1)，add()方法直接在后面添加，如果不涉及扩容那么时间复杂度是O(1)，如果指定下标添加add(index,E)，那么时间复杂度为O(n)，后面的元素需要移动，LinkedList更适合添加、删除操作。这有一些细节，LinkedList可以通过getFirst()和getLast()直接获取第一个或最后一个元素，时间复杂度,O(1)。如果get(index)指定下标元素，那需要依次遍历，时间复杂度O(n)，LinkedList的add()方法，直接在链表尾端添加元素O(1)，如果指定下标add(i,E)添加元素，那么需要先遍历到指定下边位置，再添加元素O(n)。



## 怎么创建线程安全的List

- 可以使用Vector，通过synchronized实现同步方法。

- 可以使用Collections的静态方法synchronizedList(List<> list)，通过synchronized同步代码块实现。

- ==todo 待完善==CopyOnWriteArrayList容器：读不加锁，写入时加锁，先copy一个新数组，写入完成后用新数组替换旧数组。



## HashMap jdk1.7与jdk1.8版本的区别

1、1.7版本的HashMap底层数据结构是==数组+链表==，1.8版本HashMap底层数据结构是==数组+链表+红黑树==，加入红黑树的目的是提高HashMap的插入和查询效率。

2、1.7中链表使用的是头插法，1.8中链表使用的是尾插法，因为1.8中需要遍历链表统计链表个数，因此正好使用尾插法，插入后链表数量如果大于等于8，链表将转换为红黑树，此时先判断数组长度是否达到64，如果没达到，就需要扩容，如果达到64，链表转红黑树。

3、1.7中哈希算法比较复杂，存在各种右移与异或运算，1.8中进行了简化，把key的hash值右移16位与hash值异或运算。因为复杂的哈希运算可以提高散列性，提高HashMap的整体效率，而1.8引入了红黑树，可以适当简化哈希算法，节省CPU资源

4、HashMap有两个参数影响其性能：**初始容量**和**加载因子**。

- 初始容量，指HashMap在创建时的容量，默认初始容量为16，每次扩容扩大为原来容量2倍。
- 加载因子，指HashMap在其容量自动扩容前可以达到的最大比例，默认加载因子为0.75。

创建HashMap时，jdk1.8对传入的加载因子直接使用。对指定的初始容量，通过右移1位与位移前异运算，得到的值再右移2位，4位，8位，16位都做异运算，得到的结果如果大于2^30，就设置容量为 2^30，如果小于 2^30，就设置容量为结果+1，这样得到的结果一定是==比入参大的最小的 2^n==，可以减少hash碰撞。

5、加载因子初始值设置为0.75，是时间和空间上的一种折衷。

- 加载因子越低，HashMap所能容纳键值对变少，扩容时，重新将键值对存储新的桶数组里，键之间产生的哈希碰撞会降低，此时HashMap的增删改查等操作效率变高，这是典型的拿空间换时间。
- 加载因子越高，HashMap所能容纳的键值对数量变多，空间利用率高，哈希碰撞率也高，这意味着链表变长，效率降低，这是拿时间换空间。

## 简述HashMap的put方法

hashMap的put方法的大体流程：

1. 计算key的hash值（把key的hashCode右移16位与原值**异或**计算），然后判断hashMap是否已经初始化，如果没有需要先初始化，然后用hashMap数组长度 -1 与hash值**与**运算，得到数组下标

2. 如果数组下标位置元素为空，则将key和value封装为Node对象(jdk1.7中是Entry对象)，放入该位置

3. 如果数组下标位置元素不为空，则要分情况讨论

   - a. 如果是jdk1.7，则先判断是否需要扩容，如果需要扩容就扩容，如果不需要扩容就生成Entry对象，并使用头插法添加到当前位置的链表中

   - b. 如果是jdk1.8，则先判断当前位置上的Node类型，看是红黑树Node还是，链表Node、
     - i. 如果是红黑树Node，则将key和value封装为一个红黑树节点，并添加到红黑树中去，在这个过程中会判断红黑树是否存在当前key，如果存在则更新value
     - ii. 如果此位置上Node是链表节点，将key和value封装为一个链表Node，并通过尾插法插入到链表最后位置去，因为是尾插法，需要遍历链表，在遍历链表过程中会判断是否存在当前key，如果存在则更新value。当遍历完链表后，将新链表Node插入到链表中，插入到链表后，会看当前链表节点个数，如果大于等于8，将该链表转成红黑树，此时先判断数组长度是否达到64，如果没达到，就需要扩容，如果达到64，链表转红黑树
     - iii. 将key和value封装为Node插入到链表或红黑树中后，判断是否需要进行扩容，如果需要就扩容

​			

## HashMap与HashTable区别

- HashMap 方法没有synchronized修饰，是非线程安全的；HashTable是线程安全的
- HashMap允许key为null，HashTable 不允许key为null
- HashMap中key为null的键值对，存储在数组的下标为0的位置



## LinkedHashMap实现原理

LinkedHashMap继承自HashMap，在HashMap基础上维护了一条双向链表，解决了HashMap不能随时保持遍历顺序和插入顺序一致的问题。

可以通过继承LinkedHashMap实现一个简单的LRU策略缓存。当我们基于LinkedHashMap实现缓存时，通过覆写removeEldestEntry 实现自定义策略的LRU缓存。



## HashMap、LinkedHashMap和TreeMap简介

HashMap、LinkedHashMap和TreeMap 三个映射类基于不同数据结构实现了不同功能。

- HashMap，底层基于拉链式的散列结构，在jdk1.8中引入红黑树优化过长链表问题，提供了高效的增删改查操作
- LinkedHashMap，继承HashMap，通过维护一条双向链表，实现了散列数据结构的有序遍历。
- TreeMap，底层基于红黑树实现，利用红黑树特性，实现了键值对排序功能。





## ConcurrentHashMap原理，jdk1.7与jdk1.8区别

### jdk1.7

1、数据结构，ReentrantLock+Segment+HashEntry，一个Segment中包含一个HashEntry数组，每个HashEntry又是一个链表结构。

元素查询：二次hash，第一次hash定位到segment，第二次hash定位到元素所在的链表头部（即HashEntry数组对应下标位置）

2、锁，Segment分段锁，Segment继承了ReentrantLocal，锁定操作的Segement，其他的Segment不受影响，并发度为Segment个数，可以通过构造函数指定，数组扩容不会影响其他的Segment

get方法无需加锁，volatile保证可见性。

**Segment数量确定，当数据量很大时不能有更好的效率**

### jdk 1.8

1、数据结构：synchronized+CAS+Node+红黑树，Node的val和next都用volatile修饰，保证可见性。

查找，替换，赋值操作都使用CAS，不支持key=null的键值对。存入数据时，对key进行hash运算得到数组下表位置，如果计算位置上是空的，通过cas更新，更新失败或者位置上有元素，先在这个节点加synchronized锁，然后更新。

2、锁，锁链表的head节点，不影响其他元素的读写，锁粒度更细，效率更高，扩容时，阻塞所有读写操作、并发扩容

读操作无锁:Node的val和next使用volatile修饰，读写线程对该变量互相可见。数组volatile Node<K,V>[]用volatile修饰，保证扩容时被读线程感知。



# 线程




## 什么是守护线程?

守护线程就是为所有非守护线程(即用户线程)提供服务的线程，如GC垃圾回收线程。

当用户线程全都结束执行，守护线程也立即结束执行。因此，不应该用守护线程来做一些IO、File、数据库更新等操作。

可以调用isDaemon()方法检查线程是否为守护线程。

在Thread start之前，调用Thread.setDaemon(true)，可以将用户线程设置为守护线程。如果在thread start之后调用thread.setDaemon(true)，会抛出IllegalThreadStateException，不可以将正在运行的线程设置为守护线程。

守护线程中新产生的线程也是守护线程。

在java自带多线程框架，如ExecutorService中执行守护线程，会将守护线程设置为非守护线程。所以如果要使用守护线程，就不能用java的多线程框架。

![img](https://github.com/lission/markdownPics/blob/main/java/%E5%AE%88%E6%8A%A4%E7%BA%BF%E7%A8%8B.png?raw=true)