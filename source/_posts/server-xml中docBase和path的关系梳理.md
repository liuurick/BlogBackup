---
title: server.xml中docBase和path的关系梳理
date: 2019-02-13 19:03:35
tags: tomcat
categories: [后端,服务器软件,应用服务器]
---

今天项目遇到这么个问题，想通过第三方存储的方式获取图片，服务器中显示路径为本地路径+数据库中存储的路径。

<!--more-->

主要有两种方案：

- 通过tomcat的server.xml文件配置
- 通过IDEA设置虚拟路径



**方案一：**

在tomcat的server.xml文件中加入配置语句

![image](/images/2019031501.png)这样当我们读取到这个/upload时就会转为 D:/upload，之后加上后面的路径凑成完成绝对路径，进行访问

然后设置idea的tomcat，勾选`Deploy applications configured in Tomcat instance`

![image](/images/2019031502.png)

 

**方案二：**

直接用idea设置虚拟路径。

取消勾选`Deploy applications configured in Tomcat instance`

在`Deployment`设置虚拟路径--->`External Source`-->选择替换的路径--->在Application context输入被替换的路径即可
