[TOC]

# spring事务

## spring的事务传播行为

事务的传播行为，**多个事务方法相互调用时，事务如何在这些方法间传播**。

7种事务传播类型，默认值为**Propagation.REQUIRED**。可以手动指定其他事务传播行为，如下：

- Propagation.REQUERIED：表示**当前方法必须在事务中运行**。
  - 如果调用者有事务，则当前方法**加入到调用者事务中**运行。
  - 如果调用者没有事务，则当前方法自己新开启一个事务运行。

- Propagation.SUPPORTS：表示当前方法**不必(可有可无)在事务**中运行。
  - 如果调用者有事务，则当前方法加入到调用者事务中运行。
  - 如果调用者没有事务，则当前方法**以非事务的形式**运行

- Propagation.MANDATORY：表示当前方法**必须在调用者事务中**运行。
  - 如果调用者有事务，则当前方法加入到调用者事务中运行。
  - 如果调用者没有事务，则当前方法**抛出异常**。

- Propagation.NESTED：表示当前方法**必须在事务中**运行。
  - 如果调用者有事务，则当前方法以**“嵌套事务”（子事务）的形式加入到调用者事务中**运行。
  - 如果调用者没有事务，则当前方法自己新开启一个事务运行。

- Propagation.NEVER：表示调用者**必须以非事务形式**运行。
  - 如果**调用者有事务，则抛出异常**。
  - 如果调用者没有事务，则当前方法以非事务的形式运行

- Propagation.REQUIRES_NEW：表示当前方法**必须在事务中**运行。
  - 如果调用者有事务，则**==挂起调用者事务==**，当前方法自己**新开启一个事务**运行。
  - 如果调用者没有事务，则当前方法自己新开启一个事务运行

- Propagation.NOT_SUPPORTED：表示当前方法**不支持在事务**中运行。
  - 如果**调用者有事务，则==挂起调用者的事务==，当前方法以非事务的形式运行**。
  - 如果调用者没有事务，则当前方法以非事务的形式运行


> NESTED与REQUIRES_NEW区别：
>
> **REQUIRES_NEW是创建一个新事务并且新开启的事务==与原事务无关==**，而NESTED则是当前事务存在事务时，(当前事务称为父事务)会开启一个**嵌套事务**(称为子事务)。在NESTED情况下父事务回滚时，子事务也会回滚；在REQUIRES_NEW情况下，原事务回滚，不影响子事务。
>
> NESTED与REQUIRED区别：
>
> RQUEIRED情况下，调用方法存在事务时，则被调用方与调用方使用同一事务，被调用方出现问题时，由于共用事务，无论调用方是否catch其异常，事务都会回滚。而在NESTED情况下，被调用方发生异常时，调用方可以catch其异常，这样只有子事务回滚，父事务不受影响。



## spring事务实现原理

@Transactional注解是声明式的事务实现。spring事务采用AOP方式实现，在一个方法上添加了@Transactional注解后，**Spring会基于这个类生成一个代理对象，会将这个代理对象作为Bean**。***默认回滚异常是RuntimeException和Error***。

事务执行时先执行**TransactionAspectSupport.invokeWithinTransaction()**方法，在这个方法中：

- 1）获取事务属性(@Transactional注解的参数)
  - rollbackFor指定回滚哪种异常
  - noRollbackFor指定不回滚哪种异常
  - propagation指定传播行为
  - readOnly是否只读
  - isolation隔离级别
  - timeout超时时间

- 2）加载配置中的TransactionManager
- 3）获取收集事务信息TransactionInfo
- 4）**执行目标方法**（即添加注解的方法）
- 5）出现异常，尝试处理
- 6）清理事务相关信息
- 7）提交事务

## spring事务隔离级别

**spring事务隔离级别就是数据库的隔离级别**，外加一个默认级别（**DEFAULT 这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别**）

- read uncommitted（未提交读）
- read committed（提交读、不可重复度）
- repeatable read（可重复度）
- serializable（可串行化）

> Mysql数据库默认隔离级别是rr，spring默认隔离级别
>
> **以spring隔离级别为准**，如果spring设置的隔离级别数据库不支持，效果取决于数据库



# spring

## spring的bean在什么情况下不使用单例

**有状态的bean不用单例**。即有属性会被修改的bean。



## spring循环依赖

[参考](https://mrbird.cc/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Spring%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.html)

所谓循环依赖是指：BeanA对象的创建依赖于BeanB，BeanB对象的创建也依赖于BeanA，造成了死循环。

spring通过**提前曝光机制**，**利用三级缓存**解决循环依赖问题。

常见的两种循环依赖情况：

- 普通bean与普通bean之间的循环依赖
- 普通bean与代理bean之间的循环依赖

### bean创建过程

IOC容器获取bean的入口为**AbstractBeanFactory的getBean方法**，**具体逻辑在doGetBean方法内**：

> doGetBean方法中先通过getSingleton(String beanName)方法**从三级缓存中获取Bean实例**，如果不为空则进行后续处理；如果为空，则通过getSingleton(String beanName,ObjectFactory<?> singletonFactory)方法创建Bean实例并进行后续处理。
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
- earlySingletonObjects，二级缓存，key为Bean名称，value为Bean实例。这里的Bean实例指的是仅完成实例化的Bean，**还未进行属性填充等后续操作**。**用于提前曝光，供别的Bean引用，解决循环依赖**。
- singletonFactories，三级缓存，key为Bean名称，value为Bean工厂。**在Bean实例化后，属性填充之前，如果允许提前曝光，spring会把该Bean转换成Bean工厂并加入到三级缓存**。在需要引用提前曝光对象时再通过工厂对象的getObject()方法获取。

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

- 基于属性注入的情况，setter注入可以处理单例模式的循环依赖：a创建成功，并暴露objectFactory，同时b创建也能成功，b创建a所以b会去缓存中查找a并注入，a的原始bean经创建成功，所以可以set到b的属性，b也可以set到a的属性。

- prototype作用域的bean无法处理，因为**spring不缓存prototype作用域的bean**。



## spring的FactoryBean与BeanFactory

- BeanFactory：**Bean工厂，是一个工厂**，是ioc容器的最顶层接口，作用是**管理Bean**，即实例化、定位、配置应用程序中的对象及建立这些对象之间的依赖，ApplicationContext就是它的子类
- FactorBean：**工厂bean，是一个bean**，可以用于**自定义bean的创建过程**，这个接口有三个方法，getObject()、getObjectType()、isSingleton()用于获取bean对象，bean的类型，和是否单例，实现这个接口可以自定义bean的创建。spring中的bean包括普通bean和factorybean。在bean加载过程中会用到，单例已经开始初始化的时候通过getObject()方法初始化factorybean。

## BeanFactory和ApplicationContext区别

- BeanFactory，是spring中ioc容器的最顶层接口，是ioc的核心，拥有多种实现如：application、xmlbeanfactory
- ApplicationContext：**继承BeanFactory接口，高级的容器**。除了BeanFactory的初始化bean和获取bean功能，还提供了更多有用的功能，如：继承了MessageSource,支持国际化；**统一的资源文件访问方式**；**同时加载多个配置文件**；**载入多个（有继承关系）上下文，使每一个上下文都专注于一个特定的层次**，比如应用的web层等。
- **BeanFactory是延迟加载**，在使用的时候才会初始化Bean，而**ApplicationContext在启动时就默认加载了所有bean**（可以设置延迟）。
- 使用BeanFactory启动时占用资源少，如果bean配置的有问题ApplicationContext在启动时就可以发现配置的问题，同时bean加载之后系统运行较快。

## spring支持的几(6)种bean的作用域

- singleton，单例模式，默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护，该对象的生命周期与spring ioc容器一致。（第一次注入时才被创建）

- prototype，原型模式，**为每一个==bean请求==创建一个bean实例，在==每次注入时==都会创建一个新的对象**

- request，bean被定义为在**每个http请求中**创建一个单例对象，也就是在单个请求中都会复用这个bean实例

- session，与request范围类似，确保**每个session中都有一个bean实例**，在**session过期后，bean随之失效**

- application，bean被定义为在**ServletContext的生命周期**中复用一个单例对象

  > servletContext是一个域对象，是服务器在内存上创建的存储空间，用于在不同动态资源(servlet)之间传递与共享数据
  >
  > servlet是运行在服务器上的一个小程序，用来处理服务器请求

- websocket，bean被定义为在**websocket的生命周期**中复用一个单例对象

  > websocket，一种在单个TCP连接上进行双向通信的协议，保持一个长连接



## spring bean加载过程和生命周期

spring 通过容器存放应用中bean，并管理bean从创建到销毁的完整生命周期
spring bean的完整生命周期：

1. 对**bean实例化**，默认单例bean
2. 对bean进行**依赖注入，填充属性**，为属性或引用的bean赋值
3. 如果bean实现了BeanNameAware接口，将bean的id传递给setBeanName()方法
4. 如果bean实现了BeanFactoryAware接口，调用setBeanFactory()方法，将BeanFactory实例传入
5. 如果bean实现了ApplicationContextAware接口，调用setApplicationContext()接口，将bean所在应用上下文引用传入进来
6. 如果bean实现了BeanPostProcessor接口，调用它们的postProcessorBeforeInitialization()方法
7. 如果bean中有方法添加了@PostConstruct注解，那么该方法将被调用；
8. 如果实现了InitializingBean接口，spring将调用它们的afterPropertiesSet()方法，如果bean使用init-method声明了初始化方法，该方法也将被调用
9. 如果实现了BeanPostProcessor接口，spring将调用它们的postProcessorAfterInitialization()方法
10. 此时，bean准备就绪，可以使用，将一直停留在应用上下文中，直到上下文被销毁
11. 如果bean实现了DisposableBean接口，spring将调用它的destroy()接口，类似的，如果bean使用destroy-method声明了销毁方法，该方法也会被调用

**简单总结：**

bean实例化——》依赖注入，属性填充——》各种前置处理，初始化及各种后置处理——》准备就绪，停留在应用上下文中，直至上下文被销毁



## spring框架中的单例bean是线程安全的吗

**单例bean是非线程安全的**。

如果bean是有状态的，需要开发人员自己来进行线程安全的保证。**最简单办法是改变bean作用域，将singleton改变为prototype，每次请求bean时都创建一个新的bean实例**。

- 有状态就是有数据存储功能
- 无状态就是不会保存数据

**不要在bean中声明任何有状态的实例变量或成员变量，如果必须如此，那么就使用ThreadLocal把变量变为线程私有的**，如果bean的实例变量或类变量需要**在多个线程间共享，那么就要使用synchronized，lock，cas等实现线程同步**。

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
- @Lazy：标识bean是否需要延迟加载(第一次调用时才会加载)。可以加在类、方法、变量上。可以用于解决循环依赖问题

## spring中使用到的设计模式

单例模式、工厂模式、适配器模式(spring mvc的handlerAdapter)、代理模式(aop)、模板方法(父类定义了框架，特定实现通过子类)

## BeanFactoryPostProcessor与BeanPostProcessor

- BeanFactoryPostProcessor，**BeanFactory级别的处理，针对整个Bean工厂进行处理**。只有postProcessorBeanFactory一个方法，实现该接口，可以在spring的bean创建之前，修改bean的定义属性。spring允许BeanFactoryPostProcessor在容器实例化任何其他bean之前读取配置元数据，并可以根据需要进行修改。**BeanFactoryPostProcessor是在spring容器加载了bean的定义文件之后，在bean实例化之前执行的**。接口方法的入参是ConfigurableListableBeanFactory，使用该参数可以获取到相关bean的定义信息

- BeanPostProcessor，**bean级别的处理，针对某个具体bean进行处理**，接口提供了两个方法，分别是初始化前和初始化后执行方法，具体这个初始化方法指的是什么方法，类似我们在定义bean时，定义了init-method所指定的方法<bean id = "xxx" class = "xxx" init-method = "init()">这两个方法分别在init方法前后执行，需要注意一点，我们定义一个类实现了BeanPostProcessor，默认会对镇个别spring容器中所有的bean进行处理，可以看到方法中有两个参数，类型分别为Object和String，第一个参数是每个bean的实例，第二个参数是每个bean的name或id属性的值。所以我们可以用第二个参数，来确认我们将要处理的具体的bean。这个处理是发生在spring容器的实例化和依赖注入之后。在spring容器实例化bean之后，**在执行bean的初始化方法前后，添加一些自己的处理逻辑**。这里说的初始化方法，指下面两种：

  - bean实现了InitializingBean接口，对应的方法为afterPropertiesSet
  - 在bean定义的时候，通过init-method设置的方法

  **BeanPostProcessor是spring容器加载了bean的定义文件并且实例化bean之后执行的。BeanPostProcessor的执行顺序是在BeanFactoryPostProcessor之后**

# spring ioc

## 如何理解ioc

ioc-inversion of control，即**控制反转**，**不是什么技术，而是一种思想**。在java开发中，ioc意味着将你设计好的对象交给**容器**控制，而不是传统的直接在对象内部直接控制。

ioc container 管理的是spring bean，**spring里边的bean就类似是定义的一个组件，而这个组件的作用是实现某个功能**。

ioc和DI关系，控制反转是通过依赖注入实现的，即**ioc是设计思想，DI是实现方式**。

DI-Dependency Injection，即依赖注入，组件间的依赖关系由容器在运行期决定，即容器动态将某个依赖关系注入到组件中。

## spring ioc的初始化流程

spring ioc容器初始化过程，基本是整个spring启动的过程，大体流程：

spring启动 —》加载配置文件 —》将配置文件转化成Resource —》从Resource中解析转换成BeanDefinition —》主动或被动触发bean的初始化过程 —》应用程序中使用bean —》销毁bean —》容器关闭

初始化的入口在容器实现中的 refresh()调用来完成1，对bean定义载入ioc容器使用的方法是loadBeanDefinition（2-4）。

**简单总结spring ioc容器初始化流程**

1. Spring启动。
2. 加载配置文件，xml、JavaConfig、注解、其他形式等等，将描述我们自己定义的和Spring内置的定义的Bean加载进来
3. 加载完配置文件后，将配置文件转化成统一的**Resource**来处理
4. 使用Resource解析将我们定义的一些配置都转化成Spring内部的标识形式：**BeanDefinition**，容器解析得到 BeanDefinition 以后，需要把它在 IOC 容器中注册，这由 IOC 实现
5. 在低级的容器BeanFactory中，到这里就可以宣告Spring容器初始化完成了，**Bean的初始化是在我们使用Bean的时候触发的**；在**高级的容器ApplicationContext中**，会自动触发那些lazy-init=false的单例Bean，让Bean以及依赖的Bean进行初始化的流程，初始化Bean完成之后，高级容器也初始化完成了
6. 在应用中使用Bean
7. Spring容器关闭，销毁各个Bean



## spring依赖注入的方式

常用注入方式：**构造方法注入**，**setter注入**，**基于@Autowired注解的自动注入**

- 构造方法注入

```java
@Service
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    private final UserDaoImpl userDao;

    /**
     * init.
     * @param userDaoImpl user dao impl
     */
    @Autowired // 这里@Autowired也可以省略
    public UserServiceImpl(final UserDaoImpl userDaoImpl) {
        this.userDao = userDaoImpl;
    }

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return this.userDao.findUserList();
    }

}
```

- setter注入

```java
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    private UserDaoImpl userDao;

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return this.userDao.findUserList();
    }

    /**
     * set dao.
     *
     * @param userDao user dao
     */
    @Autowired
    public void setUserDao(UserDaoImpl userDao) {
        this.userDao = userDao;
    }
}
```

- 基于@Autowired注解的自动注入

```java
@Service
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    @Autowired
    private UserDaoImpl userDao;

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return userDao.findUserList();
    }

}
```

**spring推荐构造器注入方式**，这种方式**能够保证注入的组件不可变，并且确保需要的依赖不为空**。构造器注入的依赖总是能够在返回客户端代码时保证完全初始化的状态。**使用构造函数注入，如果存在循环依赖问题，spring项目启动报错**

- **依赖不可变**：指final关键字
- **依赖不为空**：当要实例化UserServiceImpl时，由于自己实现了**有参数的构造函数**，所以不会调用默认构造函数，就需要spring容器传入所需要的参数，当传入类型参数为null时会报错。
- **完全初始化的状态**：构造器传参之前，要确保注入的内容不为空，那么肯定要调用依赖组件的构造方法完成实例化。

**setter注入方法的缺点**：会有潜在空指针的风险

## @Autowired和@Resource以及@Inject等注解注入有何区别

1、@Autowired是Spring自带的，@Resource是JSR250规范实现的，@Inject是JSR330规范实现的

2、@Autowired、@Inject用法基本一样，不同的是@Inject没有required属性。

>  可以将@Autowired中required配置为false，如果配置为false之后，当没有找到相应bean的时候，系统不会抛异常

3、@Autowired、@Inject是默认按照类型匹配的，@Resource是按照名称匹配的

4、**@Autowired如果需要按照名称匹配需要和@Qualifier一起使用**，@Inject和@Named一起使用，@Resource则通过name进行指定

# spring aop

## spring aop的理解

aop，**Aspect-oriented Programming 面向切面编程**。

oop，**Object-oriented Programming 面向对象编程**。

当需要为多个不具有继承关系的对象引入同一个**公共行为**时（如：日志、权限校验、事务管理、性能监控），oop只能在每个对象里引用公共行为(方法)，这样程序中产生了大量重复代码。

AOP通过预编译和运行时动态代理，实现在不修改源代码的情况下，给程序添加统一的功能。

spring aop是一种编程范式，主要目的是将非功能性需求从功能性需求中分离出来，达到解耦目的。

## aop核心概念

- Aspect 切面，切面是一个关注点的模块化，这个关注点横跨多个类。比如：应用程序中的事务管理就是一个横切关注点。
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

## springboot的启动原理

- 通过启动类的main()方法进入，内部是SpringApplication.run()方法，在run()方法中先用启动类的class对象创建SpringApplication对像。

- 在**SpringApplication的构造函数**中，先判断启动类的class文件是否为null，接着判断应用是否为web应用，然后执行getSpringFactoriesInstances()方法：从spring-boot包下的“META-INF/spring.factories”路径加载classLoader，只取类型是ApplicationContextInitializer的ClassLoader，返回一个set（ClassLoader的类全限定名），然后加载这些类（初始化上下文）。然后以同样方式初始化监听。接着调用deduceMainApplicationClass()方法遍历栈信息，找到调用main方法所在的类，将其加载，赋值给变量mainApplicationClass，至此SpringApplication对象创建完成。
- 接着调用**run()方法**：加载监听器，从spring-boot包下“META-INF/spring.factories”路径下加载（SpringApplicationRunListener.class），然后调用starting方法，对之前加载的所有ApplicationListener根据事件类型进行广播事件（multicastEvent）。准备容器环境，会依据之前构造函数推断的应用类型返回对应的环境对象。创建容器上下文createApplicationContext()，也是依据之前的应用类型推断结果。准备容器上下文，首先将创建的context中的environment全部替换为springApplication中刚刚创建的environment，然后项context中注册beanfactory，接着所有的listener执行contextPrepared方法。之后向beanfactory中注册两个参数（spring启动类run方法的参数和banner），创建一个beanDefinitionLoader对象，该对象最主要支持了xml加载和annotated加载方式，加载所有source中的对象，这里只有一个主类，被@Component注解，将其注入到beanFactory中的beanDefinitionMap中。在refreshContext中会加载所有bean到beanFactory中。

简易总结：

1. 获取并启动监视器
2. 构造应用上下文环境
3. 初始化应用上下文
4. 刷新应用上下文前的准备阶段
5. 刷新应用上下文
6. 刷新应用上下文后的扩展接口

## springboot starter的原理

springboot starter是一个集成接合器，也叫做**场景启动器**。完成两件事：

1. 引入模块所需的相关jar包
2. 自动配置各自模块所需的属性（springboot按照某种默认的规则替我们完成自动配置的工作，这个规则就是**约定大于配置**）

springboot的自动配置在启动类的@SpringBootApplication注解中进行处理；

@SpringBootApplication注解是一个复合注解：

- 其中的**@EnableAutoConfiguration**注解就是开启自动配置；@EnableAutoConfiguration注解也是一个派生注解，其中关键功能由@Import提供，其导入的AutoConfigurationImportSelector的selectImports()方法通过SpringFactoriesLoader.loadFactoryNames()扫描所有具有META-INF/spring.factories的jar包。spring.factories文件也是一组组的key=value形式，其中一个key是EnableAutoConfiguration类的全类名，而它的value是一个xxxxAutoConfiguration的类名的列表，将这些**自动配置类加载到spring容器**中。

- 有一个**@EnableConfigurationPropoerties**注解，开启配置属性，而它后面的参数是一个ServerProperties类，这就是**约定大于配置的最终落地点**，在加载这些类的时候会读取配置文件中的配置赋值给对应属性。

自动配置生效条件：

- @ConditionalOnBean：当容器里有指定的bean的条件
- @ConditionalOnMissingBean：当容器里不存在指定bean的条件下
- @ConditionalOnClass：当类路径下有指定类的条件下
- @ConditionalOnMissingClass：当类路径下不存在指定类的条件下
- @ConditionalOnProperty：指定的属性是否有指定的值，比如@ConditionalOnProperties(prefix="xxx.xxx",value="enable",matchIfMissing=true)，代表当xxx.xxx为enable时条件的布尔值为true，如果没有设置的情况下也为true。

## springboot获取配置的方式

1. **@Value**注解
2. 通过注入环境变量Environment实现
3. 在启动类中获取springboot上下文，可以获取环境变量，也可以获取配置
4. 使用注解**@ConfigurationProperties(prefix="xxx")指定配置文件的前缀**，对成员变量生成get/set方法来获取，也可以配合@Value注解
5. **@PropertySource配合@Value获取非application配置文件的配置**（@PropertySource(value={"classpath:config/teacher.proerties"})）

## 引入非application配置文件

1. 在application.yml文件中引入（spring.profiles.active=dev可以引入application-dev.yml文件）
2. 系统启动时可以通过-spring.config.loca=xxx.properties环境变量指定配置文件
3. **@PropertySource(value={"classpath:config/teacher.proerties"})注解指定读取那个配置文件里的配置**，用在容器组件中如@Component注解修饰的类上

## springboot加载配置文件的目录及顺序

classpath:/,classpath:/config/,file:./,file:./config/

springboot加载两种配置文件的优先级

2.4.0之前版本，优先级properties>yaml

2.4.0之后，优先级yaml>properties

## springboot启动后控制台打印数据

1. 启动类的main方法后面添加打印的代码
2. ApplicationRunner、CommandLineRunner
3. @PostConstruct注解的方法
4. 如果bean使用了init-method属性声明了初始化方法，该方法也会被调用

## springboot热部署

1. 引入spring-boot-devtools依赖，添加org.springframework.boot.spring-boot-devtools:true配置
2. 引入springloaded依赖，在maven项目下，在pom.xml文件中的build中添加spring-loaded依赖，然后以cmd命令切换到项目文件目录下，以maven方式运行项目，命令：mvn spring-boot:run
3. 模板热部署，通过配置关闭模板的缓存
4. 付费的Jrebel工具



# spring MVC

## spring MVC的执行流程

processon https://processon.com/diagraming/62fa38790e3e7428ae35b130

![img](https://github.com/lission/markdownPics/blob/main/spring/spring%20MVC%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.jpg?raw=true)

1. 用户发送请求到**前端控制器DispatcherServlet**
2. DispatcherServlet收到请求调用**HandlerMapping处理映射器**
3. 处理映射器根据请求url找到具体的**Handler（后端控制器）**,生成处理器对象及处理器拦截器(如果有则生成)一并返回DispatcherSevlet
4. DispatcherServlet调用**HandlerAdapter处理器适配器**去调用Handler
5. 处理器适配器执行Handler
6. Handler执行完成给处理器适配器返回**ModelAndView**
7. 处理器适配器向前端控制器返回ModelAndView，ModelAndView是SpringMVC框架有一个底层对象，包括Model和View
8. 前端控制器请求**视图解析器**去进行视图解析，根据逻辑视图来解析真正的视图。
9. 视图解析器向前端控制器返回View
10. 前端控制器进行视图渲染，就是将模型数据(在ModelAndView对象中)填充到request域
11. 前端控制器向用户响应结果