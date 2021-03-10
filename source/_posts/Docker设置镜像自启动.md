---
title: Docker设置镜像自启动
date: 2020-11-09 22:26:44
tags: Docker
categories: [后端,分布式/微服务,虚拟化/容器化]
---

Docker便捷小技巧

<!--more-->

1.发现没启动

`docker ps `

2.查看所有安装的镜像

`docker ps -a`

3.设置总是自动启动

`docker update 镜像id --restart=always`

4.重启测试一下