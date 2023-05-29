[TOC]

# Arthas

arthas，阿里开源的Java诊断工具，arthas基于Greys二次开发而来。

可以帮助我们解决：

- 线上问题定位
- 监控JVM实时运行状况
- 查看系统的运行状况

## 1、参考内容

https://arthas.aliyun.com/doc/

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

### 3.3、stack

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

