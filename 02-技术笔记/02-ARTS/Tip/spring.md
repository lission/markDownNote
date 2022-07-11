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



# spring

## spring的bean在什么情况下不使用单例

有状态的bean不用单例。即有属性会被修改的bean。



## spring循环依赖

[参考](https://mrbird.cc/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Spring%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.html)

所谓循环依赖是指：BeanA对象的创建依赖于BeanB，BeanB对象的创建也依赖于BeanA，造成了死循环。

spring通过**提前曝光机制**，**利用三级缓存**解决循环依赖问题。

常见的两种循环依赖情况：

- 普通bean与普通bean之间的循环依赖
- 普通bean与代理bean之间的循环依赖

### bean创建过程

IOC容器获取bean的入口为AbstractBeanFactory的getBean方法，具体逻辑在doGetBean方法内：

> doGetBean方法中先通过getSingleton(String beanName)方法从三级缓存中获取Bean实例，如果不为空则进行后续处理；如果为空，则通过getSingleton(String beanName,ObjectFactory<?> singletonFactory)方法创建Bean实例并进行后续处理。
>
> 这两个方法都是AbstractBeanFactory父类DefaultSingletonBeanRegistry的方法。



getSingelton(String beanName)源码：

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {

    ......

    @Override
    @Nullable
    public Object getSingleton(String beanName) {
        return getSingleton(beanName, true);
    }

    ......

    @Nullable
    protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        // 从一级缓存中获取目标Bean实例
        Object singletonObject = this.singletonObjects.get(beanName);
        // 如果从一级缓存中没有获取到，并且该Bean处于正在创建中的状态时
        if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
            // 从二级缓存获取目标Bean实例
            singletonObject = this.earlySingletonObjects.get(beanName);
            // 如果没有获取到，并且允许提前曝光的话
            if (singletonObject == null && allowEarlyReference) {
                synchronized (this.singletonObjects) {
                    // 在锁内重新从一级缓存中往下查找
                    singletonObject = this.singletonObjects.get(beanName);
                    if (singletonObject == null) {
                        singletonObject = this.earlySingletonObjects.get(beanName);
                        if (singletonObject == null) {
                            // 从三级缓存中取出目标Bean工厂对象
                            ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                            if (singletonFactory != null) {
                                // 工厂对象不为空，则通过调用getObject方法实例化Bean实例
                                singletonObject = singletonFactory.getObject();
                                // 放到二级缓存中
                                this.earlySingletonObjects.put(beanName, singletonObject);
                                // 删除对应的三级缓存
                                this.singletonFactories.remove(beanName);
                            }
                        }
                    }
                }
            }
        }
        return singletonObject;
    }
```



所谓三级缓存指的是DeafultSingletonBeanRegistory类的三个成员变量：

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {

    /** Cache of singleton objects: bean name to bean instance. */
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

    /** Cache of singleton factories: bean name to ObjectFactory. */
    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

    /** Cache of early singleton objects: bean name to bean instance. */
    private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);

    ......

}
```

- singletonObjects，一级缓存，key为Bean名称，value为Bean实例。这里的Bean实例指的是**已经完全创建好的**，即已经经历**实例化->属性填充->初始化以及各种后置处理过程的Bean**，可直接使用。
- earlySingletonObjects，二级缓存，key为Bean名称，value为Bean实例。这里的Bean实例指的是仅完成实例化的Bean，还未进行属性填充等后续操作。**用于提前曝光，供别的Bean引用，解决循环依赖**。
- singletonFactories，三级缓存，key为Bean名称，value为Bean工厂。在Bean实例化后，属性填充之前，如果允许提前曝光，spring会把该Bean转换成Bean工厂并加入到三级缓存。在需要引用提前曝光对象时再通过工厂对象的getObject()方法获取。

如果通过三级缓存的查找都没有找到目标Bean示例，则通过getSingleton(String beanName,ObjectFactor<?> singletonFactory)方法创建：

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {

    ......

    public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        synchronized (this.singletonObjects) {
            // 从一级缓存获取
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                // 为空则继续
                ......
                // 方法内会将当前Bean名称添加到正在创建Bean的集合（singletonsCurrentlyInCreation）中
                beforeSingletonCreation(beanName);
                boolean newSingleton = false;
                ......
                try {
                    // 通过函数式接口创建Bean实例，该实例已经经历实例化->属性填充->初始化以及各种后置处理过程，可直接使用
                    singletonObject = singletonFactory.getObject();
                    newSingleton = true;
                }
                catch (IllegalStateException ex) {
                   ......
                }
                finally {
                    ......
                }
                if (newSingleton) {
                    // 添加到缓存中
                    addSingleton(beanName, singletonObject);
                }
            }
            return singletonObject;
        }
    }

    protected void addSingleton(String beanName, Object singletonObject) {
        synchronized (this.singletonObjects) {
            // 添加到一级缓存
            this.singletonObjects.put(beanName, singletonObject);
            // 删除对应的二三级缓存
            this.singletonFactories.remove(beanName);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }

    ......

}
```



## spring怎么处理循环依赖

bean循环依赖a——》b——》a

- 如果是**构造函数注入，无法解决**，**这是因为对象的提前曝光时机发生在对象实例化之后，而构造器注入时机为对象实例化时，所以此时还未进行提前曝光操作，循环依赖也就没办法解决了**。

  a在创建的时候会放到当前创建池(缓存)中，如果创建成功就从池中删除，创建时发现需要使用到b，就去创建b，这时发现又需要创建a，而a在当前创建池中，这时就会抛出异常。

- 基于属性注入的情况，setter注入可以处理单例模式的循环依赖：a创建成功，并暴露objectFactory，同时b创建也能成功，b创建a所以b会去缓存中查找a并注入，a的原始bean经创建成功，所以可以setter到b的属性，b也可以set到a的属性。

- prototype作用域的bean无法处理，因为spring不缓存prototype作用域的bean。