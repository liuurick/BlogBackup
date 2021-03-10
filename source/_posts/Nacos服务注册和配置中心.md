---
title: Nacos服务注册和配置中心
date: 2021-02-04 11:33:25
tags: SpringCloud Alibaba
categories: [后端,分布式/微服务]
---

[TOC]

<!--more-->

Nacos（Dynamic Naming and Configuration Service ） 等价于  `注册中心`+`配置中心`的组合

> 官方文档：https://github.com/alibaba/Nacos
>
> 下载地址：https://github.com/alibaba/nacos/releases/tag/1.1.4

# 1 Docker安装启动Nacos

## 1.1 拉取镜像

```
docker pull nacos/nacos-server
```

## 1.2 创建本地的映射文件 custom.properties

```
mkdir -p /root/nacos/init.d /root/nacos/logs
vim /root/nacos/init.d/custom.properties
```

在文件中写入以下配置

```
management.endpoints.web.exposure.include=*
```

##  1.3 创建容器并启动提供a、b两种方案

a.创建容器：使用`standalone`模式并开放`8848`端口，并映射配置文件和日志目录，数据库默认使用 `Derby`

```bash
docker run -d -p 8848:8848 -e MODE=standalone -e PREFER_HOST_MODE=hostname -v /root/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties -v /root/nacos/logs:/home/nacos/logs --restart always --name nacos nacos/nacos-server
```

**注意**：记得开放端口

![image-20210204153835750](/images/2021020401.png)

 b.使用docker-compose启动 [docker-compose安装教程](https://www.cnblogs.com/ZCQ123456/p/11921817.html)

首先配置docker-compose文件 `standalone-derby.yaml` 

```yaml
version: "2"
services:
  nacos:
    image: nacos/nacos-server:latest
    container_name: nacos
    environment:
    - MODE=standalone
    volumes:
    - /root/nacos/logs:/home/nacos/logs
    -  /root/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties
    ports:
    - "8848:8848"
```

启动、关闭、移除、关闭并移除命令如下：

```bash
#启动
docker-compose -f standalone-derby.yaml up
#关闭
docker-compose -f standalone-derby.yaml stop
#移除
docker-compose -f standalone-derby.yaml rm
#关闭并移除
docker-compose -f standalone-derby.yaml down
```

## 1.4 访问

> 输入网址：http://192.168.60.129:8848/nacos  账号默认nacos、密码默认nacos

![image-20210204142942591](/images/2021020402.png)

进入控制台：

![image-20210204143107879](/images/2021020403.png)



# 2 Nacos作为服务注册中心

创建两个服务提供者和一个服务消费者

创建module主要分为以下几步：

1. 建module
2. pom
3. yml
4. 主启动
5. 业务类



![image-20210204154139801](/images/2021020404.png)

## 服务注册中心对比

Nacos全景图：

![image-20210204154903838](/images/2021020405.png)



![img](/images/2021020406.png)



**Nacos支持AP和CP模式的切换**



# 3 Nacos作为服务配置中心

## 3.1 Nacos作为配置中心-基础配置

cloudalibaba-config-nacos-client3377

自带动态刷新

## 3.2 Nacos作为配置中心-分类配置

- DataID方案

- Group方案

- Namespace方案

# 4 Nacos集群和持久化配置

> Nacos官方文档：https://nacos.io/zh-cn/docs/deployment.html

## 4.1 Nacos支持三种部署方式

- 单机模式：用于测试或单机使用
- 集群模式：用于生产环境，确保高可用
- 多集群模式：用于多数据中心场景



## 4.2 Nacos持久化配置

Nacos默认自带的是嵌入式数据库`derby`，derby到mysql切换配置步骤

启动nacos，可以看到是个全新的空记录界面，以前是记录进derby



## 4.3 Linux版Nacos+MySQL生产环境配置

> https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html