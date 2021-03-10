---
title: Java注解详解
date: 2020-12-27 18:32:49
tags: 注解
categories: [编程基础,Java语言,JavaSE基础]
---

[TOC]

<!--more-->

> https://blog.csdn.net/javazejian/article/details/71860633
>
> https://www.runoob.com/w3cnote/java-annotation.html

# Annotation 架构

![img](/images/2020122704.png)

# 基本Annotation

`@Override`：限定重写父类方法，强制一个子类必须覆盖父类的的方法。

```java
@Target(value=METHOD)
@Retention(value=SOURCE)
public @interface Override
```



`@Deprecated`：标记已过时，用于表示某个程序元素（类，方法等）已过时，当其他程序使用已过时的类，方法时，编译器将会给出警告。

```java
@Documented
@Retention(value=RUNTIME)
@Target(value={CONSTRUCTOR,ElementType,LOCAL_VARIABLE,METHOD,PACKAGE,PARAMETER,TYPE})
public @interface Deprecated
```



`@SuppressWarnings`：抑制编译器警告，指示被该Annotation修饰的程序元素（以及该程序元素中的所有子元素）取消显示指定的编译器警告。

```java
@Target(value={TYPE,ElementType,METHOD,PARAMETER,CONSTRUCTOR,LOCAL_VARIABLE})
@Retention(value=SOURCE)
public @interface SuppressWarnings
```



`@SafeVarargs`：Java7的“堆污染”警告

堆污染：当把一个不带泛型的对象赋给一个带泛型的的变量时，就会引发堆污染。

```java
@Documented
@Retention(value=RUNTIME)
@Target(value={CONSTRUCTOR,METHOD})
public @interface SafeVarargs
```



`@FunctionalInterface`：函数式接口

```java
@Documented
@Retention(value=RUNTIME)
@Target(value=TYPE)
public @interface FunctionalInterface
```



# JDK的元Annotation



`@Retention`：指示要注释具有注释类型的注释的保留时间

```java
@Documented
@Retention(value=RUNTIME)
@Target(value=ANNOTATION_TYPE)
public @interface Retention
```



`@Target`：指示注释类型适用的上下文。

```java
@Documented
@Retention(value=RUNTIME)
@Target(value=ANNOTATION_TYPE)
public @interface Target
```



`@Documented`：用于指定被元注解修饰的Annotation类建被javadoc工具提取成文档，如果定义Annotation类时使用了@Documented修饰，则所有使用该Annotation修饰的程序元素的API文档中将会包含该Annotation说明。

```java
@Documented
@Retention(value=RUNTIME)
@Target(value=ANNOTATION_TYPE)
public @interface Documented
```



`@Inherited`：表示注释类型自动继承。如果在注释类型声明中存在继承的元注释，并且用户在类声明上查询注释类型，并且类声明没有此类型的注释，则该类的超类将自动查询注释类型。将重复此过程，直到找到此类型的注释，或者达到类层次结构（Object）的顶部。 如果没有超类具有此类型的注释，则查询将指示所讨论的类没有这样的注释。 

```java
@Documented
@Retention(value=RUNTIME)
@Target(value=ANNOTATION_TYPE)
public @interface Inherited
```





# 自定义注解

定义新的Annotation类型使用@interface关键字（在原有的interface关键字前增加@符号）定义一个新的Annotation类型与定义一个接口非常像。