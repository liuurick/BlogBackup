---
title: Spring知识点梳理
date: 2019-10-23 11:14:51
tags: Spring
categories: [后端,Spring家族,Spring]
---

Spring

- IOC
- AOP

<!--more-->

# IOC

IOC是Spring Core最核心的部分。更确切的说是一种思想，将我们从繁琐的对象交互中解脱，进而专注于对象本身。

## 1 IOC原理

IOC（Inversion of Control）：控制反转

通过一个例子了解IOC：**设计行李箱**

![image](/images/2019102301.png)

行李箱依赖于箱体，箱体依赖于底盘，底盘依赖于轮子，这样的设计非常不合理，耦合度较高，当需要修改轮子的尺寸的时候，那么间接导致底盘，箱体等信息都需要修改。



转换为代码设计：

**上层建筑依赖下层建筑**

![image](/images/2019102302.png)

```java
Luggage luggage = new Luggage();
luggage.move();
```

当需要修改轮子尺寸时，会暴露出很多问题。

![image](/images/2019102303.png)

```java
int size = 30;
Luggage luggage = new Luggage();
luggage.move();
```

可见牵一发而动全身，这种代码设计是不可维护的。

### 1.1 依赖注入

> 含义：把底层类作为参数传递给上层类，实现上层对下层的“控制”

行李箱设计引入依赖注入

![image](/images/2019102304.png)

此时修改轮子信息

![image](/images/2019102305.png)

```java
int size = 30;
Tire tire = new Tire(size);
Bottom bottom = Bottom(tire);
Framework framework = new Framework(bottom);
Luggage luggage = new Luggege(framework);
luggage.move();
```



### 1.2 依赖注入的方式

IOC，DI，DL的关系

![image](/images/2019102306.png)



依赖注入（Dependency Injection）方式，主流使用

1. 构造函数（Constructor），实现特定参数的构造函数，在创建对象时将底层类作为参数传递给上层类，让IOC容器注入所依赖的类型的对象
2. Set注入（Setter），实现特定属性的public set()方法来让IOC容器调用注入所依赖的类型的对象
3. 接口注入（Interface），实现特定的接口以供IOC容器注入所依赖的类型的对象
4. 注解注入（Annotation），通过java的注解机制，让IOC容器注入所依赖的类型的对象。例如，Sprint框架里的AutoWired这一类的标签，能够实现注解功能。
5. Dependency Lookup方式，被放弃使用

### 1.3 IOC底层原理

IOC思想离不开依赖注入（DI）方法，基于IOC思想提出了**IOC容器**。IOC容器**管理bean的生命周期，控制bean的依赖注入。**

![image](/images/2019102307.png)



### 1.4 IOC容器的优势

- 使用依赖注入会进行大量new操作，IOC容器可以自动对代码进行初始化，只需维护一个config文件，例如xml文件即可。避免在各处使用new来创建类，并且可以做到统一维护依赖关系
- 创建实例时不需要了解其细节，我们只需请求一个对象（很多类作为该对象类的底层）的实例，IOC容器按照Config文件创建一个该对象的实例。

## 2 Spring IOC的应用

1. Spring启动时，读取应用程序提供的Bean配置信息，并在Spring容器中生成一份Bean的注册表
2. 根据注册表实例化Bean，Spring提供一个配置文件描述Bean和Bean之间的依赖关系，利用Java的反射功能实例化Bean。
3. 将Bean实例放到Spring容器的Bean缓存池中
4. 使用Bean，装配好Bean之间的依赖关系，为上层提供准备就绪的运行环境。

![image](/images/2019102308.png)



### 2.1 Spring IOC支持的功能

- **依赖注入**

  外部注入bean的三种方式：

  - set()方法
  - 构造器
  - 属性反射（AutoWired）

- **自动装配**

- 依赖检查

- 支持集合

- 指定初始化方法和销毁方法

- 支持回调某些方法，需要实现Spring接口，带有侵入性，应谨慎使用

### 2.2 Spring IOC容器的接口

**核心接口（IOC容器的两种实现方式）**：

- **BeanFactory**：Spring框架最核心的接口

  - 提供IOC的配置机制---》加载配置文件时，不会创建对象，只有在使用对象时才会创建（懒加载机制）
  - 包含Bean的各种定义，便于实例化Bean
  - 建立Bean之间的依赖关系
  - Bean生命周期的控制
  - 主要方法
    - getBean()，从Spring容器中获取Bean
    - isSingleton()，Bean在容器中是否是单例，默认是单例

  

  BeanFactory体系结构：

  ![image](/images/2019102309.png)

- ApplicationContext

  - 加载配置文件时，对象就会创建
  - 继承了BeanFactory：能够管理、装配Bean
  - 继承了ResourcePatternResolver：能够加载资源文件
  - 继承了MessageSource：能够实现国际化等功能
  - 继承了ApplicationEventPublisher：能够注册监听器，实现监听机制

- BeanFactory与ApplicationContext的比较

  - BeanFactory是Spring框架的基础设施，面向Spring
  - ApplicationContext面向使用Spring框架的开发者

**主要接口**：

- **BeanDefinition**，主要描述Bean的定义
- **BeanDefinitionRegistry**，提供向IOC容器注册BeanDefinition对象的方法



## 3 Spring IOC的refresh源码解析

> Spring IOC 容器源码分析：https://javadoop.com/post/spring-ioc

## 4 Spring IOC的getBean方法的解析

### 4.1 IOC操作Bean管理

**什么是Bean管理？**Bean管理指的是两个操作

- Spring创建对象
- Spring注入属性



**Bean管理操作两种实现方式**

- 基于注解方式

- 基于xml配置文件方式，在Spring配置文件中，使用bean标签，标签里面添加对应的属性，就可以实现对象创建、属性注入。

  1.创建对象

  - bean标签的常用属性：
    - id，对象起的唯一别名、标识
    - class，类全路径
    - property，属性
  - 创建对象时，默认调用的是无参构造器

  2.注入属性

  - 使用set()方法进行注入，需要在类中实现set()方法

    ```xml
    <bean id="user" class="com.mm.spring5.User">
    <property name="userName" value="mm"></property>
    </bean>
    ```

  - 使用有参数构造器注入

    ```xml
    <bean id="order" class="com.mm.spring5.Order">
    <constructor-arg name="oName" value="abc"></constructor-arg>
    <constructor-arg name="address" value="china"></constructor-arg>
    </bean>
    ```

  3.xml注入其他类型属性

  - null值

    ```xml
    <property name="content">
                <null/>
    </property>
    ```

  4.注入外部bean

  ```xml
  <bean id="userService" class="com.mm.spring5.service.UserService">
      <property name="userDao" ref="userDaoImpl"></property>
  </bean>
  <bean id="userDaoImpl" class="com.mm.spring5.dao.UserDaoImpl"></bean>
  ```

### 4.2 getBean方法的代码逻辑

- 转换beanName
- 从缓存中加载实例
- 实例化Bean
- 检测parentBeanFactory
- 初始化依赖的Bean
- 创建Bean

### 4.3 Spring Bean的作用域

- singleton，Spring的默认作用域，容器里拥有唯一的Bean实例
- protitype，针对每个getBean请求，容器都会创建一个Bean实例
- request，会为每个Http请求创建一个Bean实例
- session，会为每个session创建一个Bean实例
- globalSession，会为每个全局Http Session创建一个Bean实例，该作用域仅对Portlet有效

### 4.4 Spring Bean的生命周期

由SpringIOC容器进行管理

- Bean的创建

  - 实例化Bean对象，在BeanDefinition类中设置Bean属性

    （通过ware注入Bean对容器基础设施层面的依赖，对实例化完成的Bean添加自定义的处理逻辑）

  - 使用BeanDefinitionRegistry注册BeanDefinition，建立映射

  - 调用定制的Bean init方法，进行初始化相关的操作

  - Bean实例初始化之后的自定义工作

  - Bean初始化完毕

  ![image](/images/2019102310.png)

- Bean的运行

  - 在应用程序中使用通过getBean()方法获取并使用Bean

- Bean的销毁

  - 若实现了DisposableBean接口，则会调用destory方法
  - 若配置了destory-method属性，则会调用其配置的销毁方法



## 5 Spring实例

目的：

- Bean如何装载到IOC容器中
- 如何从IOC容器获取Bean实例、使用Bean实例
- 分析ICO容器的创建配置以及Bean如何获取的底层原理

方法：

- 使用@Bean装配Bean，效率不高
  - 通过run()获取ApplicationContext的实例ctx
  - 调用ctx的getBean()方法获取Bean
- 使用SpringBoot方式，扫描并装配Bean到IOC容器，在装载类上加上@Componet，表明哪个类需要被扫描到SpringIOC容器中
  - 类里面的成员变量若为类对象，需要使用@Autowired进行注释，根据属性类型找到对应的Bean进行注入。注入机制最重要的一条就是根据类型，通过BeanFactory接口，IOC容器通过getBean()方法，支持根据类型、名称来获取Bean。



# AOP

> AOP（Aspect Oriented Programming），面向切面编程。
>
> 思想：
>
> **关注点分离，不同的问题交给不同的部分去解决**
>
> - 通用化功能代码的实现，对应的就是所谓的切面（Aspect）
> - 业务功能代码和切面代码分开，架构将变得高内聚低耦合
> - 确保功能的完整性，切面最终需要被合并（织入）到业务中（Weave）
>
> AOP总体比较简单易用，处理好www问题即可---what(切面)，where(切面的织入发生在哪里)，when

> Spring AOP 使用介绍，从前世到今生:https://javadoop.com/post/spring-aop-intro

## AOP的三种织入方式

- 编译时织入：需要特殊的Java编译器，如AspectJ
- 类加载时织入：需要特殊的Java编译器，如AspectJ和AspectWerkz
- 运行时织入：Spring采用的方式，通过动态代理的方式，实现简单



## AOP的主要名称概念

- Aspect，通用功能的代码实现（例如日志切面，就是一个普通的java类实现日志记录功能 ）
- Target，目标对象，即被织入Aspect的对象（）
- Join Point，可以作为切入点的机会，所有的方法都的执行处可以作为切入点（例如HelloWorld的请求方法）
- Pointcut，Aspect实际被应用在Join Point，支持正则。Pointcut定义Join Point应该切入到哪个地方。
- Weaving，AOP的实现过程
  - 编译时织入
  - 类加载时织入
  - 运行时织入，Spring采用的方式，通过动态代理的方式，实现简单
- Advice的种类，类里面的方法以及这个方法如何织入到目标方法的方式
  - 前置通知（Before）
  - 后置通知（AfterReturning）
  - 异常通知（AfterThrowing）
  - 最终通知（After）
  - 环绕通知（Around）

## AOP的实现

代理模式：接口+真实实现类+代理类

> AOP的实现：JdkProxy和Cglib
>
> - 由AopProxyFactory根据AdvisedSupport对象的配置来决定
> - 默认策略如何目标类是接口，则用JdkProxy来实现，否则用后者
> - JdkProxy的核心
>   - InvocationHandler接口
>   - Proxy类
> - Cglib：以继承的方式动态生成目标类的代理
>
> JDKProxy: 通过Java的内部放射机制实现
>
> Cglib: 借助ASM实现
>
> 反射机制在生成类的过程中比较高效
>
> ASM在生成类之后的执行过程中比较高效



## 实例之日志记录

- 目的：记录访问的ip、时间、返回值等
- 流程：
  - What，将记录日志的功能代码分离出来，做成一个切面，在切面中解决What的内容，即实现日志功能（Aspect）
  - Where，Target目标对象，需要使用该切面功能的类对象。Pointcut可使用正则确定切入点，什么类型的方法可以作为切入点。
  - When，确定什么时候进行织入，即调用目标对象中的方法之前还是之后还是其他时候进行



## Spring AOP的原理

> Spring AOP 源码解析:https://javadoop.com/post/spring-aop-source

## Spring事务的相关考点

- ACID
- 隔离级别
- 事务传播

