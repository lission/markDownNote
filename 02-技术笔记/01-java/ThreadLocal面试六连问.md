[TOC]

说一下ThreadLocal？

ThreadLocal是java线程本地存储工具，可以实现线程间的数据隔离，将数据缓存在线程内部，在该线程的任意时刻、任意方法中都可以获取缓存的数据

ThreadLocal底层通过ThreadLocalMap实现，每一个线程Thread都存在一个ThreadLocalMap成员变量，ThreadLocalMap由Entry数组构成，Entry继承自WeakReference<ThreadLocal<?>>，一个Entry由ThreadLocal对象和Object对象构成，Entry的key为ThreadLocal对象,并且是一个弱引用，value为需要缓存的对象。

线程池中使用ThreadLocalMap会造成内存泄露，因为线程池中线程不会回收，线程对象通过强引用指向ThreadLocalMap，ThreadLocalMap也是强引用指向Entry对象，线程不回收，Entry对象也不回收，从而出现内存泄露，为避免内存泄露需要手动调用ThreadLocal的remove方法，手动清除Entry对象

典型应用:连接管理



ThreadLocal为Java并发提供了一个新的思路， 它用来**存储Thread的局部变量**， 从而达到**各个Thread之间的隔离运行**。它被广泛应用于框架之间的**用户资源隔离、事务隔离等**。==**用不好会导致内存泄露**==

# 1、内存泄露原因探索

ThreadLocal操作不当会引发内存泄露，主要原因在于它的内部类ThreadLocalMap中的Entry的设计。

Entry继承了WeakReference<ThreadLocal<?>>，即Entry的key是ThreadLocal对象是弱引用，所以当没有指向key的强引用后，**key会在垃圾回收的时候被回收掉，而key对应的value则不会被回收，这会导致一种现象：key为null，value有值。**key为空的话value是无效数据，久而久之，value累加会导致内存泄露。

![img](https://raw.githubusercontent.com/lission/markdownPics/main/java/threadLocal-entry.webp)

# 2、如何解决内存泄露问题

每次使用完ThreadLocal都调用它的remove()方法清除数据。因为它的remove()方法会主动将当前的key和value(Entry)进行清除。

![img](https://raw.githubusercontent.com/lission/markdownPics/main/java/threadLocal-remove.webp)

e.clear()用于清除Entry的key，它调用的是WeakReference中的方法：this.referent = null

expungeStateEntry(i)用于清除Entry对应的value

# 3、JDK开发者是如何避免内存泄露的

ThreadLocal的设计者意识到了内存泄露的风险，他们在一些方法中埋了对key=null的value擦除操作。

以ThreadLocal提供的get()方法举例，它调用了ThreadLocalMap#getEntry()方法，对key进行了校验和对null key进行擦除。

![img](https://raw.githubusercontent.com/lission/markdownPics/main/java/threadlocal-key-remove.webp)

如果key为null，则会调用getEntryAfterMiss()方法，在这个方法找那个，如果k==null，则调用expungeStaleEntry(i)方法。

expungeStaleEntry(i)方法完成了对key=null的key所对应的value进行赋空，释放了空间避免内存泄露。同时它遍历下一个key为空的entry，并将value赋值为null，等待下次GC释放其空间。

![img](https://raw.githubusercontent.com/lission/markdownPics/main/java/threadlocal-remove-value.webp)

同理，set()方法最终也是调用该方法expungeStaleEntry(i)，调用路径：

set(T value)->

map.set(this, value)->

rehash()->

expungeStaleEntries()

remove方法

remove()->

ThreadLocalMap.remove(this)->

expungeStaleEntry(i)

这样做只能是尽可能避免内存泄露，但并不会完全解决内存泄露这个问题。比如极端情况，我们只创建ThreadLocal但不调用set、get、remove方法等。所以最能解决问题的办法就是用完ThreadLocal后手动到右remove()。

# 4、手动释放ThreadLocal遗留存储？如何设计/实现？

包装其父类remove方法为静态方法，如果是spring项目，可以借助于bean的生命周期，在拦截器的aftrCompletion阶段进行调用。

==**弱引用导致内存泄露，那为什么key不设置为强引用**==

如果key设置为强引用，当threadLocal实例释放后，threadLocal=null，但是threadLocal会有强引用指向threadLocalMap，threadLocalMap.Entry又强引用ThreadLocal，这样会导致threadLocal不能正常被GC回收。

弱引用虽然会引起内存泄露，但是也有set、get、remove方法操作对nullkey进行擦除的补救措施，方案更优。

==**线程执行结束后会不会自动清空Entry的value**==

当currentThread执行结束后，threadLocalMap变得不可达而被回收，Entry等也就都被回收了，但这个环境要求不对Thread进行复原，但是我们==**项目中经常会复用线程来提高性能**==，所以currentThread一般不会处于终止状态。

# 5、Thread和ThreadLocal有什么联系？

Thread和ThreadLocal是绑定的，ThreadLocal依赖于Thread去执行，Thread将需要隔离的数据存放到ThreadLocal(准确讲是ThreadLocalMap)中，实现多线程处理的数据隔离。

# 6、Spring如何处理Bean多线程下的并发问题？

ThreadLocal天生为解决相同变量的访问冲突问题，对spring的默认单例bean的多线程访问是一个完美的解决方案。spring使用了ThreadLocal来处理多线程下相同变量并发的线程安全问题。

==**spring如何保证数据库事务在同一个连接下执行的**==

要想实现jdbc事务，必须在同一个连接对象中操作，多个连接下事务就会不可控，需要借助分布式事务完成。spring是如何保证数据库事务在同一个连接下执行的呢？

DataSourceTransactionManager是spring的数据源事务管理器，它会在你调用getConnection()的时候从数据库连接池中获取一个connection，然后**将其与ThreadLocal绑定，事务完成后解除绑定，这样保证了事务在同一连接下完成**

==**概要源码**==：

- 1、事务开始阶段：

org.springframework.jdbc.datasource.DataSourceTransactionManager#doBegin->TransactionSynchronizationManager#bindResource->org.springframework.transaction.support.TransactionSynchronizationManager#bindResource

![img](https://raw.githubusercontent.com/lission/markdownPics/main/java/transaction-start.webp)

- 2、事务结束阶段：

org.springframework.jdbc.datasource.DataSourceTransactionManager#doCleanupAfterCompletion->TransactionSynchronizationManager#unbindResource->org.springframework.transaction.support.TransactionSynchronizationManager#unbindResource->TransactionSynchronizationManager#doUnbindResource

![img](https://raw.githubusercontent.com/lission/markdownPics/main/java/transaction-end.webp)