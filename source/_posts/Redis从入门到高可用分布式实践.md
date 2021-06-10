---
title: Redis从入门到高可用分布式实践
date: 2021-05-22 21:50:07
tags: Redis
categories: [后端,中间件,缓存]
---

系统学习Redis

[TOC]

<!--more-->

# 缓存概述

## 什么是缓存？

缓存就是将数据放在距离计算最近的位置以加快处理速度。缓存是改善软件性能的第一手段，现代CPU越来越快的一个重要因素就是使用了更多的缓存，在复杂的软件设计中，缓存几乎无处不在。大型网络架构设计在很多方面都使用了缓存设计。

## 缓存的分类

**CDN**：即内容分发网络，部署在距离终端用户最近的网络服务商，用户的网络请求总是先到达他的网络服务商那里，在这里缓存网络的一些静态资源（较少变化的数据），可以就近以最快速度返回给用户，如视频网站和门户网站会将用户访问量大的热点内容缓存在CDN。

**反向代理**：反向代理属于网站前端架构的一部分，部署在网站的前端，当用户请求到达网站的数据中心时，最先访问到的就是反向代理服务器，这里缓存网站的静态资源，无需将请求继续转发给应用服务器就能返回给用户。

**本地缓存**：在应用服务器本地缓存着热点数据，应用程序可以在本机内存中直接访问数据，而无需访问数据库。

**分布式缓存**：大型网站的数据量非常庞大，即使只缓存一小部分，需要的内存空间也不是单机能承受的，所以除了本地缓存，还需要分布式缓存，将数据缓存在一个专门的分布式缓存集群中，应用程序通过网络通信访问缓存数据。



## 使用缓存的前提

使用缓存有两个前提：

1. 数据访问热点不均衡，某些数据会被频繁的访问，这些数据应该放在缓存中；
2. 数据在某个时间段内有效，不会很快过期，否则缓存的数据就会因已经失效而产生脏读，影响结果的正确性。

网站应用中，缓存除了可以加快数据访问速度，还可以减轻后端应用和数据存储的负载压力，这一点对网站数据库架构至关重要，网站数据库几乎都是按照有缓存的前提进行负载能力设计的。

## 缓存的本质

缓存的本质是一个`内存Hash表`，网站应用中，数据缓存以一对Key，Value的形式存储在内存Hash表中。Hash表数据读写的时间复杂度为O(1)，下图是一对KV在Hash表中存储。

![image-20210610100319366](/images/2021061001.png)

计算KV对中Key的HashCode对应的Hash表索引，可快速访问Hash表中的数据。许多语言支持获得任意对象的HashCode，可以把HashCode理解为对象的唯一标示符，Java语言中Hashcode方法包含在根对象Object中，其返回值是一个Int。然后通过Hashcode计算Hash表中的索引下标，最简单额是余数法，使用Hash表数组长度对Hashcode求余，余数即为Hash表索引，使用改索引可直接访问得到Hash表中存储的KV对。Hash表是软件开发中常用到的一种数据结构，其设计思想在很多场景下都可以应用。

![image-20210610100852737](/images/2021061002.png)



缓存主要用来存放那些读写比很高，很少变化的数据，如商品的类目信息，热门词的搜索列表信息，热门商品信息。应用程序读取数据时，先到缓存中读取，如果读取不到或数据已经失效，再访问数据库，并将数据写入缓存。

![image-20210610090930809](/images/2021061003.png)

# 1 Redis初识

Redis 是 C 语言开发的一个开源的（遵从 BSD 协议）高性能键值对（key-value）的内存数据库，可以用作数据库、缓存、消息中间件等。

Redis作为一个nosql，拥有非常多的优点：

- 性能优秀，数据在内存中，读写速度非常快，支持并发 10W QPS。
- 单进程单线程，是线程安全的，采用 IO 多路复用机制。
- 丰富的数据类型，支持字符串（strings）、散列（hashes）、列表（lists）、集合（sets）、有序集合（sorted sets）等。
- 支持数据持久化。
- 可以将内存中数据保存在磁盘中，重启时加载。
- 主从复制，哨兵，高可用。
- 可以用作分布式锁。
- 可以作为消息中间件使用，支持发布订阅。



# 2 Redis安装与基本操作

## 2.1 安装

- docker安装
- Linux安装
- Windows安装



## 2.2 Redis的基本命令

### KEYS

![这里写图片描述](https://img-blog.csdn.net/20180729000353251?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pwY2FuZHpoag==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### EXISTS

![这里写图片描述](https://img-blog.csdn.net/20180729000401883?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pwY2FuZHpoag==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### DEL

![这里写图片描述](https://img-blog.csdn.net/20180729000413138?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pwY2FuZHpoag==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### TYPE

![这里写图片描述](https://img-blog.csdn.net/20180729000419615?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pwY2FuZHpoag==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### HELP

HELP 空格 tab键
![这里写图片描述](https://img-blog.csdn.net/20180729000426124?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pwY2FuZHpoag==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)





# 3 API的理解和使用

全面介绍了Redis提供的5种数据结构字符串（string）、哈希（hash）、列表（list）、集合（set）、有序集合（zset）的数据模型、常用命令、典型应用场景。同时本章还会对Redis的单线程处理机制、键值管理做一个全面介绍，通过对这些原理的理解，听众可以在合适的应用场景选择合适的数据结构。 ...

共 11 节 (101分钟)
收起列表
2-1 -课程目录 (01:22)
2-2 -通用命令 (12:46)
2-3 数据结构和内部编码 (03:21)
2-4 单线程 (04:48)
2-5 字符串 (20:23)
2-6 hash (1) (06:01)
2-7 hash (2) (10:48)
2-8 list(1) (02:45)
2-9 list(2) (10:33)
2-10 set (10:27)
2-11 zset (16:55)



# 4 Redis客户端的使用
- Java客户端：Jedis 
- Python客户端：redis-py 
- Go客户端：redigo

## 4.1 Jedis的使用





## 4.2 Spring项目集成Redis

## 4.3 SpringBoot项目集成Redis

pom依赖导入

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```





单服务器配置：

properties：

```properties
# redis单服务器配置
spring.redis.host=127.0.0.1
spring.redis.port=6379
# 连接redis的密码 默认是没有的,需要在 redis.conf文件中配置
spring.redis.password=abc123456
spring.redis.database=0
spring.redis.jedis.pool.max-active=8
spring.redis.jedis.pool.max-wait=-1
spring.redis.jedis.pool.min-idle=0
spring.redis.jedis.pool.max-idle=8
spring.redis.timeout=2000
```

yml

```yml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    password: abc123456
    timeout: 10000
    lettuce:
      pool:
        max-active: 8
        max-wait: -1ms
        min-idle: 0
        max-idle: 8
```



## 4.4 Jedis配置优化



# 5 Redis其他功能
除了5种数据结构外，Redis还提供了诸如慢查询、Pipeline、Bitmap、HyperLogLog、发布订阅、GEO等附加功能，在这些功能的帮助下，Redis的应用场景更加丰富。

共 7 节 (53分钟)
收起列表
4-1 课程目录 (01:03)
4-2 慢查询 (09:49)
4-3 pipeline (08:58)
4-4 发布订阅 (07:12)
4-5 bitmap (11:25)
4-6 hyperloglog (07:54)
4-7 geo (06:15)

# 6 Redis持久化的取舍和选择
Redis的持久化功能有效避免因进程退出造成的数据丢失问题，本章将介绍介绍RDB和AOF两种持久化配置和运行流程，以及选择策略

共 9 节 (63分钟)
收起列表
5-1 目录 (01:03)
5-2 持久化的作用 (02:52)
5-3 RDB(1) (08:16)
5-4 RDB(2) (14:18)
5-5 RDB(3) (03:46)
5-6 AOF(1) (08:10)
5-7 AOF(2) (10:37)
5-8 AOF实验 (04:54)
5-9 RDB和AOF抉择 (08:05)

# 7 常见的持久化开发运维问题
本章探讨了常见的持久化问题进行定位和优化，最后结合Redis常见的单机多实例部署场景进行优化

共 4 节 (13分钟)
收起列表
6-1 常见问题目录 (00:43)
6-2 fork (03:37)
6-3 子进程开销和优化 (04:57)
6-4 AOF阻塞 (02:44)

# 8 Redis复制的原理与优化
复制是实现高可用的基石，但复制同样是运维的痛点，本部分详细分析复制的原理，讲解运维过程中可能遇到的问题。

















共 9 节 (59分钟)
收起列表
7-1 目录 (01:13)
7-2 什么是主从复制 (05:49)
7-3 主从复制配置-介绍 (05:07)
7-4 主从复制配置-操作 (13:13)
7-5 runid和复制偏移量 (04:06)
7-6 全量复制 (03:22)
7-7 全量复制开销 + 部分复制 (03:48)
7-8 故障处理 (05:49)
7-9 主从复制常见问题 (15:29)



# 9 Redis Sentinel
本章将一步步解析Redis Sentinel的相关概念、安装部署、配置、客户端路由、原理解析，最后分析了Redis Sentinel运维中的一些问题。
收起列表
8-1 sentinel-目录 (01:06)
8-2 主从复制高可用？ (03:57)
8-3 redis sentinel架构 (04:44)
8-4 redis sentinel安装与配置 (06:24)
8-5 redis sentinel安装演示-1 (03:32)
8-6 redis sentinel安装演示-2 (09:44)
8-7 java客户端 (06:42)
8-8 python客户端 (01:19)
8-9 实现原理-1-故障转移演练 (08:19)
8-10 实现原理-2.故障转移演练(客户端) (03:58)
8-11 实现原理-3.故障演练(日志分析) (06:48)
8-12 三个定时任务 (05:34)
8-13 主观下线和客观下线 (04:48)
8-14 领导者选举 (04:06)
8-15 故障转移 (05:52)
8-16 常见开发运维问题-目录 (00:33)
8-17 节点运维 (05:20)
8-18 高可用读写分离 (09:26)
8-19 本章总结 (04:32)
# 10 Redis Cluster
> https://liuurick.github.io/2020/07/30/Redis%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA/



# 11 缓存设计与优化

讲解将缓存加入应用架构后带来的一些问题，这些问题常常会成为应用的致命点。

共 9 节 (65分钟)
收起列表
11-1 目录 (01:25)
11-2 缓存的受益和成本 (06:36)
11-3 缓存的更新策略 (07:09)
11-4 缓存粒度问题 (05:14)
11-5 缓存穿透问题 (12:30)
11-6 缓存雪崩优化 (09:53)
试看
11-7 无底洞问题 (07:32)
11-8 热点key的重建优化 (11:45)
11-9 本章总结 (02:41)

# 12 Redis云平台CacheCloud
本章结合前面的知识介绍redis规模化后使用云平台如何进行机器部署、应用接入、用户相关功能维护等问题

共 7 节 (41分钟)
收起列表
12-1 _目录 (01:45)
12-2 _Redis规模化困扰 (05:27)
12-3 _快速构建 (05:25)
12-4 机器部署 (08:22)
12-5 应用接入 (13:34)
12-6 用户功能 (02:58)
12-7 运维功能 (03:20)

