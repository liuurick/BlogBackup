---
title: ASM开发入门
date: 2020-12-30 10:41:45
tags:
---

认识ASM：是什么，有什么，能干什么

ASM编程模型和核心API

ASM开发实战：实现统计方法的运行时间

<!--more-->

# ASM概述

> ASM是一个Java字节码操作框架，它能被用来动态生成类或者增强既有类的功能
>
> ASM可以直接产生二进制Class文件，也可以在类被加载入虚拟机之前动态改变类行为，ASM从类文件中读取信息后，能够改变类行为，分析类信息，甚至能根据要求生成新类
>
> 目前许多框架如cglib，Hibernate，Spring都直接或间接地使用ASM操作字节码



# ASM编程模型：

> **Core API**：提供了基于事件形式的编程模型。该模型不需要一次性将整个类的结构读取到内存中，因此这种方式更快，需要更少的内存，但这种编码方式难度较大。
>
> **Tree API**：提供了基于树形的编程模型。该模型需要一次性将一个类的完整结构全部读取到内存当中，所以这种方法需要更多的内存，这种编码格式较简单。



# ASM的Core API

> ASM Core API中操纵字节码的功能基于ClassVisitor接口。这个接口中的每个方法对应了class文件中的每一项。
>
> ASM提供了三个基于ClassVisitor接口的类来实现class文件的生成的转换

1.ClassReader：ClassReader解析一个类的class字节码

2.ClassAdapter：ClassAdapter是ClassVisitor的实现类，实现要变化的功能

3.ClassWriter：ClassWriter也是ClassVisitor的实现类，可以用来输出变化后的字节码

ASM给我们提供了ASMifiter工具来帮助开发，可使用**ASMifier**工具生成ASM结构来对比





# ClassVisitor开发









# MethodVisitor开发 





# 实现模拟AOP功能 