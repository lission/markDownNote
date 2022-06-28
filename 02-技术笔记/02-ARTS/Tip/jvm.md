[TOC]

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



# jmap

jmap的作用是监控内存内的java对象

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

arths，阿里开源的Java诊断工具

## 1、参考内容

https://www.jianshu.com/p/507f7e0cc3a3

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

### 3.2、jvm

查看jvm信息

### 3.3、thread命令

查看当前线程信息，查看线程的堆栈

- thread -b可以找到阻塞住其他线程的线程；
- thread 线程id 查看指定线程的运行堆栈；
- thread -n 3 查看最忙的三个线程，打印堆栈；

### 3.4、heapdump

生成内存快照，类似于jmap的head dump功能

```shell
headdump /tmp/dump.hprof
```

### 3.5、dashboard

查看线程状况和内存使用情况
