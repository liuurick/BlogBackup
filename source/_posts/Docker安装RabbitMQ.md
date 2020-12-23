---
title: Docker安装RabbitMQ
date: 2020-07-27 16:33:08
tags: [Docker,RabbitMQ]
categories: RabbitMQ
---
 RabbitMQ是基于AMQP的一款消息管理系统。AMQP(Advanced Message Queuing Protocol)，是一个提供消息服务的应用层标准高级消息队列协议，其中RabbitMQ就是基于这种协议的一种实现。

<!--more-->

# Docker快速安装

1.docker search选择版本

2.docker pull 下载镜像

3.docker运行即可

```
docker run -d -p 5672:5672 -p 15672:15672 rabbitmq:版本号
```



![image-20201219141443802](/images/2020072701.png)

# 安装问题

**可能出现的问题**：Windows环境下浏览器无法访问虚拟机开的rabbitmq服务

安装首先确保RabbitMQ的端口等配置正确，另外开放linux防火墙端口。如果依旧不能访问web界面，那么可能是rabbitmq没有开启web管理。

只需要通过`rabbitmq-plugins list`命令列出插件的启用和禁用状态。然后再修改即可。

## 解决方法

1.开启RabbitMQ  　　

```
docker run -d -p 5672:5672 -p 15672:15672 rabbitmq
```

2.进入RabbitMQ命令界面　　

```
docker exec -it rabbitmq /bin/bash
```

3.开启　

```
rabbitmq-plugins enable rabbitmq_management
```

