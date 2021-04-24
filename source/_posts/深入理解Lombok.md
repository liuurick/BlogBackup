---
title: 深入理解Lombok
date: 2021-04-24 18:01:12
tags: Lombok
categories:
---

[TOC]

<!--more-->

# Lombok简介

Project Lombok是一个Java库，可以自动插入编辑器并构建工具，为您的Java增添色彩。永远不要再写另一个getter或equals方法，使用一个注释，您的类具有一个功能齐全的构建器，自动化您的日志记录变量。--Lombok官网



# Lombok实现原理

注解的两种解析方式

- 运行时解析---类似于反射
- 编译时解析---Lombok

编译时解析的两种机制

- Annotation Processing Tool（注解处理器）----JDK1.8去除
- Pluggable Annotation Processing API（JSR269插入式注解处理器）

![image](/images/2020042401.png)

# Lombok插件安装

> IDEA-->setting-->plugins-->搜索lombok安装即可



# Lombok常用注解使用

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

# Lombok优缺点

lombok虽然可以极大的提高我们的开发效率，但却降低了源代码的可读性和完整性，也加大了对问题排查的难度

