---
title: Mycat学习笔记
date: 2021-01-12 17:35:29
tags: Mycat
categories: [应用框架,后端,数据库,分库分表]
---

**中间件**：是介于应用系统和系统软件之间的一类软件

**数据库中间件**：连接Java应用程序和数据库

<!--more-->

# 1 概述

> Mycat官方文档：https://github.com/MyCATApache/Mycat-Server/wiki

Mycat是一款流行的数据库中间件，由cobar演进而来

为什么使用Mycat

- java与数据库紧耦合
- 高访问量高并发对数据库压力较大
- 读写请求数据不一致

Mycat作用

- 读写分离

![image-20210125155408275](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210125155408275.png)

- 数据分片

垂直拆分（分库）、水平拆分（分表）、垂直+水平拆分（分库分表）

![image-20210125155436300](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210125155436300.png)

- 多数据源整合

![image-20210125155512988](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210125155512988.png)