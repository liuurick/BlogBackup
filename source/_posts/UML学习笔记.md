---
title: UML类图及时序图入门
date: 2020-12-07 17:27:13
tags: [UML,时序图]
categories: [软件工程,UML]
---

[TOC]

<!--more-->

# UML学习

## UML定义

- 统一建模语言（Unified Modeling Language）
- 非专利的第三代建模和规约语言

## UML特点

- UML是一种开发的方法
- 用于说明，可视化，构建和编写一个正在开发的面向对象的，软件密集系统的制品的开发方法

- UML展现了一系列最佳工程实践，这些最佳实践在对大规模，复杂系统进行建模方面，特别是在软件架构层次已经被验证有效

## UML2.2分类

UML2.2中一共定义了14种图示，分类如下：

- 结构式图形：强调的是系统式的建模
- 行为式图形：强调系统模型中触发的事件
- 交互式图形：属于行为式图形子集合，强调系统模型中资料流程



### 结构式图形

- 静态图（类图，对象图，包图）
- 实现图（组件图，部署图）
- 剖面图
- 复合结构图

### 行为式图形

- 活动图
- 状态图
- 用例图

### 交互式图形

- 通信图
- 交互概述图（UML2.0）
- 时序图（UML2.0）
- 时间图（UML2.0）



UML类图

- Class Diagram:用于表示类，接口，实例等之间相互的静态关系

- 虽然名字叫类图，但类图中并不只有类



记忆技巧

- UML箭头方向：从子类指向父类
- 提示：可能会认为子类是以父类为基础的，箭头应从父类指向子类



箭头方向：

- 定义子类时需要通过extends关键字指向父类
- 子类一定是知道父类定义的，但父类并不知道子类的定义
- 只有知道对方信息时才能指向对方
- 所以箭头方向是从子类指向父类



实线-继承|虚线-实现

- 空心三角箭头：继承或实现
- 实线-继承，is a 关系，扩展目的，不虚，很结实
- 虚线-实现，虚线代表“虚”无实体