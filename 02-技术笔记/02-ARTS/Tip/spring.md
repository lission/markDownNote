[TOC]

# spring事务

## spring的事务传播行为

事务的传播行为，**多个事务方法相互调用时，事务如何在这些方法间传播**。

7种事务传播类型，默认值为**Propagation.REQUIRED**。可以手动指定其他事务传播行为，如下：

- Propagation.REQUERIED：如果当前存在事务，则加入该事务；如果当前不存在事务，**则创建一个新事务**。
- Propagation.SUPPORTS：如果当前存在事务，则加入该事务；如果当前不存在事务，**则以非事务方式继续运行**。
- Propagation.MANDATORY：如果当前存在事务，则加入该事务；如果当前不存在事务，**则抛出异常**。
- Propagation.REQUIRES_NEW：重新创建一个新的事务，如果当前存在事务，挂起当前的事务。
- Propagation.NOT_SUPPORTED：以非事务的方式运行，如果当前存在事务，挂起当前事务。
- Propagation.NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
- Propagation.NESTED：如果没有事务，就新建一个事务；如果有事务，就在当前事务中**嵌套**其他事务。

> NESTED与REQUIRES_NEW区别：
>
> REQUIRES_NEW是创建一个新事务并且新开启的事务与原事务无关，而NESTED则是当前事务存在事务时，(当前事务称为父事务)会开启一个**嵌套事务**(称为子事务)。在NESTED情况下父事务回滚时，子事务也会回滚；在REQUIRES_NEW情况下，原事务回滚，不影响子事务。
>
> NESTED与REQUIRED区别：
>
> RQUEIRED情况下，调用方法存在事务时，则被调用方与调用方使用同一事务，被调用方出现问题时，由于共用事务，无论调用方是否catch其异常，事务都会回滚。而在NESTED情况下，被调用方发生异常时，调用方可以catch其异常，这样只有子事务回滚，父事务不受影响。



## spring事务实现原理

@Transactional注解是声明式的事务实现。spring事务采用AOP方式实现，在一个方法上添加了@Transactional注解后，Spring会基于这个类生成一个代理对象，会将这个代理对象作为Bean。***默认回滚异常是RuntimeException和Error***。

事务执行时先执行**TransactionAspectSupport.invokeWithinTransaction()**方法，在这个方法中：

- 1）获取事务属性(@Transactional注解的参数)，rollbackFor指定回滚哪种异常，noRollbackFor指定不回滚哪种异常，propagation指定传播行为，readOnly，是否制度，isolation隔离级别，timeout超时时间
- 2）加载配置中的TransactionManager
- 3）获取收集事务信息TransactionInfo
- 4）执行目标方法（即添加注解的方法）
- 5）出现异常，尝试处理
- 6）清理事务相关信息
- 7）提交事务

## spring事务隔离级别

spring事务隔离级别就是数据库的隔离级别，外加一个默认级别（**DEFAULT 这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别**.）

- read uncommitted（未提交读）
- read committed（提交读、不可重复度）
- repeatable read（可重复度）
- serializable（可串行化）

> Mysql数据库默认隔离级别是rr，spring默认隔离级别
>
> 以spring隔离级别为准，如果spring设置的隔离级别数据库不支持，效果取决于数据库