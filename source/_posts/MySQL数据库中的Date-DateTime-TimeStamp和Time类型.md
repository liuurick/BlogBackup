---
title: 'MySQL数据库中的Date,DateTime,TimeStamp和Time类型'
date: 2020-11-18 19:52:52
tags: MySQL日期类型
categories: [编程基础,MySQL]
---

**DATETIME**类型用在你需要同时包含日期和时间信息的值时。MySQL检索并且以'YYYY-MM-DD HH:MM:SS'格式显示DATETIME值，支持的范围是'1000-01-01 00:00:00'到'9999-12-31 23:59:59'。（“支持”意味着尽管更早的值可能工作，但不能保证他们可以。）

<!--more-->

**DATE**类型用在你仅需要日期值时，没有时间部分。MySQL检索并且以'YYYY-MM-DD'格式显示DATE值，支持的范围是'1000-01-01'到'9999-12-31'。



**TIMESTAMP**列类型提供一种类型，你可以使用它自动地用当前的日期和时间标记INSERT或UPDATE的操作。



**TIME**数据类型表示一天中的时间。MySQL检索并且以"HH:MM:SS"格式显示TIME值。支持的范围是'00:00:00'到'23:59:59'。



创建数据库表测试：

![image-20201115191901682](/images/2020111501.png)

![img](http://www.linuxidc.com/upload/2012_08/120811102669351.jpg)



**datetime和timestamp的区别：**
1.datetime 的日期范围比较大；如果有1970年以前的数据还是要用datetime.但是timestamp 所占存储空间比较小。
2.timestamp 类型的列还有个特性：默认情况下，在 insert, update 数据时，timestamp 列会自动以当前时间（CURRENT_TIMESTAMP）填充/更新。

3.timestamp比较受时区timezone的影响以及MYSQL版本和服务器的**SQL** MODE的影响.

使用一个常用的格式集的任何一个，你可以指定DATETIME、

**DATE和TIMESTAMP值：**
'YYYY-MM-DD HH:MM:SS'或'YY-MM-DD HH:MM:SS'格式的一个字符串,允许一种"宽松"的语法:任何标点可用作在日期部分和时间部分之间的分隔符。例如，'98-12-31 11:30:45'、'98.12.31 11+30+45'、'98/12/31 11*30*45'和'98@12@31 11^30^45'是等价的。