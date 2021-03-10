---
title: MyBatis二级缓存的一个小细节
date: 2020-10-23 19:04:22
tags: MyBatis缓存
categories: [后端,数据库,ORM框架]
---

记录一个小细节

<!--more-->

​	Mybatis的一级缓存是SqlSession级别的缓存，当在同一个SqlSession中执行两次相同的SQL语句时，会将第一次执行查询的数据存入一级缓存中，第二次查询时会从缓存中获取数据，而不用再去数据库中查询，从而提高了查询性能。

​	Mybatis的二级缓存是mapper级别的缓存，多个SqlSession共用二级缓存，他们使用的同一个mapper的SQL语句操作数据库，获得的会存放在二级缓存中。

Mybatis默认没有开启二级缓存，需要在Mybatis的配置文件mybatis-config.xml中开启，配置如下：

```xml
<settings>
	<setting name="cacheEnabled" value="true" />
</settings>
```

**注意：**`settings元素要放在properties元素之后，typeAliases元素之前，否则配置文件会报错。`