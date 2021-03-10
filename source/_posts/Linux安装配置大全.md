---
title: Linux安装配置大全
date: 2020-11-27 15:26:42
tags: Linux
categories: [开发工具,Linux系统]
---

Linux各种版本安装配置

<!--more-->

# CentOS 7 

[CentOS 7 安装教程](https://mp.weixin.qq.com/s?__biz=MzUxNjg4NDEzNA==&mid=2247488461&idx=1&sn=3199f4d7f6854ab450a088135122d2ff&chksm=f9a1c004ced64912098b6e1b1d4406dbcc8858a6fc7b9d9e76196e33f28b613f940dd6d682dc&sessionid=1606460612&scene=126&clicktime=1606460870&enterid=1606460870&ascene=3&devicetype=android-27&version=2700143f&nettype=3gnet&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&exportkey=A%2F6Xg4qYU%2FwWCE0t8HkjwzY%3D&pass_ticket=Mq6%2FQAWSPqK%2FgoRY6hwN80HcANi%2FWrcs4QkV5e%2BUTRSlviV9nk6vBi1LDmOauhub&wx_header=1)



# Ubuntu 20.04

[Ubuntu安装教程](https://mp.weixin.qq.com/s/vkLZ_3Jp4HdQ8PDIMYsGEw)





# Linux常见操作

## linux磁盘分区扩容

大概分为以下几步：

1. 分区`fdisk`
2. 格式化`mkfs`
3. 挂载`mount`

> 详细操作：https://www.cnblogs.com/chenmh/p/5096592.html



**其他操作：**

`du` 查看文件数据占用多少磁盘空间
`swap`交换分区 是一种通过在磁盘中预先划分一定的空军，然后就讲把内存中暂时不常用的数据临时存放在磁盘中，以便腾出物理内存空间让更活跃的程序服务来使用的技术，其设计目的是为了解决真实物理内存不足的问题



## 快速查找特定文件

```
find 路径 -name "target.java"  :精确查找文件
find 路径 -name "target*"  :模糊查找文件
find 路径 -iname "targe*"  :不区分文件名大小写法去查找文件
man find：更多关于find指令的使用说明
```

## 检索文件内容

`grep` (global search regular expression)：用于查找文件里符合条件的字符串。

```
grep [-abcEFGhHilLnqrsvVwxy][-A<显示行数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]
```

管道操作符 `|`  ：可将指令连接起来，前一个指令的输出作为后一个指令的输入

> https://www.runoob.com/linux/linux-comm-grep.html

## 对日志内容做统计

`awk`：是一种处理文本文件的语言，是一个强大的文本分析工具。

- 一次读取一行文本，按输入分隔符进行切片，切成多个组成部分
- 将切片直接保存在内建的变量中，$1,$2....($0表示行的全部)
- 支持对单个切片的判断

```bash
awk [选项参数] 'script' var=value file(s)
或
awk [选项参数] -f scriptfile var=value file(s)
```

> https://www.runoob.com/linux/linux-comm-awk.html

## 批量替换文件内容

`sed`：利用脚本来处理文本文件。

```
sed [-hnV][-e<script>][-f<script文件>][文本文件]
```

> https://www.runoob.com/linux/linux-comm-sed.html