---
title: 解决Docker上安装RabbitMQ后Web管理页面打不开的问题
date: 2020-07-27 16:33:08
tags: [Docker,RabbitMQ]
categories: RabbitMQ
---
### 

<!--more-->

首先确保RabbitMQ的端口等配置正确，进入RabbitMQ中，开启一项配置。

### 解决方法

1.开启RabbitMQ  　　

`docker run -itd --name myrabbitmq -p 15672:15672 -p 5672:5672 rabbitmq`

2.进入RabbitMQ　　

`docker exec -it myrabbitmq /bin/bash`

3.开启　

`rabbitmq-plugins enable rabbitmq_management`