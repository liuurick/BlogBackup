---
title: IDEA配置与常用快捷键
date: 2021-01-20 15:21:19
tags: IDEA
categories: [开发工具,集成开发环境,IDEA] 
---

[TOC]

<!--more-->

# 1 IDEA配置

## 1.1 整合maven

## 1.2 JDK配置

1.全局设置JDK

![image-20210408200026894](/images/2021040801.png)



2.选择安装路径进行配置


![image-20210408200147520](/images/2021040802.png)



## 1.3 编码设置为UTF8

## 1.4 启动时不自动打开上一次的项目

取消勾选

![image](/images/2021012001.png)



# 2 常用快捷键

**快捷键技巧：**

对于Windows：HotKey Explorer，HotKey Commander，快捷键说明

对于macOS：CheatSheet

**查看快捷键文档：**

Helper--》Keymap Preference

## 2.1 代码格式化

```bash
去掉空白： Ctrl + Shift + J
格式化代码： Ctrl + Alt + L
```



# 3 常用插件

## 3.1 Stackoverflow 

这个插件其实是最实用的插件，程序猿遇到的问题，基本都能找到回答，但是它使用的是google搜索引擎，对于，不购买vpn的同学来说，感觉好鸡肋呀~

> 选中需要搜索的问题，然后，右键点击

## 3.2 FindBugs

Idea自带的检查工具已经很强大，如有需要也可以加上`Alibaba Java Coding Guidelines`的代码检查工具，但是，说白这些工具其实更多的是规范性检查，如果需要更深入的去检查异常，可以使用此插件~

> 右键点击文件，包或者工程，会出现如下界面

## 3.3 TranslationPlugin

对于不经常使用英语的同学来说，对类，变量，方法想取一个合适的名字，此时发现自己的词汇早已还给老师 ，怎么办，这个插件能帮到你~

> 直接选中你想要翻译的词，然后右键选择，或者快捷键 Ctrl+Shift+F3

## 3.4 Mybatis-log-plugin

开发的项目一般都少不了日志系统，而我们在书写mysql语句的时候，参数的对应，往往有时候会忽略，mybatis自己控制的参数编译对应，个人感觉有点反人类，我们可以使用这个插件变成自己比较直观的对应~

> 选中需要转换的mybatis log日志，然后点击右键，选择Restore sql from slection



## 3.5 GrepConsole

Idea console输出日志一大推，想要快速找到自己想要的类型日志，使用此插件可以快速定位到自己关注的类型日志，比如error，warn，自己也可以配置自己喜欢的颜色~

> 从settings进入，点击 other settings，可以配置自己喜欢的颜色提示，比如我只选择了默认~

## 3.6 GsonFormat

在与组外或者不同部门对接接口时候发现，有时候对方返回的是JSON对象，自己想要用一个对象去接受，以便于处理后续，此时，需要自己一个个手动去输入属性么，肯定很抓狂，不过咱们可以使用这个插件来解决这个尴尬问题，当然也可以使用外部网址解决，比如bejson这个网站~  

## 3.7 IdeaJad

以前查看class文件形式的时候或者jar，都会使用一个外部反编译工具，这样操作明显不方便，使用此插件可以一直在idea中查看文件~  ps：其实Inteli Idea这个编译器已经自带了反编译功能，老夫~~~~~~

> 选择class文件，右键 Decompile,完成反编译

## 3.8 Free-idea-mybatis

mybatis xml和对应的mapper之间来回切换的时候，有时候不同人开发，放置的位置又不同，使用此插件后，来回切换的时候异常方便，和所放置的位置无关~

## 3.9 CodeGlance

再也不用疯狂拖拽到底去找一遍啦，多不方便呀，使用此插件可以查看缩略图一样，快速切换到自己需要去的地方



## 3.10 MyBatisCodeHelperPro

这个是一款比较实用的插件。但是，现在需要收费啦，貌似是需要花费29块钱，送两个激活码。不过，也可以申请7天的免费测试码，体验一下在购买也可以的。收费掩盖不了她的魅力所在，这也是行业发展的趋势。具体功能如下（总有一款适合你~）：

提供Mapper接口与配置文件中对应SQL的导航.
编辑XML文件时自动补全.
根据Mapper接口, 使用快捷键生成xml文件及SQL标签.
ResultMap中的property支持自动补全，支持级联(属性A.属性B.属性C).
快捷键生成@Param注解.
XML中编辑SQL时, 括号自动补全.
XML中编辑SQL时, 支持参数自动补全(基于@Param注解识别参数).
自动检查Mapper XML文件中ID冲突.
自动检查Mapper XML文件中错误的属性值.
支持Find Usage.
支持重构从命名.
支持别名.
自动生成ResultMap属性.

> 快捷键: Option + Enter(Mac) | Alt + Enter(Windows).
> 安装成功最明显的标志就是~  有好多小鸟在飞~



## 3.11 VisualVM Launcher

一般可用于在本地开发进行压力测试，性能测试之类的监控器，其他场景一般不推荐使用此模式启动，还会启动另外一个Visual vm窗口，这个窗口是JDK bin目录下的JvisualVM 



安装成功并且启动后的画面如下：

![image-20210408201348794](/images/2021040803.png)



## 3.12 Jrebel

是一款比较常见的热部署插件，一般用于Run模式下的自动编译，破译版本



## 3.13 JUnitGenerator V2.0

有一个好的编写单元测试习惯的开发者，代码质量肯定是很好的，可以随时校验自己开发和改写接口的快速检查工具。也避免了测试提的bug多而影响个人绩效（有些公司把bug计入考核范围内）。



## 3.14 Maven Helper

主要功能如下：查找和排除冲突依赖项的简便方法，为包含当前文件或根模块的模块运行/调试maven目标的操作，运行/调试当前测试文件的操作



## 3.15 RestfulToolkit

根据 URL 直接跳转到对应的方法定义 ( Ctrl \ or Ctrl Alt N );
提供了一个 Services tree 的显示窗口;
一个简单的 http 请求工具;
在请求方法上添加了有用功能: 复制生成 URL;,复制方法参数...
其他功能: java 类上添加 Convert to JSON 功能，格式化 json 数据 ( Windows: Ctrl + Enter; Mac: Command + Enter )

## 3.16 Alibaba Java Coding Guidelines

一款阿里巴巴公司试行的开发设计规范



## 3.17 GenerateAllSetter

当你进行对象之间赋值的时候，你会发现好麻烦呀，能不能有一个更好的办法呢~ 有，只要你选中需要生成set方法的对象。

> 快捷键 alt+enter 



## 3.18 Save Actions

自动格式化代码插件

![image-20210408201905931](/images/2021040804.png)

设置好之后，写代码时按 `ctrl + s` 就会把无用的import自动清除