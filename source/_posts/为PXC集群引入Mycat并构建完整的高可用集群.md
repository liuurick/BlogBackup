---
title: 为PXC集群引入Mycat并构建完整的高可用集群
date: 2020-12-07 17:15:58
tags: [PXC集群,Mycat]
categories: MySQL
---

[在CentOS7下搭建PXC集群](https://blog.51cto.com/zero01/2467717)一文中，演示了如何从零开始搭建一个三节点的PXC集群。但是光搭建了PXC集群还不够，因为在实际的企业应用中，可能会存在多个PXC集群，每个集群作为一个数据分片存在。因此，在完整的架构下我们还需要为集群引入数据库中间件，以实现数据分片和负载均衡等功能。

<!--more-->

>https://blog.51cto.com/zero01/2467910?source=drh
>
>中间件mycat对pxc集群的分片处理:https://www.cnblogs.com/reblue520/p/10338496.html
>
>教程：https://coding.imooc.com/class/274.html

## MyCat 

分布式数据库分库分表中间件

> mycat官网：[http://www.mycat.io](http://www.mycat.io/)
>
> 中文官网：http://www.mycat.org.cn/
>
> MyCat是一个开源的分布式数据库系统，是一个实现了MySQL协议的服务器，前端用户可以把它看作是一个数据库代理，用MySQL客户端工具和命令行访问，而其后端可以用MySQL原生协议与多个MySQL服务器通信，也可以用JDBC协议与大多数主流数据库服务器通信。

**功能:**

- 数据库读写分离(写操作在主,读操作在从数据库)
- 读的负载均衡(一主多从)
- 垂直拆分(将表分开为多个数据库)
- 水平拆分(对表取模拆分)

能满足数据库数据大量存储；提高了查询性能

#### 基础概念

逻辑库(逻辑表类似)
![img](https://img-blog.csdnimg.cn/20181108095342784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NzYyNjc3,size_16,color_FFFFFF,t_70)

### Mycat的使用

[[mycat安装教程](https://blog.csdn.net/jaysonhu/article/details/52858535)]

![img](https://img-blog.csdnimg.cn/20181108104504507.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NzYyNjc3,size_16,color_FFFFFF,t_70)
[[配置文件解析](https://blog.csdn.net/wrs120/article/details/80417345#1什么是mycat)]
读写分离时需要修改的配置文件

- schema.xml
- server.xml

### mycat管理

通过mysql客户端进行管理 9066端口在server.xml配置
`mysql -u root -p123456 -h 127.0.0.1 -P9066`

### 日志管理

log4j: /conf/log4j.xml

------

### PXC

读写强一致性(牺牲性能)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181107185432729.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NzYyNjc3,size_16,color_FFFFFF,t_70)

