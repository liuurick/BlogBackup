---
title: ElasticSearch笔记
date: 2020-12-02 19:55:02
tags: ElasticSearch
categories: ElasticSearch
---

# ElasticSearch原理并部署

## 搜索原理





搜索

分词

倒排索引





ES是什么

独立的网络上的一个或一组进程节点

对外提供搜索服务（http/transport协议）

对内就是一个搜索数据库



## 名词定义

索引=数据库

类型=表

文档=行数据



![image-20201204150702449](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201204150702449.png)



![image-20201204152921640](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201204152921640.png)

分词     

搜索是以词为单位做最基本的搜索单元

依靠分词器构造分词    英文分词器：空格分词器

用分词构建倒排索引





正向索引

![image-20201204153757270](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201204153757270.png)



倒排索引

![image-20201204153725464](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201204153725464.png)



TF-IDF打分

TF：词频 这个document文档包含了多少个这个词，包含越多表明越相关

DF：文档频率 包含改词的文档总数目

IDF：DF取反



## 分布式原理

分片



主从



路由







# ElasticSearch基础语法及应用







# ElasticSearch高级语法及应用