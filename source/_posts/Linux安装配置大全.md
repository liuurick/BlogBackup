---
title: Linux安装配置大全
date: 2020-11-27 15:26:42
tags: Linux
categories: Linux
---

Linux各种版本安装配置

<!--more-->

# CentOS 7 

[CentOS 7 安装教程](https://mp.weixin.qq.com/s?__biz=MzUxNjg4NDEzNA==&mid=2247488461&idx=1&sn=3199f4d7f6854ab450a088135122d2ff&chksm=f9a1c004ced64912098b6e1b1d4406dbcc8858a6fc7b9d9e76196e33f28b613f940dd6d682dc&sessionid=1606460612&scene=126&clicktime=1606460870&enterid=1606460870&ascene=3&devicetype=android-27&version=2700143f&nettype=3gnet&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&exportkey=A%2F6Xg4qYU%2FwWCE0t8HkjwzY%3D&pass_ticket=Mq6%2FQAWSPqK%2FgoRY6hwN80HcANi%2FWrcs4QkV5e%2BUTRSlviV9nk6vBi1LDmOauhub&wx_header=1)



# Ubuntu 20.04

[Ubuntu安装教程](https://mp.weixin.qq.com/s/vkLZ_3Jp4HdQ8PDIMYsGEw)



# CentOS7磁盘操作

## linux磁盘分区扩容

大概分为以下几步：

1. 分区fdisk
2. 格式化mkfs
3. 挂载mount

> 详细操作：https://www.cnblogs.com/chenmh/p/5096592.html



**其他操作：**

du 查看文件数据占用多少磁盘空间
swap交换分区 是一种通过在磁盘中预先划分一定的空军，然后就讲把内存中暂时不常用的数据临时存放在磁盘中，以便腾出物理内存空间让更活跃的程序服务来使用的技术，其设计目的是为了解决真实物理内存不足的问题