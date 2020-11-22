---
title: 快速构建Java Web项目的开发环境
date: 2020-11-20 17:48:18
tags: Java Web
categories: Java Web
---

Idea通过maven快速构建项目

<!--more-->

# 1.搭建CRM项目的开发环境

- 设置字体
- 设置工作区的字符集为utf-8
- 设置basePath变量及base标签

```xml
<%
String basePath = request.getScheme() + "://" + request.getServerName() + ":" +   request.getServerPort() + request.getContextPath() + "/";
%>
<base href="<%=basePath%>">
```



# 2.创建Maven项目前的准备工作

使用settings引入本地仓库![image-20201120175318348](/images/2020112001.png)



打开settings.xml设置本地仓库位置

![](/images/2020112002.png)

 

创建Maven web项目

![image-20201120175508091](/images/2020112003.png)

![image-20201120175542687](/images/2020112004.png)



 

新创建的Maven项目，目录结构如下：

![image-20201120175616811](/images/2020112005.png)



将pom.xml中没有用的信息去除掉



**在main文件夹下建立java文件夹和resources文件夹**

**与main文件夹平级创建一个test文件夹**

**test文件夹下创建java文件夹和resources文件夹**

![image-20201120175810912](/images/2020112006.png)

**注意：为文件夹赋予功能（颜色）后，文件夹才会生效**

 

将我们自己的web.xml模板导入

通过pom.xml导入我们需要的jar包



在pom.xml中加入构建资源

```xml
</build>
    <resources>
        <resource>
            <directory>src/main/java</directory><!--所在的目录-->
            <includes><!--包括目录下的.properties,.xml文件都会扫描到-->
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```



点击左侧Maven Projects，点击刷新

![image-20201120180057366](/images/2020112007.png)



在resource路径下引入开发用的配置文件和属性文件

jdbc.properties（修改数据库名）

log4j.properties

mybatis-config.xml

SqlMapper.xml（建立后台包结构后加入该文件）



项目文件结构如下：

![image-20201120180226721](/images/2020112008.png)

 

# 3.创建一个数据库

![image-20201120180410393](/images/2020112009.png)



* 将项目原型拷贝到webapp目录下

* 启动Tomcat服务器，测试