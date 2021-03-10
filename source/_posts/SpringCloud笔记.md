---
title: SpringCloud笔记
date: 2020-08-15 18:33:20
tags: SpringCloud
categories: [后端,分布式/微服务]
---

[TOC]

<!--more-->

# 相关资料

> Spring Boot:https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/htmlsingle/

> Spring Cloud:https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/
>
> Spring Cloud中文文档:https://www.bookstack.cn/read/spring-cloud-docs/docs-index.md



# 版本对应

- SpringCloud F


- SpringBoot 2.2.2.RELEASE


- SpringCloud Alibaba 2.1.0.RELEASE


- JDK1.8


- Maven 3.5及以上


- mysql 5.7及以上




# SpringCloud笔记

## 微服务理解

[微服务的理解](https://liuurick.github.io/2021/01/08/微服务的理解/)

## 服务注册与发现（Eureka）

- Eureka Server

- Eureka Client

- Eureka高可用（两节点，三节点演示）

  多个节点是两两注册

- 服务发现原理剖析

## 服务通信

- 通信机制剖析
- Feign
- Ribbon（带领分析源码，了解底层）
- RestTemplate

## 分布式配置（Spring Cloud Config）

- Config Server
- Config Client
- Git和Refresh
- 自动刷新
- SpringCloud Bus（配合RabbitMQ）

## 网关（Zuul）

- 动态路由
- Zuul高可用
- 异常网关统一处理
- 验证与安全

## 熔断 （Hystrix）

- Hystrix Dashboard
- 熔断机制
- 目的和重要性



