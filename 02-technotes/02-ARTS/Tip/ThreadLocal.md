[TOC]

# 基础

## 1、什么是ThreadLocal

ThreadLocal是java线程本地存储工具，实现线程间的数据隔离，缓存数据至线程内部，线程可以在任意时刻、任意方法获取到缓存的数据。

## 2、ThreadLocal底层原理

ThreadLocal底层通过ThreadLocalMap实现，每个Thread内部有一个ThreadLocalMap成员变量，ThreadLocalMap由Entry数组组成。Entry继承了WeakReference<ThreadLocal<?>>，由一个ThreadLocal对象和一个Object对象构成，Entry的key为ThreadLocal对象，并且是一个弱引用，value为需要缓存的对象。

### 存储数据

调用ThreadLocal的set()方法时，ThreadLocal首先获取当前Thread对象，然后通过Thread对象获取该线程的ThreadLocalMap，以当前ThreadLocal为key，存入value数据至ThreadLocalMap中。

### 获取数据

调用ThreadLocal的get()方法时，ThreadLocal获取当前Thread对象，然后通过Thread对象获取该线程的ThreadLocalMap，再以当前ThreadLocal为key，获取value数据。

## 3、ThreadLocal 应用场景

通过ThreadLocal可以实现Thread之间的数据隔离，也可以使线程在任意时刻、任意方法获取缓存的数据

- 利用ThreadLocal的在线程内部缓存数据，可以在任意时刻，任意方法获取缓存数据的特性，可以作为跨层传递参数
- 利用ThreadLocal实现线程间的数据隔离特性，可以实现事务隔离，session会话管理等

## 4、ThreadLocal内存泄露风险

两种场景下会发生内存泄漏

![img](https://raw.githubusercontent.com/lission/markdownPics/main/java/threadLocal-entry.webp)

- 一个原因是ThreadLocal的内部类ThreadLocalMap中的Entry设计，Entry继承了WeakReference<ThreadLocal<?>>，Entry key为ThreadLocal，是弱引用，如果没有对key的强引用，key会被垃圾回收，这会导致一种现象，key为null，value有值。key为空的话，value是无效数据，随着无效value数据累积，会造成内存泄露。
- 线程池中使用ThreadLocal，也可能会造成内存泄露。线程池中的线程是不会回收的，线程对ThreadLocalMap是强引用，ThreadLocalMap对Entry也是强引用，线程不回收，Entry对象也不会被回收，从而出现内存泄露。

## 5、如何避免内存泄露

使用ThreadLocal后，手动调用ThreadLocal的remove()方法，该方法会清除当前key和value。

![img](https://raw.githubusercontent.com/lission/markdownPics/main/java/threadLocal-remove.webp)

其中的e.clear()方法会清除Entry的Key，它调用的是WeakReference的方法， this.referent = null。expungStateEntry(i)方法会清除Entry对应的value，同时会遍历下一个key == null的Entry，并将value赋值为null，等待GC回收。



## 6、JDK开发者如何处置内存泄露

ThreadLocal的设计者意识到了内存泄露的风险，他们在一些方法中埋了对key==null的Entry擦除的操作。

以get()方法为例，它调用了ThreadLocalMap的getEntry()方法，对key进行了校验，及对null key进行擦除。

同理set()方法也最终调用了expungStateEntry(i)方法。

这样做只能尽可能避免内存泄露，但不能完全避免内存泄露问题，极端情况下，只创建了ThreadLocal，但没有调用set、get、remove方法，因此应在使用完ThreadLocal后，手动调用remove方法。



## 7、弱引用导致内存泄露，那为什么Entry中key不设置为强引用

如果将key设置为强引用，threadLocal实例释放后，threadLocal == null，但是threadLocal对ThreadLocalMap存在强引用，ThreadLocalMap中Entry对threadLocal也存在强引用，形成引用循环，这会导致threadLocal不被GC回收。

弱引用虽然可能导致内存泄露，但是通过set、get、remove等方法中主动对key==null的Entry擦除，方案更优。



# 拓展

## 1、spring如何保证数据库事务在同一个连接下执行的

 ==todo 待完善==

ThreadLocal可以实现线程间的数据隔离，对spring的默认单例bean多线程访问是一个完美的解决方案。

要想实现jdbc事务，必须在同一个连接对象中操作，否则会事务不可控。

DataSourceTransactionManager是spring中的数据源事务管理器，它会在你调用getConnection()时从数据库连接池获取一个connection，让后将其与ThreadLocal绑定，事务结束后解绑，这样保证了事务在同一连接下执行。