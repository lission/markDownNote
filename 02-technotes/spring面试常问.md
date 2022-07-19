[TOC]

# 1、spring 如何创建bean对象

bean的创建过程:

![!img](https://raw.githubusercontent.com/lission/markdownPics/main/spring/bean%E7%9A%84%E5%88%9B%E5%BB%BA%E6%B5%81%E7%A8%8B.webp)

类————》无参构造方法————》对象————》依赖注入（属性赋值）————》放入Map单例池，初始前————》初始化————》初始后————》Bean对象

- 利用该类的无参构造方法实例化得到一个对象（如果一个类中有多个构造方法，spring会进行选择进行推断构造方法）
- 得到一个对象后，spring判断该对象中是否存在被@Autowired注解了的属性，把这些属性找出来并由spring进行赋值(依赖注入)
- 依赖注入后，spring判断该对象是否实现了BeanNameAware接口、BeanClassLoaderAware接口、BeanFactoryAware接口，如果实现了，表示当前对象必须实现该接口中的setBeanName()、
setBeanClassLoader()、setBeanFactory()方法，spring调用这些方法并传入相应参数
- Aware回调后，spring会判断该对象中是否存在某个方法被@PostConstruct注解了，如果存在，spring会调用当前对象此方法（初始化前）
- 然后，spring会判断该对象是否实现了initializingBean接口，如果实现了，表示当前对象必须实现该接口中afterPropertiesSet()方法，spring就会调用
当前对象中的afterPropertiesSet()方法（初始化）
- 最后，Spring会判断当前对象需不需要进行AOP，如果不需要那么Bean就创建完了，如果需要进行AOP，会进行动态代理并生成一个对象作为Bean（初始化后）

如果当前Bean是单例Bean，那么会把该Bean对象存入一个Map<String, Object>，Map的key为beanName，value为Bean对象。这样下次getBean时就可 以直接从Map中拿到对应的Bean对象了。（实际上，在Spring源码中，这个Map就 是单例池）
如果当前Bean是原型Bean，那么后续没有其他动作，不会存入一个Map，下次 getBean时会再次执行上述创建过程，得到一个新的Bean对象。


**推断构造方法**：
- 如果一个类只有一个构造方法，那么没得选择，只能用这个构造方法。但是有参的构造方法，参数必须是spring的Bean这样spring才能拿到进行赋值。
- 如果一个类存在多个构造方法，Spring不知道如何选择，就会看是否有无参的构造方法，因为无参构造方法本身表示了一种默认的构造方法。
- 如果都没有构造方法，就是用默认的无参构造方法来创建。


# 2、什么是单例池
Map<beanName,Bean> 单例池， ConcurrentHashMap

# 3、Bean对象和普通对象之间有什么区别

