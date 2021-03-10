---
title: Mycat学习笔记
date: 2021-01-12 17:35:29
tags: Mycat
categories: [后端,数据库,分库分表]
---

**中间件**：是介于应用系统和系统软件之间的一类软件

**数据库中间件**：连接Java应用程序和数据库

<!--more-->

# 1 概述

> Mycat官方文档：https://github.com/MyCATApache/Mycat-Server/wiki
>
> Mycat语雀官方文档：https://www.yuque.com/books/share/0576de75-ffc4-4c34-8586-952ae4636944

Mycat是一款流行的数据库中间件，由cobar演进而来

为什么使用Mycat

- java与数据库紧耦合
- 高访问量高并发对数据库压力较大
- 读写请求数据不一致

Mycat作用

- 读写分离

![image-20210125155408275](/images/2021011201.png)

- 数据分片

垂直拆分（分库）、水平拆分（分表）、垂直+水平拆分（分库分表）

![image-20210125155436300](/images/2021011202.png)

- 多数据源整合

![image-20210125155512988](/images/2021011203.png)

Mycat原理：

Mycat原理就两个字**拦截**

![image-20210125203845447](/images/2021011204.png)

# 2 Mycat安装启动

常见安装方式：

```
1.rpm安装
  .rpm安装包，安装顺序安装
2.yum安装
  需要联网安装
3.Docker安装
4.解压后即可使用
5.解压后编译安装
```



配置文件：

```
schema.xml：定义逻辑库，表、分片节点等内容
rule.xml：定义分片规则
server.xml：定义用户以及系统相关变量，如端口等
```





