---
title: Java高效编程总结
date: 2021-05-03 16:22:20
tags: 高效编程
categories: 高效编程
---

[TOC]

<!--more-->

# 1 函数式编程

你是否见过“撸”代码也可以像撸串儿一样？那么我们从函数式编程说起，通过学习本单元，使小伙伴们了解Lambda出现的意义和价值；会灵活使用；知晓使用时应注意的问题；明白其内部实现原理。使小伙伴们从“监听方法地狱”中解放出来。...

收起列表

- *视频：*2-1 撸串式编程开场白 (02:30)
- *视频：*2-2 实战：购物车案例 (13:27)
- *视频：*2-3 实战：硬编码业务逻辑 (07:26)
- *视频：*2-4 实战：单一维度条件参数化 (04:25)
- *视频：*2-5 实战：多维度条件参数化 (09:10)
- *视频：*2-6 实战：判断逻辑参数化-实体类 (12:42)
- *视频：*2-7 实战：判断逻辑参数化-匿名类 (02:48)
- *视频：*2-8 实战：判断逻辑参数化-Lambda表达式 (03:10)
- *视频：*2-9 Lambda表达式 (06:23)
- *视频：*2-10 实战：自定义函数式接口 (10:38)
- *视频：*2-11 常用函数接口与方法引用 (06:36)
- *视频：*2-12 总结乃成功她亲爹 (03:50)**试看**
- *视频：*2-13 方法引用精讲 (22:55)

# 2 流式编程
代码简洁之道

  你是否也曾听过成熟的大牛们自诩“代码洁癖”，本章让你体验什么是【大道至简】？本章重点讲解Stream的三类操作，Stream的并行执行的特性和一些需要注意的问题点。通过学习本单元，使小伙伴能够利用Stream【化繁为简】，简化集合的繁琐操作。...

  收起列表

  - *视频：*3-1 流式编程开场与案例场景概述 (01:05)
  - *视频：*3-2 实战：传统方式处理业务逻辑 (12:16)
  - *视频：*3-3 实战：利用Lambda+Stream处理业务逻辑 (07:58)
  - *视频：*3-4 实战案例归纳总结 (00:32)
  - *视频：*3-5 流的初体验 (05:41)
  - *视频：*3-6 流操作分类 (02:29)
  - *视频：*3-7 实战：常用中间操作演示之过滤/映射/扁平化 (08:48)
  - *视频：*3-8 实战：常用中间操作演示之遍历/排序 (06:11)
  - *视频：*3-9 实战：常用中间操作演示之去重/跳过/截断 (05:03)
  - *视频：*3-10 实战：常用中间操作总结 (03:09)
  - *视频：*3-11 实战：常用终端操作演示之匹配 (06:50)
  - *视频：*3-12 实战：常用终端操作演示之查找 (03:53)
  - *视频：*3-13 实战：常用终端操作演示之最大/最小/计数 (03:42)
  - *视频：*3-14 常用操作总结与流构建描述 (00:51)
  - *视频：*3-15 实战：流的构建四种形式 (08:31)
  - *视频：*3-16 收集器与预定义收集器概述 (01:51)
  - *视频：*3-17 实战案例预定义收集器 (08:21)
  - *视频：*3-18 总结乃成功她祖奶奶 (04:26)
  - *图文：*3-19 补充教辅文档_使用SQL检索集合数据
  - *视频：*3-20 归约操作原理讲解 (04:43)
  - *视频：*3-21 归约操作实战案例 (16:26)
  - *视频：*3-22 汇总操作原理讲解 (02:40)
  - *视频：*3-23 汇总操作实战案例 (12:27)
  - *视频：*3-24 收集器接口讲解 (04:23)
  - *视频：*3-25 实战案例一：查找 (12:31)
  - *视频：*3-26 实战案例二：去重 (07:04)
  - *视频：*3-27 实战案例三：扁平化 (06:49)
  - *视频：*3-28 实战案例四：分组 (07:52)
  - *视频：*3-29 实战案例五：排序 (10:40)
  - *视频：*3-30 实战案例后会有期 (00:21)

# 3 资源关闭

  对比JDK7之前关闭资源和使用JDK7特性try-with-resource关闭资源，凸显处理资源关闭上便捷性；介绍相关技术点，如AutoCloseable；分析TWS的实现原理；利用这一特性，实现扩展功能。通过学习本单元，使小伙伴们能够利用此特性优雅的关闭资源。...

  收起列表

  - *视频：*4-1 普通码农与风骚码农的资源关闭PK (04:28)
  - *视频：*4-2 垃圾回收与物理资源释放 (02:20)
  - *视频：*4-3 实战：传统方式关闭流资源 (09:09)
  - *视频：*4-4 实战：TWR方式关闭流资源 (06:13)
  - *视频：*4-5 实战：TWR进阶与特殊情况 (06:25)
  - *视频：*4-6 总结乃成功她亲孙子 (03:19)



## 3.1 垃圾回收（GC）的特点

- 垃圾回收机制只负责回收堆内存资源，不会回收任何物理资源
- 程序无法精确控制垃圾回收动作的具体发生时间

- 在垃圾回收之前，总会先调用它的finalize方法



## 3.2 常见需手动释放的物理资源

- 文件/流资源
- 套接字资源

- 数据库连接资源



## 3.3 物理资源可以不手动释放吗？

- 资源被长时间无效占用
- 超过最大限制后，将无资源可用

- 导致系统无法正常运行



## 3.4 传统方式关闭流资源

```java
/**
 * JDK7之前的文件拷贝功能
 */
public class FileCopyTest {

    @Test
    public void copyFile() {
        /**
         * 1. 创建输入/输出流
         * 2. 执行文件拷贝，读取文件内容，写入到另一个文件中
         * 3. **关闭文件流资源**
         */

        // 定义输入路径和输出路径
        String originalUrl = "lib/FileCopyTest.java";
        String targetUrl = "targetTest/target.txt";

        // 声明文件输入流，文件输出流
        FileInputStream originalFileInputStream = null;
        FileOutputStream targetFileOutputStream = null;

        try {
            // 实例化文件流对象
            originalFileInputStream =
                    new FileInputStream(originalUrl);

            targetFileOutputStream =
                    new FileOutputStream(targetUrl);

            // 读取的字节信息
            int content;

            // 迭代，读取/写入字节
            while ((content = originalFileInputStream.read()) != -1) {
                targetFileOutputStream.write(content);
            }

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {

            // 关闭流资源
            if (targetFileOutputStream != null) {
                try {
                    targetFileOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (originalFileInputStream != null) {
                try {
                    originalFileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

}
```

## 3.5 TWR方式关闭流资源

```
public class NewFileCopyTest {

    @Test
    public void copyFile() {

        // 先定义输入/输出路径
        String originalUrl = "lib/NewFileCopyTest.java";
        String targetUrl = "targetTest/new.txt";

        // 初始化输入/输出流对象
        try (
                FileInputStream originalFileInputStream =
                        new FileInputStream(originalUrl);

                FileOutputStream targetFileOutputStream =
                        new FileOutputStream(targetUrl);
        ) {

            int content;

            // 迭代，拷贝数据
            while ((content = originalFileInputStream.read()) != -1) {
                targetFileOutputStream.write(content);
            }

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

}
```

## 3.6 TWR简介

try-with-resource

- Java7引入新特性
- 优雅关闭资源

- 一种java语法糖



## 3.7 TWR使用

- 多资源自动关闭
- 实现AutoCloseable接口

- 避免异常屏蔽



## 3.8 资源关闭顺序问题

- 先开后关原则
- 从外到内原则

- 底层资源原则声明



## 3.9 资源关闭特殊情况

- 资源对象被return的情况，由调用方关闭
- ByteArrayInputStream等不需要检查关闭的资源对象

- 使用Socket获取InputStream和OutputStream对象不需要关闭

# 4 工具集
## 4.1 Guava简介

背靠Google好乘凉，Guava工程包含了若干被Google的Java项目广泛依赖的核心库，例如：集合，缓存，原生类型支持，并发库，通用注解，字符串处理，I/O等等。

所有这些工具每天都在被Google的工程师应用在产品中。

## 4.2 使用和避免null

大多数情况下，使用null表明的是某种缺失情况。

Guava引入`Optional<T>`表明可能为null的T类型引用。Optional实例可能包含非null的引用（引用存在），也可能什么也不包括（引用缺失）。

正是受到Guava的启发，Java8将Optional类作为一个新特性引入到Java8的类库。



## 4.3 实战案例：Java8新特性Optional



## 4.4 不可变集合

创建对象的不可变拷贝是一项很好的防御性编程技巧

Guava为所有的JDK标准集合类型和Guava新集合类型都提供了简单易用的不可变版本。



不可变对象的优点

![img](https://cdn.nlark.com/yuque/0/2021/png/1214202/1619749492738-a922d6de-f917-40cf-9d5d-283b53c40e3b.png)



## 4.5 新集合类型



## 4.6 实战：新集合类型使用



## 4.7 集合工具类

## 4.8 实战：IO流工具类的使用



## 4.9 总结



# 5 线程池

## 5.1 线程池简介

线程池顾名思义就是事先创建若干个可执行的线程放入一个池（容器）中，需要的时候从池中获取线程不用自行创建，使用完毕不需要销毁线程而是放回池中，从而减少创建和销毁线程对象的开销。

优点：

- 降低资源消耗
- 提高响应速度
- 提高线程的可管理性



5.2 简单线程池的设计 (05:58)

*视频：*6-4 线程池参数与处理流程 (04:00)

*视频：*6-5 线程池可选择的阻塞队列 (14:09)

*视频：*6-6 线程池可选择的饱和策略 (03:57)

*视频：*6-7 线程池的执行示意图 (01:27)

*视频：*6-8 替换6-8~9 线程池可选饱和策略与执行示意图 (07:55)

*视频：*6-9 常用线程池 (02:38)

*视频：*6-10 向线程池提交任务 (08:41)

*视频：*6-11 线程池的状态 (02:20)

*视频：*6-12 总结乃成功她丈母娘 (03:34)

*视频：*6-13 线程池饱和策略之终止策略 (18:43)

*视频：*6-14 线程池饱和策略之其他三种饱和策略 (10:47)



# 6 实用工具

## 6.1 Lombok简介

> `Project Lombok`是一个Java库，可以自动插入编辑器并构建工具，为Java增添色彩。永远不要再写另一个getter或equals方法，使用一个注释，您的类具有一个功能齐全的构建器，自动化您的日志记录变量。
>
> ​																																																			--Lombok官网

## 6.2 Lombok实现原理

注解的两种解析方式

- 运行时解析---类似于反射
- 编译时解析---Lombok

编译时解析的两种机制

- Annotation Processing Tool（注解处理器）----JDK1.8去除
- Pluggable Annotation Processing API（JSR269插入式注解处理器）

![image](/images/2020042401.png)

## 6.3 Lombok插件安装

> IDEA-->setting-->plugins-->搜索lombok安装即可



## 6.4 Lombok常用注解使用

> 实战：演示Lombok注解使用方式，并通过查看编译后class文件，理解其工作原理
>
> https://www.cnblogs.com/ooo0/p/12448096.html
>
> https://www.cnblogs.com/muxi0407/p/11611885.html

![image](/images/2020042402.png)

1.@Data注解

一种大而全的注解：包含@Getter，@Setter，@ToString，@EqualsAndHashCode，@NoArgsConstructor



2.@Val注解

弱语言变量，作用范围是本地方法

> https://blog.csdn.net/cauchy6317/article/details/102851120



3.@NotNull注解：生成非空的检查

4.@Builder注解：简化对象创建过程

5.@Singular注解：配合@Builder注解，简化集合类型操作

## 6.5 Lombok优缺点

lombok虽然可以极大的提高我们的开发效率，但却降低了源代码的可读性和完整性，也加大了对问题排查的难度



# 7 验证框架

## 7.1 分层验证与JavaBean验证

### 分层验证模型

![image-20210506214821951](/images/2021060901.png)



### JavaBean验证

![image-20210506214839961](/images/2021060902.png)

Bean Validation简介

> Bean Validation为JavaBean验证定义了相应的元数据模型和API





JCP，JSR简介

>JCP(Java Community Process)成立于1998年，是使有兴趣的各方参与定义Java的特性和未来版本的正式过程
>
>JCP使用JSR（Java规范请求，Java Specification Requests）作为正式规范文档，描述被提议加入到Java体系中的规范和技术。

JSR303,Bean Validation 1.0

JSR349,Bean Validation 1.1

JSR380,Bean Validation 2.0



Bean Validation与Hibernate Validator

Bean Validation 1.0参考实现：Hibernate Validator 4.3.1.Final

Bean Validation 1.1参考实现：Hibernate Validator 5.1.1.Final

Bean Validation 2.0参考实现：Hibernate Validator 6.0.1.Final



常用约束注解

- 空值校验类：@Null，@NotNull，@NotEmpty，@NotBlank等
- 范围校验类：@Min，@Size，@Digits，@Future，@Negative等
- 其他校验类：@Email，@URL，@AssertTrue，@Pattern等

**@NotBlank 用于 String 判断空格**
**@NotEmpty 用于集合**
**@NotNull 用在基本类型上**



| **注解**     | **描述**                                                 |
| ------------ | -------------------------------------------------------- |
| @AssertFalse | 被注释的元素必须为 false                                 |
| @AssertTrue  | 同@AssertFalse                                           |
| @DecimalMax  | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @DecimalMin  | 同DecimalMax                                             |
| @Digits      | 被注释的元素是数字                                       |
| @Future      | 将来的日期                                               |
| @Max         | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @Min         | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @NotNull     | 不能是Null                                               |
| @Null        | 元素是Null                                               |
| @Past        | 被注释的元素必须是一个过去的日期                         |
| @Pattern     | 被注释的元素必须符合指定的正则表达式                     |
| @Szie        | 被注释的元素必须在指定长度内                             |
| @Email       | 元素必须是格式良好的电子邮箱地址                         |
| @Length      | 字符串的大小必须在指定的范围内，有min和max参数           |
| @NotEmpty    | 字符串的不能是空                                         |
| @NotBlank    | 字符串不能使空，但是与@NotEmpty不同的是尾随的空白被忽略  |
| @URL         | 字符串必须是一个URL                                      |





 8-5 案例演示框架搭建 (05:13)

引入pom

```xml
<!-- Validation 相关依赖 -->
    <dependency>
      <groupId>javax.validation</groupId>
      <artifactId>validation-api</artifactId>
      <version>2.0.1.Final</version>
    </dependency>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-validator</artifactId>
      <version>6.0.16.Final</version>
    </dependency>
    <dependency>
      <groupId>javax.el</groupId>
      <artifactId>javax.el-api</artifactId>
      <version>3.0.0</version>
    </dependency>
    <dependency>
      <groupId>org.glassfish.web</groupId>
      <artifactId>javax.el</artifactId>
      <version>2.2.6</version>
    </dependency>
```



简单的使用，还是以Person实体类为例（大家在尝试时最好在web项目上，毕竟这个功能基本也都在web上才用得到。非web项目可能会出现一些错误，这时候就要引入el依赖）：

```java
@Data
@AllArgsConstructor
public class Person {
    @NotNull(message = "用户名不能为空")
    @Size(min = 2,max = 15)
    private String name;
    @NotNull
    private int age;
    @NotNull(message = "性别不能为空")
    private String gender;
}

```



```java
private static Validator validator=Validation.buildDefaultValidatorFactory().getValidator();//初始化验证器
public static void main(String[] args) {
    Person person=new Person("",15,"男");//新建一个数据
    Set<ConstraintViolation<Person>> validate = validator.validate(person);//进行验证
    validate.forEach(item ->System.out.println(item.getMessage()));//对验证结果进行输出，就是输出注解后面的message。
}
```



## 7.2 完成验证的步骤

- 约束注解的定义
- 约束验证规则（结束验证器）
- 约束注解的声明
- 约束验证流程

## 7.3 测试案例

自定义手机号约束注解

- 定义@interface Phone注解
- 实现约束验证器PhoneValidator.java
- 声明@Phone约束验证
- 执行手机号约束验证流程





# 8 常用工具

>开发常用工具总结：https://liuurick.github.io/2021/04/18/%E5%BC%80%E5%8F%91%E5%B8%B8%E7%94%A8%E5%B7%A5%E5%85%B7%E6%80%BB%E7%BB%93/

## 开发工具

## 自测工具

## 检查工具



# 9 综合实战

整合一下，搭建通用型基础组件

- 集成MyBatis-plus实现数据持久化
- 集成CaffeineCache（GuavaCache替代品）实现本地缓存
- 集成Lombok注解
- 集成校验框架实现自动/手动数据校验
- 实现统一异常处理
- 集成Guava令牌桶实现全局限流器
- 使用线程池实现任务异步处理
- 使用TWR实现文件关闭
- 集成EasyExcel实现Excel异步导出
- 实现VO/DTO/DO实现代码分层
- 使用Stream实现集合操作
- 集成Swagger2实现接口文档自动生成
- 使用TraceId实现系统请求追踪



引入依赖

```xml
<dependencies>

    <!-- Swagger2支持 -->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.9.2</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.9.2</version>
    </dependency>

    <!-- EasyExcel相关支持 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>easyexcel</artifactId>
        <version>2.1.6</version>
    </dependency>
    <dependency>
        <groupId>org.ow2.asm</groupId>
        <artifactId>asm-all</artifactId>
        <version>5.2</version>
    </dependency>

    <!-- Guava支持 -->
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>28.0-jre</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
    </dependency>
    <!-- Caffeine Cache支持 -->
    <dependency>
        <groupId>com.github.ben-manes.caffeine</groupId>
        <artifactId>caffeine</artifactId>
    </dependency>

    <!-- Mybatis Plus集成 -->
    <!-- MySql驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <!-- mybatis支持 -->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.1</version>
    </dependency>
    <!-- mybatis plus支持 -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.3.1</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```





## 集成Mybatis Plus步骤

1. 在pom.xml文件引入相关的jar包依赖
2. 实现XxxMapper接口。通过此接口来操作数据持久化
3. 对XxxDO实体进行注解的定义，如：数据库表名，字段的定义
4. 如需修改Plus默认配置，需实现MybatisPlusConfig类
5. 如需要自定义一个方法，需实现XxxMapper.xml，来定义自定义SQL



## 实现VO/DTO/DO实现代码分层

> 领域模型转换那些事儿:http://www.imooc.com/article/293314

![图片描述](http://img1.sycdn.imooc.com/5d9f49df0001fcfc07070586.png)

## 自动更新系统级字段

1. 公共元数据处理器
2. 为XxxDO配置注解

## 集成验证框架步骤

1. 在pom.xml引入相关的jar包支持
2. 在待验证的实体里面添加相应的注解
3. 在Controller中添加相应的注解
4. 做参数校验工具类，完成service层的参数校验

## 实现统一异常处理功能

1. 实现一个异常处理的类，并且用@ControllerAdvice修饰

## 集成CaffeineCache缓存功能

1. @Cacheable : 缓存数据，一般用在查询方法上。将查询到的数据进行缓存 @CachePut : 更新方法上，将数据从缓存中进行更新 @CacheEvict : 删除缓存
2. pom.xml cache相关的jar包支持
3. CacheManager Bean
4. 使用注解，标识我们的方法哪些需要缓存

## 集成Guava令牌桶实现全局限流功能

1. 先pom.xml引入Guava工具包的支持
2. 定义一个拦截器，实现令牌的发放和获取
3. 将拦截器配置到web系统中

## 使用TraceId实现日志跟踪

1. 建立一个过滤器，在过滤器中给线程设置TraceId
2. 将日志配置文件进行修改，把TraceId打印到日志中

## 文件上传下载

1. 文件上传的Controller，负责处理文件上传
2. 文件上传的服务接口，通过接口的形式来定义出文件上传的功能
3. 实现文件上传业务逻辑
4. 文件下载，采用静态路径映射的方式实现

## 数据导出功能

1. pom.xml把相关的jar配置好
2. UserController新增加数据导出的方法
3. 要实现数据导出的功能
   - 定义导出实体
   - 分批加载数据，分批使用EasyExcel导出
   - 将导出的文件上传

## 导出功能异步化

1. 先创建线程池
2. 将导出方法使用@Aync注解标记为异步执行

## 使用Swagger2帮助我们生成API文档

1. pom.xml引入jar包
2. 配置Swagger2的配置类
3. Controller及相关的实体中写对应的注解