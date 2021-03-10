---
title: Java异常详解
date: 2020-12-27 18:31:53
tags: 异常
categories: [编程基础,Java语言,JavaSE基础]
---

[TOC]

<!--more-->

> java提高篇-----异常(一)：https://www.cnblogs.com/chenssy/p/3438130.html
>
> java提高篇-----异常(二)：https://www.cnblogs.com/chenssy/p/3453039.html

# 异常类体系

​		Throwable是所有异常的超类，两个直接子类分别为Exception和Error，其中Exception又可分为RuntimeException和非RuntimeException，这两种异常有很大的区别，也称之为不检查异常（Unchecked Exception）和检查异常（Checked Exception）。

​		其中CheckException发生在编译阶段，必须要使用try…catch（或者throws）否则编译不通过。而UncheckedException发生在运行期，具有不确定性，主要是由于程序的逻辑问题所引起的，难以排查，我们一般都需要纵观全局才能够发现这类的异常错误，所以在程序设计中我们需要认真考虑，好好写代码，尽量处理异常，即使产生了异常，也能尽量保证程序朝着有利方向发展。

![1354439580_6933](/images/2020122703.png)

RuntimeException：不可预知的，程序应当自行避免

非RuntimeException：可预知的，从编译器校验的异常



# 常见的Error以及Exception

**RuntimeException**

1. NullPointerException-空指针引用异常
2. ClassCastException-类型强制转换异常
3. IllegalArgumentException-传递非法参数异常
4. IndexOutOfBoundsException-下标越界异常
5. NumberFormatException-数字格式异常

**非RuntimeException**

1. ClassNotFoundException-找不到指定class的异常
2. IOException-IO操作异常

**Error**

1. NotClassDefFoundError-找不到class定义的异常

   NotClassDefFoundError的成因：

   1. 类依赖的class或者jar不存在
   2. 类文件存在，但是存在不同的域中
   3. 大小写问题，javac编译的时候是无视大小写的，很有可能编译出来的class文件就与想要的一样

2. StackOverflowError-深递归导致栈被耗尽而抛出的异常

3. OutOfMemoryError-内存异常异常



# Java异常的处理原则

- 具体明确：抛出的异常应能通过异常类名和message准确说明异常的类型和异常产生的原因；
- 提早抛出：应尽可能早的发现并抛出异常，便于精确定位问题；
- 延迟捕获：异常的捕获和处理应尽可能延迟，让掌握更多信息的作用域来处理异常。





# 自定义异常

   Java自定义异常的使用要经历如下四个步骤：

1. 定义一个类继承Throwable或其子类。
2. 添加构造方法(当然也可以不用添加，使用默认构造方法)。
3. 在某个方法类抛出该异常。
4. 捕捉该异常。



# 异常链

​		在设计模式中有一个叫做责任链模式，该模式是将多个对象链接成一条链，客户端的请求沿着这条链传递直到被接收、处理。同样Java异常机制也提供了这样一条链：异常链。

​		**通过使用异常链，我们可以提高代码的可理解性、系统的可维护性和友好性。**