[TOC]

# JVM内存结构

![img](https://github.com/lission/markdownPics/blob/main/java/jvm%20data%20areas.jpeg?raw=true)

JVM运行时数据区

- 线程私有：程序计数器、虚拟机栈、本地方法区
- 线程共享：堆、方法区(Java7的永久代或JDK8的元空间)、堆外内存(jdk8的元空间)

## 程序计数器

PC寄存器存储指向下一条指令的地址，即 将要执行的指令代码。

**为何PC寄存器设定为线程私有的？**

多线程在一个特定时间只会执行一个线程，CPU会不停切换任务，这样会导致线程中断或恢复。为了能够准确记录每个线程正在执行的当前字节码指令地址，所以为每个线程分配一个PC寄存器，每个线程独立计算，互不影响。

## 虚拟机栈

主管java程序的运行，保存方法的局部变量、部分结果、参与方法的调用和返回。每个线程在创建的时候会创建一个虚拟机栈，其内部保存一个个的栈帧(Stack Frame)，对应一次次java方法调用。

**特点：**

- 栈是一种快速有效的分配存储方式，访问速度仅次于程序计数器
- JVM直接对虚拟机栈的操作只有两个：每个方法执行，伴随着入栈，方法执行结束出栈
- 可以通过参数-Xss设置线程最大栈空间，栈大小决定了函数调用的最大可达深度

## 方法区

方法区 只是**JVM规范中定义的一个概念**，用于存储类信息、常量池、静态变量、JIT编译后的代码等数据，并没有规定如何去实现它，不同厂商有不同实现。**HotSpot在jdk7以前使用永久代实现方法区，jdk8以后使用元空间实现方法区**。永久代和元空间可以理解为方法区的落地实现。

jdk8以前调节方法区大小的方法:

```shell
-XX:PermSize=N //方法区（永久代）初始大小
-XX:MaxPermSize=N //方法区（永久代）最大大小，超出这个值将会抛出OutOfMemoryError 
```

jdk8元空间参数设置：

```shell
-XX:MetaspaceSize=N//设置Metaspace初始大小
-XX:MaxMetaspaceSize=N//设置Metaspace最大大小
```

## 本地方法栈

- 本地方法接口，一个Native Method 就是一个java调用非java代码的接口。Unsafe类就有很多本地方法
- 本地方法栈，虚拟机栈用于管理java方法的调用，**本地方法栈用于管理本地方法的调用**

## 堆

堆用于存放对象实例。为了高效垃圾回收，虚拟机把堆内存逻辑上划分成三块区域（分代的唯一理由是优化GC性能）

堆内存分为新生代和老年代，比例是1:2；新生代分为eden区和两个survivor区，比例是8:1:1。

> -XX:NewRatio=2 //老年代与新生代的比例



- 新生代（Young Generation），新对象和没达到一定年龄的对象都在新生代，年轻代垃圾收集称为Minor GC（Young GC）。
  - 大多数新创建的对象都位于Eden空间
  - **当Eden空间被对象填充满时，执行Minor GC**，并将所有幸存对象移至一个Survivor空间
  - Minor GC 检查幸存者对象，并将它们移动至另一个Survivor空间，因此总有一个Survivor空间是空的
  - 经过多次Minor GC后存活下来的对象被移动到老年代，通过设置年轻代的年龄阈值实现（**默认为15，可以通过-XX:MaxTenuringThreshold=N 设置年龄阈值**）
- 老年代（Old Generation），被长时间使用的对象，**大对象直接进入老年代（避免在Eden区和两个Survivor之间发生大量内存拷贝）**，老年代的内存空间应该要比年轻代大。通常，垃圾收集是在**老年代内存满时执行**，老年代垃圾收集称为Major GC（Old GC）。

> 部分收集：不是完整收集整个 Java 堆的垃圾收集。其中又分为：
>
> - 新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集
> - 老年代收集（Major GC/Old GC）：只是老年代的垃圾收集
>   - 目前，只有 CMS GC 会有单独收集老年代的行为
>   - 很多时候 Major GC 会和 Full GC  混合使用，需要具体分辨是老年代回收还是整堆回收
> - 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集
>   - 目前只有 G1 GC 会有这种行为
>
> 整堆收集（Full GC）：收集整个 Java 堆和方法区的垃圾

![img](https://github.com/lission/markdownPics/blob/main/java/jvm%20heap.jpeg?raw=true)

JVM虚拟机规范规定，java堆可以是处于物理上的不连续的内存空间，只要逻辑上是连续的即可，像磁盘一样。实现时，既可以是固定大小的，也可以是可扩展的，主流虚拟机都是可扩展的，（通过-Xmx和-Xms控制），如果堆中没有完成实例分配，且堆无法再扩展时，抛出OutOfMemeryError异常。

![img](https://github.com/lission/markdownPics/blob/main/java/jvm%20heap%20detail.jpeg?raw=true)



# 可能导致StackOverFlowError与OutOfMemeryError的原因，如果出现该如何处理

**OOM**：内存不足，垃圾回收后任然不足。

**可能产生原因：**

- 内存中加载的数据过多，如一次从数据库中取出过多数据；
- 集合对象引用过多且使用完后没有清空
- 代码中存在死循环或循环产生过多重复对象；
- 堆内存分配不合理。

**OOM处理：**

- jmap -histo:live [pid] 可以查看当前java进程创建的活跃对象数目和内存占用大小；

- jmap -dump:live,format-b,file=xxx.xxx [pid] 则可以将当前java进程的内存占用情况导出来，方便专门内存分析工具（如MAT）来分析。

**StackOverFlowError**：栈溢出，每个线程都拥有一个私有的虚拟机线程栈，用于存放当前线程的栈帧(被调用函数的参数、局部变量、返回地址等)。如果某个线程的线程栈空间被耗尽，没有足够资源分配给新创建的栈帧，就会抛出StackOverFlowError。

**可能产生的原因：**

- 递归调用过深，没有跳出递归条件
- 执行大量方法，导致线程栈空间耗尽
- 不断创建线程

**StackOverFlowError处理：**

- cpu100%处理：top命令查找问题进程；
- 如果是java进程，通过top -pH命令查找对应线程；
- 然后通过jstack命令查看线程堆栈情况，堆栈信息中nid对应线程id，top -Hp命令可以查看线程id。



# java类加载过程

类加载：虚拟机把类的数据加载到内存中，对数据进行校验、解析、初始化，最终到可以被虚拟机直接使用。

类加载的全过程是：加载—>验证—>准备—>解析—>初始化。

- **加载**：通过类的全限定类名获取类的二进制流，把类的静态存储结构(class文件)转换成方法区内运行时的数据结构。在堆中为该类生成一个class对象
- **验证：**验证class文件中的字节流信息是不是符合虚拟机的要求，不会威胁到jvm安全。验证阶段不是必须的，生产环境可考虑关闭**-Xverify:none**
- **准备：**为class对象的**静态变量**分配内存，并设置初始值。**按照变量类型赋值各自的默认值，比如int型赋值为0**
- **解析：**符号引用转为直接引用，比如：a类中引用b类，编译时a无法获取b的内存地址，用b的类名符号代替，解析时把b类的符号替换成内存地址
- **初始化：**执行构造器，静态代码块等

# 双亲委派模型

类加载器收到一个类加载请求时，首先委派父类加载器，父类类加载器在自己搜索范围内找不到这个类(无法加载)，子类才会加载。

- 启动类加载器（BootStrap ClassLoader）：加载java的核心类库，无法被java程序直接使用 如 rt.jar
- 扩展类加载器（Ext ClassLoader）：加载java扩展库，在java虚拟机的扩展库目录里查找加载java类 jre/lib/ext路径下
- 系统类加载器（Application ClassLoader）：根据java类的路径加载类，加载java应用里的类，加载与程序有关系的和我们写的java类
- 自定义类加载器：自己写的继承ClassLoader

双亲委派可以**防止内存中出现相同的字节码**。防止核心类不会被篡改，即使篡改也无法加载。

打破双亲委派模型，自定义类加载器，继承ClassLoader，重写LoadClass方法。

# 垃圾回收
## 如何判断一个对象是否可以回收

- **引用计数器算法**，给对象引用添加一个计数器，当引用+1时，计数器+1，引用失效时，计数器-1。当计数器为0时，对象可回收。如果出现循环引用，计数器永不为0，因此java虚拟机不使用引用计数算法。
- **可达性分析算法**，通过GC Roots作为起始点进行搜索，能够到达的对象都是存活的，不可达的对象被回收。java虚拟机使用该算法来判断对象是否可被回收。

> 对象被系统宣告死亡，至少要标记两次。当对象到GC Roots没有任何引用链，就认为是不可用的，会被第一次标记；第二次是虚拟机自动建立的finalizer队列中判断是否需要执行finalizer()方法。

> 当一个对象可被回收时，如果需要执行该对象的 finalize() 方法，那么就有可能通过在该方法中让对象重新被引用，从而实现自救。自救只能进行一次。**不推荐使用该方法，该方法运行代价高昂，不确定性大，无法保证各个对象调用顺序**

GC Roots包含以下内容：

- 虚拟机栈中引用的对象
- 本地方法栈中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中的常量引用的对象



## 如何触发Young GC和Full GC

对于Yong GC(Minor GC)，触发条件很简单，当Eden空间满时，就触发一次Young GC。

**Full GC**是对整个堆和方法区进行垃圾回收，触发条件相对复杂，有以下条件：

- **调用System.gc()**，只是建议虚拟机进行Full GC，虚拟机不一定执行。不建议采用这种方式
- **老年代空间不足**
- **空间分配担保不足**，使用复制算法的Minor GC需要老年代的内存空间作担保，如果担保失败，执行一次Full GC。

> 在发生Minor GC之前，虚拟机先检查老年代最大可用连续空间是否大于新生代所有对象总空间，如果条件成立，那么Minor GC可以确认是安全的

- **jdk7及以前的永久代空间不足**，在jdk及以前，HotSpot虚拟机中方法区是用永久代实现的，永久代中存放一些类信息、常量、静态变量等数据。当系统中要加载的类、反射的类、调用的方法较多时，永久代可能会占满，在未配置CMS GC情况下会执行Full GC。
- **Concurrent Mode Failure**，执行CMS GC 的过程中同时有对象放入老年代，老年代空间不足（可能是GC过程中浮动垃圾过多导致暂时性的空间不足），便会报告Concurrent Mode Failure，并触发Full GC。

## 对于跨代垃圾回收，如果存在跨代引用，如何处理

Card Table，卡表，用来标记卡页的状态，每个卡表项对应一个卡页，当卡页中一个对象引用有写操作时，写屏障会标记对象所在的卡表状态为dirty，垃圾回收时只要筛选出卡表中变脏的元素，轻易得出哪些卡页对应的内存包含跨代指针。卡表的本质是用来解决跨代引用。使用卡精度（记录的是新生代一段地址是否存在被老年代引用的记录）的方式实现记忆集。HotSpot中使用字节数组形式实现卡表。



## java四种引用类型

- 强引用，被强引用引用的对象，不会被垃圾回收。通过new 一个对象，创建一个强引用

```java
Object obj = new Object();
```

- 软引用，被软引用引用的对象，只有在内存空间不足时，才会被垃圾回收。使用SoftReference类创建软引用

```java
Object obj = new Object();
SoftReference srf = new SoftReference<Object>(obj);
obj = null;//使对象只被软引用引用
```

- 弱引用，被弱引用引用的对象，一定会被垃圾回收器回收，即它只能存活到下一次垃圾回收之前。使用WeakReference类创建弱引用

```java
Object obj = new Object();
WeakReference wrf = new WeakReference<Object>(obj);
obj = null;//使对象只被弱引用引用
```

- 虚引用，一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用取得对象。对一个对像设置虚引用关联的唯一目的是就能在这个对象被回收时收到一个系统通知。使用PhantomReferrnce来实现虚引用

```java
Object obj = new Object();
PhantomReference prf = new PhantomReference<Object>(obj);
obj = null;//使对象只被虚引用引用
```



## 垃圾回收算法

### 标记-清除

![img](https://github.com/lission/markdownPics/blob/main/java/%E6%A0%87%E8%AE%B0-%E6%B8%85%E9%99%A4%E6%B3%95.jpeg?raw=true)

将存活的对象进行标记，然后清理掉未被标记的对象。

不足：

- 标记、清除效率都不高
- 会产生大量不连续的内存碎片，导致无法给大对象分配内存

### 标记-整理

![img](https://github.com/lission/markdownPics/blob/main/java/%E6%A0%87%E8%AE%B0%E6%95%B4%E7%90%86%E6%B3%95.jpeg?raw=true)

让存活的对象向一端移动，然后清理边界之外的内存

### 复制

![img](https://github.com/lission/markdownPics/blob/main/java/%E5%A4%8D%E5%88%B6%E6%B3%95.jpeg?raw=true)

将内存划分为大小相等的两部分，每次只使用其中一块，当这一块内存用完了，就将还存活的对象复制到另一块上，然后把使用过的内存进行一次清理。

主要不足是，只使用一半内存。

现代商用虚拟机都采用这种收集算法来回收新生代，但不是将新生代划分为相等的两块，而是划分为一块较大的Eden区和两个相等的Survivor区。每次使用Eden空间和一个Survivor空间。在回收时，将Eden和Survivor中存活的对象一次复制到另一块Survivor空间上，最后清理Eden和使用过的Survivor区。

HotSpot虚拟机Eden和Survivor大小比例默认8:1,保证内存使用率达到90%。如果每次存活有超过10%的对象存活，那么一块Survivor空间不够，此时需要老年代进行内存分配担保，借用老年代空间存储放不下的对象。

### 分代收集

现在的商用虚拟机采用分代收集算法，它根据对象存活周期将内存划分为几块，不同块采用不同算法。

一般将堆分为新生代和老年代

新生代采用复制算法

老年代采用标记清除或标记整理算法



## 垃圾收集器

- 单线程与多线程，单线程是指垃圾收集器只使用一个线程执行，而多线程使用多个线程
- 串行和并行，串行指的是垃圾收集器与用户线程交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序；并行指的是垃圾收集器与用户程序同时执行。CMS和G1垃圾收集器是并行的。
- jdk 1.8默认使用Parallel收集器，jdk9默认G1收集器，一直到jdk17

![img](https://github.com/lission/markdownPics/blob/main/java/%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8%E6%BC%94%E5%8F%98.png?raw=true)

## 什么是安全点和安全区域

- **安全点(safepoint)**，分析过程中对象引用关系不会发生变化的点。适合产生安全点的就是指令序列复用的地方，如：方法调用、循环跳转、异常跳转等。
- **安全区域(safe region)**，引用关系不会发生变化的代码片段。线程执行到安全区域时，会先标记自己已进入安全区域，不会阻碍垃圾收集的发生。

## 可达性分析必须 Stop The World（暂停用户线程，保证标记期间对象的引用关系不再变化），虚拟机如何缩短停顿时间

先进行一次并发的可达性分析，再进行一次Stop The World的可达性分析。

- 第一次可达性分析，引用发生变化时需要记录（两种方案）
  - 1、可达对象A指向未知对象，记录下这个A对象，作为第二次可达分析的GC Roots(增量更新)，应用于CMS垃圾收集器
  - 2、对象A正在分析时，删除了一个未知对象的引用，记录下这个A对象，作为第二次可达性分析的GC Roots(原始快照SATB)，应用于G1垃圾收集器
- 第二次可达性分析，以第一次分析时记录的对象作为GC Roots，GC Roots数量变少，停顿时间减少

## GC Roots根节点枚举必须Stop The World ,如何缩短停顿时间

引入Oop Map记录对象指针，这样根节点枚举不需要去检查所有的执行上下文和全局的引用位置。通过Oop Map可以快速准确完成GC Roots枚举。

**只有在安全点生成Oop Map**，减少空间成本

## 垃圾收集发生时，如果还没到达安全点（OopMap未生成），此时暂停线程，那么GC Roots根节点不正确，如何解决

当垃圾收集发生时，无法直接中断用户线程，只能简单的设置一个中断标志，线程执行到安全点后紧接着轮询检查中断标识是否为true，如果为true就中断挂起。这样**保证所有线程都是在安全点停下**。



# JVM调优

> JVM调优是最后不得已才采用的手段
>
> 多数GC频繁问题都是代码问题，而非参数设置不合理
>
> 分析JVM运行，然后优化代码比起优化虚拟机本身更常见



## 调优思路

- 内存方面
  - 合理内存总大小
  - 每个区域分配合理内存大小
  - 选择合适垃圾回收算法、控制GC停顿次数和时间
  - 解决内存泄露问题，辅助进行代码优化
  - 内存热点：检查哪些对象在系统中占用内存较大，是否可以通过代码调优
- 线程方面
  - 死锁检查，辅助代码优化
  - Dump线程详细信息，查找竞争线程，通过分析辅助进行代码优化
  - CPU热点，检查系统中哪些方法占用大量CPU时间，通过分析辅助进行代码优化

## 如何调优

- 监控JVM状态，主要有内存、垃圾回收、线程、代码、I/O
- 分析结果，判断是否需要优化
- 调整：
  - 垃圾回收(垃圾收集器、参数配置)
  - 内存分配
  - 代码优化
- 重复监控、分析、调整，找到平衡点

## 调优目标

- GC时间足够小
- GC次数足够小
- 将转移到老年代的对象数量降低到最小
- 减少FULL GC执行时间
- 发生FULL GC间隔足够长(至少1min以上)

## 调优经验

- 要想GC时间短，需要一个比较小的堆；要保证GC次数足够少，又需要更大的堆；两者冲突，取其平衡

- 通常把堆的最小、最大值设置为同一个数值【-Xms -Xmx】为了防止GC后缩容\扩容操作的开销

- 老年代和新生代比例默认为2:1，可以通过-XX:NewRation=n调整，也可以通过 `-XX:newSize` 和 `-XX:MaxNewSize` 设置，同样地，为了防止扩/缩容操作的开销，一般设置为同一个数值【简化参数：`-Xmn=n`】。

- 通过不断监控调整，合理规划新生代和老年代大小

- 应用存在大量临时对象，应选择更大的新生代；如果存在相对较多持久对象，老年代应适当增大。遵循尽量少的FULL GC原则

- 通过观察应用一段时间，看其峰值的时候老年代会占用多少内存，在不影响 Full GC 的前提下，根据实际情况加大新生代，但至少要给老年代预留 1/3 的增长空间。

- 每个线程的栈大小默认是1M，32位为512K，一版来说256K足够了，可以通过 -XX:ThreadStackSize=256设置，可以产生更多线程

# JVM调试工具

通过java调试/排查工具进行问题定位

## jstack

jstack是jdk自带的**线程堆栈分析工具**，使用该命令可以查看或导出java应用程序中线程堆栈信息。

- 常用命令

```shell
jstack 2815  # pid 
jstack -m 2815  # java 和 native所有栈信息
jstack -l 2815 # 锁信息列表 ，查看是否死锁
```

## jstat

jstat参数众多，记住一个即可

```shell
jstat -gcutil 2815 1000 # 每隔1秒钟更新出来最新的一行jstat统计信息,包含JVM内的Eden、Survivor、老年代的内存使用情况，还有Young GC和Full gC的执行次数以及耗时
```

## jmap

jmap的作用是监控内存内的java对象。

> jmap -histo将不触发完整的gc，但是jmap -histo:live将会触发

jmap [option]  [pid-进程id]

option 选项命令说明：

- -heap：打印java堆概要信息，包括使用的GC算法，堆配置参数和各代中堆内存使用情况

```shell
jmap -heap 2819
```



- -histo[:live]：打印堆中对象直方图，该图可以获取每个class对象数目，占用内存大小和类全名信息，带上:live 只统计活着的对象

```shell
jmap -histo:live 2819
```



- -finalizerinfo：打印等待回收对象信息

```shell
jmap -finalizerinfo 2819
```

- -dump：以hprof二进制格式将java堆信息输出到文件内，该文件可以用MAT、VisualVM或jhat等工具查看

  dump-option选型：

  - live：只输出活着的对象，不指定，则输出堆中所有对象
  - format=b,指定输出格式为二进制
  - file = ：指定文件名及文件存储位置：如：jmap -dump:live,format-b,file=/opt/hprof.bin [pid] 



# Arthas

arthas，阿里开源的Java诊断工具，arthas基于Greys二次开发而来。

可以帮助我们解决：

- 线上问题定位
- 监控JVM实时运行状况
- 查看系统的运行状况

## 1、参考内容

https://www.jianshu.com/p/507f7e0cc3a3

https://www.pdai.tech/md/java/jvm/java-jvm-agent-arthas.html#arthas%e7%ae%80%e4%bb%8b

## 2、启动命令

```shell
java -jar arthas-boot.jar
```

**注意，启动arthas的用户必须与要连接的服务启动用户保持一致，否则arths会attach失败**

## 3、常用命令

### 3.1、watch

watch命令，可以用来查看方法的入参及返回值

示例：

```shell
watch com.etcc.monitor.warning.business.company.provider.SCompanyDataReportStatProviderImpl getReportDataCount  "{params,returnObj}" -x 3 

watch com.swan.smartcity.etc.pre.warning.business.service.MonitorStrategyServiceImpl findMonitorStrategyPage "{params,returnObj}" -x 3 
```

### 3.2、trace

观察方法执行的时候哪个子调用比较慢

```shel
trace com.swan.smartcity.etc.pre.warning.business.service.MonitorStrategyServiceImpl findMonitorStrategyPage
```

### 3.3.、stack

查看某个方法的调用栈信息

```shell
stack com.swan.smartcity.etc.pre.warning.business.service.MonitorStrategyServiceImpl findMonitorStrategyPage
```

### 3.4、jvm

查看jvm信息

### 3.5、thread命令

查看当前线程信息，查看线程的堆栈

- thread -b可以找到阻塞住其他线程的线程；
- thread 线程id 查看指定线程的运行堆栈；
- thread -n 3 查看最忙的三个线程，打印堆栈；

### 3.6、heapdump

生成内存快照，类似于jmap的head dump功能

```shell
headdump /tmp/dump.hprof
```

### 3.7、dashboard

当前系统的实时数据面板，查看线程状况和内存使用情况

### 3.8、redefine

重新载入定义的类，无需重启和发布，很方便

```shell
redefine /tmp/com/example/demo/arthas/user/UserController.class
```

