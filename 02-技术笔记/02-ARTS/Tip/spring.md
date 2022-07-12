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



## spring的FactoryBean与BeanFactory

- BeanFactory：**Bean工厂，是一个工厂**，是ioc容器的最顶层接口，作用是管理Bean，即实例化、定位、配置应用程序中的对象及建立这些对象之间的依赖，ApplicationContext就是它的子类
- FactorBean：**工厂bean，是一个bean**，可以用于**自定义bean的创建过程**，这个接口有三个方法，getObject()、getObjectType()、isSingleton()用于获取bean对象，bean的类型，和是否单例，实现这个接口可以自定义bean的创建。spring中的bean包括普通bean和factorybean。在bean加载过程中会用到，单例已经开始初始化的时候通过getObject()方法初始化factorybean。

## BeanFactory和ApplicationContext区别

- BeanFactory，是spring中ioc容器的最顶层接口，是ioc的核心，拥有多种实现如：application、xmlbeanfactory
- ApplicationContext：继承BeanFactory接口，高级的容器。除了BeanFactory的初始化bean和获取bean功能，该提供了更多有用的功能，如：继承了MessageSource,支持国际化；统一的资源文件访问方式；同时加载多个配置文件；载入多个（有继承关系）上下文，使每一个上下文都专注于一个特定的层次，比如应用的web层等。
- BeanFactory是延迟加载，在使用的时候才会初始化Bean，而ApplicationContext在启动时就默认加载了所有bean（可以设置延迟）。
- 使用BeanFactory启动时占用资源少，如果bean配置的有问题ApplicationContext在启动时就可以发现配置的问题，同时bean加载之后系统运行较快。

## spring支持的几种bean的作用域

- singleton，单例模式，默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护，该对象的生命周期与spring ioc容器一致。（第一次注入时才被创建）
- prototype，原型模式，为每一个bean请求创建一个bean实例，在每次注入时都会创建一个新的对象
- request，bean被定义为在每个http请求中创建一个单例对象，也就是在单个请求中都会复用这个bean实例
- session，与request范围类似，确保每个session中都有一个bean实例，在session过期后，bean随之失效
- application，bean被定义为在ServletContext的生命周期中复用一个单例对象
- websocket，bean被定义为在websocket的生命周期中复用一个单例对象



## spring bean加载过程和生命周期

spring 通过容器存放应用中bean，并管理bean从创建到销毁的完整生命周期
spring bean的完整生命周期：

1. 对bean实例化，默认单例bean
2. 对bean进行依赖注入，填充属性，为属性或引用的bean赋值
3. 如果bean实现了BeanNameAware接口，将bean的id传递给setBeanName()方法
4. 如果bean实现了BeanFactoryAware接口，调用setBeanFactory()方法，将BeanFactory实例传入
5. 如果bean实现了ApplicationContextAware接口，调用setApplicationContext()接口，将bean所在应用上下文引用传入进来
6. 如果bean实现了BeanPostProcessor接口，调用它们的postProcessorBeforeInitialization()方法
7. 如果bean中有方法添加了@PostConstruct注解，那么该方法将被调用；
8. 如果实现了InitializingBean接口，spring将调用它们的afterPropertiesSet()方法，如果bean使用init-method声明了初始化方法，该方法也将被调用
9. 如果实现了BeanPostProcessor接口，spring将调用它们的postProcessorAfterInitialization()方法
10. 此时，bean准备就绪，可以使用，将一直停留在应用上下文中，直到上下文被销毁
11. 如果bean实现了DisposableBean接口，spring将调用它的destroy()接口，类似的，如果bean使用destroy-method声明了销毁方法，该方法也会被调用

## spring框架中的单例bean是线程安全的吗

单例bean是非线程安全的。

如果bean是有状态的，需要开发人员自己来进行线程安全的保证。最简单办法是改变bean作用域，将singleton改变为prototype，每次请求bean时都创建一个新的bean实例。

- 有状态就是有数据存储功能
- 无状态就是不会保存数据

不要在bean中声明任何有状态的实例变量或成员变量，如果必须如此，那么久使用ThreadLocal把变量变为线程私有的，如果bean的实例变量或类变量需要在多个线程间共享，那么就要使用synchronized，lock，cas等实现线程同步。

## spring 常用注解及用法

- @SpringBootApplication：启动类
- @EnableTransactionManagement：事务
- @Transactional：方法上加事务
- @Configuration：配置类
- @Bean：配置bean
- @Component、@Controller、@Service、@Repository：把类标识为bean
- @RequestMapping、@PostMapping、@GetMapping：controller层的接口
- @Autowired：依赖注入、 @Qualifier：指定注入的bean的名称
- @Value:读取配置信息

## spring中使用到的设计模式

单例模式、工厂模式、适配器模式(spring mvc的handlerAdapter)、代理模式(aop)、模板方法(父类定义了框架，特定实现通过子类)



# spring aop

## spring aop的理解

aop，**Aspect-oriented Programming 面向切面编程**。

oop，**Object-oriented Programming 面向对象编程**。

当需要为多个不具有继承关系的对象引入同一个**公共行为**时（如：日志、权限校验、事务管理、性能监控），oop只能在每个对象里引用公共行为(方法)，这样程序中产生了大量重复代码。

AOP通过预编译和运行时动态代理，实现在不修改源代码的情况下，给程序添加统一的功能。

spring aop是一种编程范式，主要目的是将非功能性需求从功能性需求中分离出来，达到解耦目的。

## aop核心概念

- Aspect 切面，切面一个关注点的模块化，这个关注点横跨多个类。比如：应用程序中的事务管理就是一个横切关注点。
  - 注解式声明，用@Aspect标记一个类，在这个类中通过一系列配置实现AOP
- Join point 连接点，连接点表示方法的执行
- Advice 通知，通知指一个切面在特定连接点执行的动作，使用拦截器实现。spring aop 包含五种通知

| 注解              | 通知           | 说明                                                       |
| ----------------- | -------------- | ---------------------------------------------------------- |
| `@Before`         | 前置通知       | 方法执行之前执行的通知                                     |
| `@AfterReturning` | 后置通知       | 方法正常结束执行的通知                                     |
| `@AfterThrowing`  | 异常抛出后通知 | 方法抛出异常后执行的通知                                   |
| `@After`          | 最终通知       | 连接点结束后执行的通知，异常退出也要执行                   |
| `@Around`         | 环绕通知       | 最强大的通知，可以实现其他所有通知，甚至可以连方法都不执行 |

- Pointcut 切入点，切入点是aop的核心，它是匹配连接点的断言，类比正则表达式，spring默认使用AspectJ 切入点表达式
- Target object 目标对象，也被称为通知对象，在spring 中该对象始终是一个代理对象，代理连接点所在的类实例
- introduction 引入，引入能够为目标对象附加额外的方法或字段
- AOP Proxy ，为实现面向切面编程而创建的对象，spring中使用jdk动态代理或CGLIB动态代理
- Weaving 织入，将切面与切入点所在的类或对象链接，以创建通知对象。spring在运行时执行织入操作

## aop原理

aop通过运行时代理实现的。

> 代理模式，为其他对象提供一种代理，以提供对这个对象的访问。用来增强目标对象。好处是保护目标对象，降低系统耦合度，扩展性好。

![img](https://github.com/lission/markdownPics/blob/main/spring/%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F%E5%AF%B9%E6%AF%94.png?raw=true)

# springboot

## spring、spring MVC、springboot 区别

- spring，是开源的J2EE框架，是一个轻量级的控制翻转（IOC）和面向切面（AOP）容器框架，可以使开发更便捷简单。
  - 通过ioc，达到解耦合
  - 通过aop，分离应用的业务逻辑与系统级服务
  - 包含并管理应用对象bean的配置和生命周期，这个意义上是一个容器
  - 将简单的组件配置、组合成复杂的应用，比如redis、mybatis等。这个意义上是一个框架

- spring MVC ，是spring对web框架的一个解决方案，提供了一个总的前端控制器Servlet，用来接收请求，然后定义了一套路由策略(url到handle的映射)及适配器执行handle，将handle结果使用视图解析技术生成视图展现给前端
- spring boot，是spring提供的一个快速开发工具包，让程序员能够更方便、更快速开发spring+springMVC应用。简化了配置，整合了一系列解决方案(starter机制)，可以开箱即用