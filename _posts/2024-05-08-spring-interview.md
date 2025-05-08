---
layout: post
title: "Spring Ask"
date: 2024-05-08 17:25:00 +0800
categories: [web-development]
tags: [java]
---

### 一、常见面试题？
#### 使用Spring框架的原因？
1. 轻量级，核心文件大小才约2MB
2. 支持IoC/DI：Spring的IoC容器实现了对Bean的全生命周期的管理，还可以通过DI实现依赖注入，从而实现对象依赖的松耦合
3. 支持AOP面向切面编程
4. 支持MVC，Spring MVC提供了功能更加强大且灵活的Web框架
5. 支持数据访问：Spring集成主流ORM框架
6. 声明式事务管理：Spring 通过AOP实现了事务的统一管理

#### Spring Ioc的工作流程
1. IoC就是控制反转，它的核心思想就是把对象的管理权限交给容器。如果程序需要使用某个对象实例，直接从IoC容器中获取即可，这样设计的好处就是降低了对象和对象之间的耦合性。
2. Spring中定义Bean的方式， Spring 在启动的时候，会解析这些Bean，然后保存在IoC中：
    1. XML中的<bean>标签
    2. 注解@Service @Component  @Repository 
    3. @Configuration  配置类中的@Bean  
3. Spring IoC的工作流程大致可以分为两个阶段
    1. <font style="color:#DF2A3F;">IoC容器的初始化</font>。这个阶段主要根据程序中的XML或者注解等Bean的定义，通过解析和加载生成BeanDefinition，然后把BeanDefinition注册到IoC容器（一个<font style="color:#DF2A3F;">ConcurrentHashMap</font>集合）
    2. <font style="color:#DF2A3F;">完成Bean初始化和依赖注入</font>。
        1. 通过反射针对没有设置lazy-init属性的单例Bean进行初始化
        2. 完成Bean的依赖注入
    3. <font style="color:#DF2A3F;">Bean的使用</font>。我们通过@Autowired  或者beanFactory.getBean() 从IoC容器中获取指定的Bean
4. 另外，针对<font style="color:#DF2A3F;">设置了lazy-init属性及非单例Bean的实例化是每次获取Bean的时候才初始化的，而且Spring IoC容器是不会管理这些Bean的！！</font>

#### Spring中BeanFactory和FactoryBean的区别是什么？
1. BeanFactory 是一个Factory，相当于IoC的一个顶级接口，访问Spring 容器的根接口，主要负责Bean的创建和访问。其中，最主要的方法getBean() 。同时，在BeanFactory中还会完成对Bean的依赖注入，也是DI
2. FactoryBean是一个特殊的Bean，这个Bean可以返回创建Bean的工厂。我们可以自定义FactoryBean，用它来扩展创建Bean的规则

#### 谈谈你对Spring Bean的理解
1. 在Spring中，Bean是最基本 的组成单元。一个由Spring IoC容器实例化、组装和管理的对象。
2. Spring是通过声明式配置的方式来定义Bean的，一般有3种方式：
    1. 基于XML
    2. 基于注解扫描的方式
    3. 基于元数据类的配置，例如@Configuration  和 @Bean  

#### Spring Bean的定义（配置属性）包含哪些内容？
1. Spring Bean的配置内容非常的多，而且这些属性都是要在Spring配置文件中声明的。Spring启动后，会映射到BeanDefinition的对象中。Spring Bean声明式配置和BeanDefinition的属性定义对照表，这里主要列举9个：

| 配置 | 对应属性 | |
| --- | --- | --- |
| class | beanClass | 必填项，指向一个具体存在的Java类，Spring容器创建的Bean就是这个Java类 |
| scope | scope | 定义作用域，默认Singleton |
| lazy-init | lazyInit | 定义是否延时加载，默认false |
| depends-on | dependsOn | 用于定义Bean实例化的依赖关系 |
| name | factoryBeanName | 定义Bean的唯一标识，如果没有定义，默认使用首字母小写的类名作为唯一标识 |
| constructor-arg | constructorArgumentValues | 构造方法 |
| properties | properties | 用于给Bean的指定属性赋默认值 |
| init-method | initMethodName | 指定Bean初始化完成后需要调用的回调方法 |
| destroy-method | destroyMethodName | 指定Bean销毁时需要调用的回调方法 |


#### Spring Bean的生命周期
1. Bean的声明周期大致分为5个阶段：
    1. 创建前准备阶段
        1. 主要在Bean开始加载之前，从相关配置中解析并查找Bean有关的配置内容。例如init-method()、destory-method()以及Bean加载过程中的前置和后置处理（BeanFactoryPostProcessor
    2. 创建实例阶段
        1. 主要通过反射来创建Bean的实例对象
    3. 依赖注入阶段
        1. 主要检测被实例化的Bean是否存在其他依赖并注入
        2. 触发一些扩展的调用，例如BeanPostProcessors（用来实现Bean初始化前后的回调）、InitializingBean类（有一个afterPropertieasSet（）用来给属性赋值）
    4. 容器缓存阶段
        1. 主要把Bean保存到IoC容器中缓存起来，这个阶段Bean可以被开发者正常使用
    5. 销毁实例阶段
        1. 在完成Spring上下文关闭时，销毁Spring上下文中所有的Bean
        2. 如果配置了destory-method属性，则这个阶段会被调用

#### Spring中Bean的作用域有哪些？
1. 在Spring配置中，我们可以通过scope属性来定义Spring Bean的作用域，共有5中类型
    1. singleton，用来定义一个单例Bean。Spring中的Bean默认都是单例，它的作用域范围是ApplicationContext
    2. prototype，用来定义一个Bean为多例，每次请求获取Bean时都会重新创建一个实例。它的作用域范围是调用getBean()方法直至获取到对象
    3. request，定义一个作用域范围仅在request中的Bean，每次发起HTTP请求时都会创建一个实例，仅在当前request中有效。它的作用域范围是每次发起HTTP请求直至拿到结果
    4. session，定义一个作用域范围仅在session中的Bean，仅在当前HTTP Session中有效。它的作用域范围是从浏览器首次访问到浏览器关闭
    5. globalSession，定义一个作用域范围仅在globalSession中的Bean，作用范围是整个WebApplicationContext
2. 作用范围从小到大依次是：prototype->request->session->globalSession->singleton

#### Spring 中的Bean是生命安全的吗？
1. 单例Bean是所有线程共享的一个实例，可能存在线程安全问题。
    1. 单例中的无状态Bean（不会修改成员变量的值）不存在安全问题
    2. 单例中的有状态Bean（会修改成员变量的值）存在安全问题
2. 多例Bean不存在线程安全问题

#### Spring有几种依赖注入的方式？
1. 通过构造器注入
    1. 官方推荐使用的方式
    2. 这种方式在注入对象很多的情况下，构造参数列表会很长。但是，好处就是可以检测循环依赖
2. 通过Setter方法注入
    1. 利用set方法来注入，每个set方法只能单独注入一个对象
    2. 当然也可以检测循环依赖
3. 通过Filed属性注入
    1. 例如@Autowired  @Resource   
    2. 如果不是被Spring托管的对象，则无法自动注入
    3. 无法检测循环依赖问题

#### Spring中使用了哪些设计模式
1. 待定？？？？

#### Spring如何解决循环依赖问题？
    1. ![](https://cdn.nlark.com/yuque/0/2025/png/35576929/1740040490169-09563306-124e-4144-8998-f71280cf1b56.png)
    2. 概念和原理
        1. Spring的三级缓存，都是使用ConcurrentHashMap存储
            1. <font style="color:rgb(44, 44, 54);">singletonObjects，一级缓存，存放已经完全初始化好的单例Bean实例</font>
            2. <font style="color:rgb(44, 44, 54);">earlySingletonObjects，二级缓存，存放早期暴露的Bean（尚未完成初始化的Bean）</font>
            3. <font style="color:rgb(44, 44, 54);">singletonFactories，存放Bean工厂对象，这些工厂可以用来创建提前暴露的Bean</font>
        2. 哪些Bean可能会进入三级缓存？
            1. Singleton作用域的Bean，如其他作用域（Prototype）不支持循环依赖解决方案
            2. 存在循环依赖的Singleton Bean
        3. 三级缓存的层级关系
            1. singletonFactories -> earlySingletonObjects -> singletonObjects，这种层次结构确保了 Bean 在不同阶段的状态可以被正确管理和转换
        4. **<font style="color:#DF2A3F;">Spring本身只能解决单实例存在的循环依赖问题</font>**。其他情况需要人为干预？
            1. 构造器注入导致的循环依赖，可以通过@Lazy  进行注解、改为 setter 方法注入或字段注入
            2. Prototype 作用域的 Bean 间的循环依赖，需要把Bean改为Singleton
            3. DependsOn 导致的循环依赖，找到注解循环依赖的地方，迫使它不循环依赖。备注：@DependsOn 可以：显式指定某些 Bean 必须在另一个 Bean 初始化之前被初始化，即指定Bean初始化顺序

#### Spring中事务的传播行为有哪些？Spring中事务的隔离有哪些？
1. Spring事务隔离级别
    1. ISOLATION_DEFAULT（默认）：使用后端数据库的默认隔离级别。不同的数据库可能有不同的默认值
    2. ISOLATION_READ_UNCOMMITTED（读未提交）：允许一个事务读取另一个事务未提交的数据。这是最低的隔离级别，可能导致脏读、不可重复读和幻读的问题
    3. ISOLATION_READ_COMMITTED（读已提交）：确保一个事务只能看到其他事务已经提交的数据。这可以防止脏读，但不能避免不可重复读和幻读
    4. ISOLATION_REPEATABLE_READ（可重复读）：保证在同一事务中多次读取同样的数据时结果一致，不受其他事务影响。此级别可以防止脏读和不可重复读，但理论上仍可能出现幻读
    5. ISOLATION_SERIALIZABLE（串行化）：最高的隔离级别，完全遵循ACID原则，确保事务被完全隔离，即每个事务都在独立的状态下执行，互不干扰。它可以防止所有并发问题（脏读、不可重复读和幻读），但是因为增加了大量的锁定机制，所以会极大地降低系统的并发性能
2. Spring中事务传播行为
    1. PROPAGATION_REQUIRED，Spring中默认的事务传播行为。表示如果当前存在事务，则加入这个事务，如果不存在，则新建一个事务。
        1. 例如，执行serviceA.methodA()的时候Spring已经起了事务，这时候调serviceB.methodB()，看到自己已经运行在serviceA.methodA()内部事务，就不再起新的事务；如果没有事务则会自己创建新事务
        2. 在serviceA.methodA()和serviceB.methodB()任何地方异常，事务都会被回滚
    2. PROPAGATION_REQUIRE_NEW，表示不管是否存在事务，都会创建一个新事务，新建的事务和原来已经定义的事务相互独立。
        1. 即外部事务的异常回滚，不会影响内部事务的正常提交
    3. PROPAGATION_NESTED，如果当前存在事务，则嵌套在当前事务中执行。如果当前没有事务，则新建一个事务。
        1. 例如执行serviceA.methodA()，调用执行serviceB.methodB()。
        2. 假设执行serviceB.methodB()执行失败，已经回滚到执行之前的SavePoint，所以不会产生脏数据。
        3. 外部事物即serviceA.methodA()可以自己决定commit或者rollback，并不会对serviceA.methodA()产生影响
    4. PROPAGATION_SUPPORTS
        1. 表示支持当前事务，如果没有当前事务，就以非事务方式执行
    5. PROPAGATION_NOT_SUPPORTED
        1. 表示以非事务的方式执行，如果当前存在事务，则把当前事务挂起
    6. PROPAGATION_MANDATORY
        1. 强制事务执行，如果当前不存在事务，则抛异常
    7. PROPAGATION_NEVER
        1. 以非事务执行，如果当前存在事务，则抛异常

#### 导致Spring事务失效的原因有哪些？
1. 主要有以下几个原因：
    1. 方法没有被public修饰，如果@Transactional 事务注解添加在不是public修饰的方法上，事务会失效
    2. 类没有被Spring托管
    3. 不正确的异常捕获，如果事务方法抛出的异常被catch处理了，则@Transactional将无法回滚导致事务失效
    4. 在同一个类里面，A方法调用B方法，如果A没有添加事务，B添加了@Transactional事务，B事务会失效
    5. rollbackFor参数配置错误，标注了错误的异常类型。
        1. 默认情况下，Spring 的事务管理只会在遇到运行时异常（继承自 RuntimeException）和错误（Error）时回滚事务。如果抛出的是受检异常（即继承自 Exception 但不包括 RuntimeException），则不会触发事务回滚。
        2. 如果需要抛出受检异常，需要在rollbackFor特别指定

#### Spring AOP的原理和理解
1. 执行过程，大概分为4个阶段
    1. 创建代理对象阶段
        1. 首先检查目标类是否满足切面规则，满足的话就调用ProxyFactory创建代理Bean并缓存到IoC容器
        2. 如果目标类实现了接口，Spring会默认选择JDK Proxy
        3. 如果没有实现接口，则默认选择Cglib Proxy
    2. 拦截目标对象阶段
        1. 当调用目标对象的某个方法时，会被AopProxy的对象拦截，并形成一条由MethodInterceptot元素组成的拦截链
    3. 调用代理对象阶段
        1. 执行拦截链中的被织入代码片段
    4. 调用目标对象阶段
        1. 执行MethodInterceptot接口中的invoke()方法，触发目标对象方法的调用
2. 图解
    1. 略
3. 概念
    1. 切面（Aspect）：包含通知（Advice）和切入点（Pointcut）。
        1. 通知定义了需要插入的额外逻辑
        2. 切入点则定义了这些逻辑应该在哪种情况下被应用（例如，在哪些方法调用前、后或周围）。
    2. 通知类型
        1. 前置通知：在目标方法执行之前执行
        2. 后置通知：在目标方法成功执行之后执行
        3. 异常通知：在目标方法抛出异常之后执行
        4. 最终通知：无论方法是否成功完成或抛出异常，都会执行
        5. 环绕通知：包裹目标方法的调用，可以在方法调用前后执行自定义行为

#### 谈谈你对Spring MVC的理解
1. Spring MVC的具体工作流程![](https://cdn.nlark.com/yuque/0/2025/png/35576929/1740143626871-2e108b7f-13fd-4fbe-8430-d88310d77aea.png)
2. 流程如上

#### 简述Spring MVC的核心执行流程
#### Spring MVC 中9大核心组件
#### Spring 中@Autowire 和 @Resource 的区别
1. 注解内部的定义不同
    1. @Autowired  只包含一个required参数，默认为true，表示开启自动注入。如果不开启自动装配时，可设置为false
    2. @Resource  有一个name和type参数
2. 装配方式的默认值不同
    1. @Autowired  默认按type自动装配
    2. @Resource  默认按照那么装配。可以自定义装配方式，指定name，则按照name自动装配。指定type，则按照type
3. 应用范围不同
    1. @Autowired  能够在构造方法、成员变量、方法参数及注解上
    2. @Resource  能在类、成员变量和方法上
4. 出处不同
    1. @Autowired  在Spring中定义
    2. @Resource  在JDK中定义
5. 







