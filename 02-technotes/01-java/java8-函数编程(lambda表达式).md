[TOC]

# 1、简介

面向对象编程是对数据进行抽象，面向函数编程是对行为进行抽象。

面向函数编程核心思想：**使用不可变值和函数，函数对一个值进行处理，映射成另一个值**。

[参考](https://www.pdai.tech/md/java/java8/java8-stream.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E5%87%BD%E6%95%B0%E6%8E%A5%E5%8F%A3)

# 2、stream & parallelStream



## 2.1、 stream & parallelStream

stream 顺序流；parallelStream 并行流

==并行流，数组被分成多段，每一个在不同的线程中处理，结果一起输出（利用多核技术）==



## 2.2、parallelStream原理

```java
List originalList = someData;
split1 = originalList(0, mid);//将数据分小部分
split2 = originalList(mid,end);
new Runnable(split1.process());//小部分执行操作
new Runnable(split2.process());
List revisedList = split1 + split2;//将结果合并
```

拓展：

hadoop(能够对大数据处理的分布式系统架构)，其中的MapReduce，处理大数据核心思想是大而化小，分配到不同机器运行map，最终通过reduce将所有机器结果结合起来得到最终结果。

Stream利用多核技术将大数据通过多核并行处理，MapReduce可以分布式

### stream与parallelStream性能测试对比

如果是多核机器，理论上并行流会比顺序流快上一倍



# 3、 min,max,summaryStatistics

summaryStatistics

```java
        List<Integer> primes = Arrays.asList(2, 3, 5, 7, 11, 13, 17, 19, 23, 29);
        IntSummaryStatistics stats = primes.stream().mapToInt((x) -> x).summaryStatistics();
        System.out.println("Highest prime number in List : " + stats.getMax());
        System.out.println("Lowest prime number in List : " + stats.getMin());
        System.out.println("Sum of all prime numbers : " + stats.getSum());
        System.out.println("Average of all prime numbers : " + stats.getAverage());
```



# 4、内置四大函数接口

- 消费型接口: Consumer< T> void accept(T t)，==有参数，无返回值==的抽象方法；
- 供给型接口: Supplier < T> T get()，==无参有返回值==的抽象方法；
- 断定型接口: Predicate<T> boolean test(T t):==有参，但是返回值类型是固定的boolean==
- 函数型接口: Function<T,R> R apply(T t)，==有参有返回值==的抽象方法；