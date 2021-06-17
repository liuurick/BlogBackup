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

keys命令一般不再生产环境使用

![这里写图片描述](/images/2021061501.png)

### EXISTS

![这里写图片描述](/images/2021061502.png)

### DEL

![这里写图片描述](/images/2021061503.png)

### TYPE

![这里写图片描述](/images/2021061504.png)

### HELP

HELP 空格 tab键
![这里写图片描述](/images/2021061505.png)





# 3 API的理解和使用

全面介绍Redis提供的5种数据结构字符串（string）、哈希（hash）、列表（list）、集合（set）、有序集合（zset）的数据模型、常用命令、典型应用场景。同时本章还会对Redis的单线程处理机制、键值管理做一个全面介绍，通过对这些原理的理解，听众可以在合适的应用场景选择合适的数据结构。 ...

## 3.1 数据结构和内部编码 

![image-20210615214817775](/images/2021061514.png)

## 3.2 单线程 

redis在一个段时间内只会执行一个线程

> Redis为什么会使用单线程？单线程为什么这么快？
>
> 1.纯内存
>
> 2.非阻塞IO
>
> 3.不免线程切换和竞态切换

## 3.3 string

## 3.4 hash

## 3.5 list

## 3.6 set 
## 3.7 zset





# 4 Redis客户端的使用
- Java客户端：Jedis 
- Python客户端：redis-py 
- Go客户端：redigo

## 4.1 Jedis的使用

## 4.2 Spring项目集成Redis

## 4.3 SpringBoot项目集成Redis

### 1.pom依赖导入

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 2.单服务器配置：

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
  redis:
    host: localhost # Redis服务器地址
    database: 0 # Redis数据库索引（默认为0）
    port: 6379 # Redis服务器连接端口
    password: # Redis服务器连接密码（默认为空）
    jedis:
      pool:
        max-active: 8 # 连接池最大连接数（使用负值表示没有限制）
        max-wait: -1ms # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-idle: 8 # 连接池中的最大空闲连接
        min-idle: 0 # 连接池中的最小空闲连接
    timeout: 3000ms # 连接超时时间（毫秒）

```

### 3.在根节点下添加Redis自定义key的配置

```yml
# 自定义redis key
redis:
  key:
    prefix:
      authCode: "portal:authCode:"
    expire:
      authCode: 120 # 验证码超期时间
```

### 4.添加RedisService接口用于定义一些常用Redis操作

```java
/**
 * redis操作Service,
 * 对象和数组都以json形式进行存储
 */
public interface RedisService {
    /**
     * 存储数据
     */
    void set(String key, String value);

    /**
     * 获取数据
     */
    String get(String key);

    /**
     * 设置超期时间
     */
    boolean expire(String key, long expire);

    /**
     * 删除数据
     */
    void remove(String key);

    /**
     * 自增操作
     * @param delta 自增步长
     */
    Long increment(String key, long delta);

}
```

### 5.注入StringRedisTemplate，实现RedisService接口

```java
import com.macro.mall.tiny.service.RedisService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

/**
 * redis操作Service的实现类
 */
@Service
public class RedisServiceImpl implements RedisService {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public void set(String key, String value) {
        stringRedisTemplate.opsForValue().set(key, value);
    }

    @Override
    public String get(String key) {
        return stringRedisTemplate.opsForValue().get(key);
    }

    @Override
    public boolean expire(String key, long expire) {
        return stringRedisTemplate.expire(key, expire, TimeUnit.SECONDS);
    }

    @Override
    public void remove(String key) {
        stringRedisTemplate.delete(key);
    }

    @Override
    public Long increment(String key, long delta) {
        return stringRedisTemplate.opsForValue().increment(key,delta);
    }
}
```

### 6.添加UmsMemberController

> 添加根据电话号码获取验证码的接口和校验验证码的接口

```java
package com.macro.mall.tiny.controller;

import com.macro.mall.tiny.common.api.CommonResult;
import com.macro.mall.tiny.service.UmsMemberService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * 会员登录注册管理Controller
 */
@Controller
@Api(tags = "UmsMemberController", description = "会员登录注册管理")
@RequestMapping("/sso")
public class UmsMemberController {
    @Autowired
    private UmsMemberService memberService;

    @ApiOperation("获取验证码")
    @RequestMapping(value = "/getAuthCode", method = RequestMethod.GET)
    @ResponseBody
    public CommonResult getAuthCode(@RequestParam String telephone) {
        return memberService.generateAuthCode(telephone);
    }

    @ApiOperation("判断验证码是否正确")
    @RequestMapping(value = "/verifyAuthCode", method = RequestMethod.POST)
    @ResponseBody
    public CommonResult updatePassword(@RequestParam String telephone,
                                 @RequestParam String authCode) {
        return memberService.verifyAuthCode(telephone,authCode);
    }
}
Copy to clipboardErrorCopied
```

### 7.添加UmsMemberService接口

```java
package com.macro.mall.tiny.service;

import com.macro.mall.tiny.common.api.CommonResult;

/**
 * 会员管理Service
 * Created by macro on 2018/8/3.
 */
public interface UmsMemberService {

    /**
     * 生成验证码
     */
    CommonResult generateAuthCode(String telephone);

    /**
     * 判断验证码和手机号码是否匹配
     */
    CommonResult verifyAuthCode(String telephone, String authCode);

}
Copy to clipboardErrorCopied
```

### 8.添加UmsMemberService接口的实现类

> 生成验证码时，将自定义的Redis键值加上手机号生成一个Redis的key,以验证码为value存入到Redis中，并设置过期时间为自己配置的时间（这里为120s）。校验验证码时根据手机号码来获取Redis里面存储的验证码，并与传入的验证码进行比对。

```java
package com.macro.mall.tiny.service.impl;

import com.macro.mall.tiny.common.api.CommonResult;
import com.macro.mall.tiny.service.RedisService;
import com.macro.mall.tiny.service.UmsMemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;

import java.util.Random;

/**
 * 会员管理Service实现类
 */
@Service
public class UmsMemberServiceImpl implements UmsMemberService {
    @Autowired
    private RedisService redisService;
    @Value("${redis.key.prefix.authCode}")
    private String REDIS_KEY_PREFIX_AUTH_CODE;
    @Value("${redis.key.expire.authCode}")
    private Long AUTH_CODE_EXPIRE_SECONDS;

    @Override
    public CommonResult generateAuthCode(String telephone) {
        StringBuilder sb = new StringBuilder();
        Random random = new Random();
        for (int i = 0; i < 6; i++) {
            sb.append(random.nextInt(10));
        }
        //验证码绑定手机号并存储到redis
        redisService.set(REDIS_KEY_PREFIX_AUTH_CODE + telephone, sb.toString());
        redisService.expire(REDIS_KEY_PREFIX_AUTH_CODE + telephone, AUTH_CODE_EXPIRE_SECONDS);
        return CommonResult.success(sb.toString(), "获取验证码成功");
    }


    //对输入的验证码进行校验
    @Override
    public CommonResult verifyAuthCode(String telephone, String authCode) {
        if (StringUtils.isEmpty(authCode)) {
            return CommonResult.failed("请输入验证码");
        }
        String realAuthCode = redisService.get(REDIS_KEY_PREFIX_AUTH_CODE + telephone);
        boolean result = authCode.equals(realAuthCode);
        if (result) {
            return CommonResult.success(null, "验证码校验成功");
        } else {
            return CommonResult.failed("验证码不正确");
        }
    }

}
```

## 4.4 Jedis配置优化



# 5 Redis其他功能
除了5种数据结构外，Redis还提供了诸如慢查询、Pipeline、Bitmap、HyperLogLog、发布订阅、GEO等附加功能，在这些功能的帮助下，Redis的应用场景更加丰富。



# 6 Redis持久化的取舍和选择
- AOF
- RDB

# 7 常见的持久化开发运维问题


# 8 Redis复制的原理与优化
Redis主从复制

# 9 Redis Sentinel
# 10 Redis Cluster
> https://liuurick.github.io/2020/07/30/Redis%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA/



# 11 缓存设计与优化

## 11.1 缓存击穿

概念

key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求会瞬间把后端DB压垮。

特点：超热点key的数量很少，例如“爆款”商品，可以提取预知

![image-20210611093028050](/images/2021061506.png)

解决方案：

- 使用互斥锁（mutex key）
- 设置key永不过期（update的情况下删除缓存）



## 11.2 缓存穿透

用户想要查询一个数据，发现redis中没有，也就是缓存没有命中，于是向持久层数据库查询。发现也没有，于是本次查询失败。当高并发时，缓存都没有命中，于是都去请求数据库。这会给数据库造成很大的压力，甚至宕机，这时候叫做出现了缓存穿透。

**缓存击穿与缓存穿透的区别**

缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。而穿透是redis和数据库都没有改数据，比如黑客发起的攻击。



解决方案

- 布隆过滤器

![image-20210611102421271](/images/2021061507.png)



2 设置空值方案

![image-20210611102438499](/images/2021061508.png)



设置空值带来的问题

如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有更多的空值的键

即使对空值设置了过期时间，还是会存在redis和DB的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响（生效之后删除缓存）

## 11.3 缓存雪崩

缓存雪崩是指缓存层出现了错误，不能正常工作了。于是所有的请求都会达到存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况。



正常从Redis中获取

![image-20210611102031182](/images/2021061509.png)

缓存失效瞬间

![image-20210611102314700](/images/2021061510.png)



解决方案

- redis高可用（集群）
- 加随机因子（根据商品冷热程度）
- 限流降级（已经发生雪崩的情况下，系统架构级别策略）
- 加锁排队（同缓存击穿）



## 11.4 数据库和Redis双写一致性策略

> 如何保证 Redis 缓存与数据库双写一致性？https://mp.weixin.qq.com/s/5I4IQFYZDdeNulSZfEj79A

1. 先更新数据库，后更新缓存
2. 先更新数据库，后删除缓存
3. 先更新缓存，后更新数据库
4. 先删除缓存，后更新数据库

## 分布式锁

1. 基于关系数据库乐观锁 

2. 基于 Redis 的分布式锁 
3. 基于 ZooKeeper 的分布式锁

# 12 Redis云平台CacheCloud

结合前面的知识介绍redis规模化后使用云平台如何进行机器部署、应用接入、用户相关功能维护等问题

