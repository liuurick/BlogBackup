---
title: ElasticSearch7笔记
date: 2020-12-02 19:55:02
tags: ElasticSearch
categories: ElasticSearch
---

ElasticSearch7学习笔记

<!--more-->

# ElasticSearch原理及部署

> 中文官网：https://www.elastic.co/cn/elasticsearch/
>
> 阮一峰：http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html

## 搜索原理

ElasticSearch是独立的网络上的一个或一组进程节点，对外提供搜索服务（http/transport协议），对内就是一个搜索数据库。

对于搜索，是以词为单位做最基本的搜索单元，依靠分词器构造分词（其中英文分词器有空格分词器），然后用分词构建倒排索引。



![image-20201204152921640](/images/2020120401.png)



正向索引

![image-20201204193628122](/images/2020120402.png)



倒排索引

![image-20201204193651906](/images/2020120403.png)





试想根据某个词查找，查找出一堆文档，那到底是哪个匹配度更高呢，这时候就需要打分的逻辑

- TF：词频，这个document文档包含了多少个这个词，包含越多表明越相关
- DF：文档频率，包含该词的文档总数目
- IDF：DF取倒数
- 打分常用计算公式：TF * IDF



## 名词定义

关系型数据库MySQL对应ElasticSearch：

1. MySQL 中的数据库（DataBase），等价于 ES 中的索引（Index）。
2. MySQL 中一个数据库下面有 N 张表（Table），等价于1个索引 Index 下面有 N 多类型（Type）。
3. MySQL 中一个数据库表（Table）下的数据由多行（Row）多列（column，属性）组成，等价于1个 Type 由多个文档（Document）和多 Field 组成。
4. MySQL 中定义表结构、设定字段类型等价于 ES 中的 Mapping。举例说明，在一个关系型数据库里面，Schema 定义了表、每个表的字段，还有表和字段之间的关系。与之对应的，在 ES 中，Mapping 定义索引下的 Type 的字段处理规则，即索引如何建立、索引类型、是否保存原始索引 JSON 文档、是否压缩原始 JSON 文档、是否需要分词处理、如何进行分词处理等。
5. MySQL 中的增 insert、删 delete、改 update、查 search 操作等价于 ES 中的增 PUT/POST、删 Delete、改 _update、查 GET。其中的修改指定条件的更新 update 等价于 ES 中的 update_by_query，指定条件的删除等价于 ES 中的 delete_by_query。
6. MySQL 中的 group by、avg、sum 等函数类似于 ES 中的 Aggregations 的部分特性。
7. MySQL 中的去重 distinct 类似 ES 中的 cardinality 操作。
8. MySQL 中的数据迁移等价于 ES 中的 reindex 操作。



![img](/images/2020120404.png)







## 分布式原理

https://www.cnblogs.com/jajian/p/10176604.html





## 集群搭建

[https://liuurick.github.io/2020/10/11/ElasticSearch%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA/](https://liuurick.github.io/2020/10/11/ElasticSearch集群搭建/)



# ElasticSearch基础语法及应用

## 索引创建，更新，删除

> 结构化索引，类似MySQL，我们会对索引结构做预定义，包括字段名，字段类型等；
>
> 非结构化索引，就类似Mongo，索引结构未知，根据具体的数据来update索引的mapping。
>
> 那么如何选择两种索引呢，还是跟具体的使用场景有关，结构化相比非结构化，更易优化，性能好些，非结构化相较灵活，只是频繁update索引mapping会有一定的性能损耗。

```
#1.索引创建
PUT /mytest
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  }
}

#2.索引删除
DELETE /mytest
```



## 索引简单语句

```
#3.非结构化方式新建索引
PUT /employee/_doc/1
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "name": "liubin",
  "age": 24
}

#4.获取索引记录
GET /employee/_doc/1

#5.指定字段修改
POST /employee/_update/1
{
  "doc":{
    "name": "刘斌"
  }
}

#6.强制指定创建，若已经存在，则失败
POST /employee/_create/2
{
  "name":"123",
  "age": 30
}

#7.删除某个文档
DELETE /employee/_doc/1

#8.查询全部文档
GET /employee/_search
```



```
#9.使用结构化的方式创建索引
PUT /employee
{
 "settings": {
   "number_of_shards": 1,
   "number_of_replicas": 1
 } ,
 "mappings": {
   "properties": {
     "name":{"type": "text"},
     "age":{"type": "integer"}
   }
 } 
}

#10.添加数据
POST /employee/_create/1
{
  "name":"我是打工人",
  "age": 30
}

#11.不带条件查询所有记录
GET /employee/_search

GET /employee/_search
{
  "query": {
    "match_all": {}
  }
}

#12.分页查询
GET /employee/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 1
}
```





## 索引复杂语句

```
#13.带关键字条件的查询
GET /employee/_search
{
  "query": {
    "match": {
      "name": "工人"
    }
  }
}

#14.带排序
GET /employee/_search
{
  "query": {
    "match": {
      "name": "工人"
    }
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ]
}

#15.带filter 
GET /employee/_search
{
  "query": {
    "bool": {
      "filter": [
        {"term": {
          "age": "30"
        }}
      ]
    }
  }
}

#16.带聚合
GET /employee/_search
{
  "query": {
    "match": {
      "name": "工"
    }
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ],
  "aggs": {
    "group_by_age": {
      "terms": {
        "field": "age"
      }
    }
  }
}
```



>match和term区别：https://www.cnblogs.com/yjf512/p/4897294.html
>
>match 分词查询
>term 整体查询



# ElasticSearch高级语法及应用

更着重于分词相关的操作

## analyze分析过程

`analyse分析=分词的过程：字符过滤器-->字符处理-->分词过滤（分词转换，词干转化）`

- 字符过滤：使用字符过滤器转变字符。
- 文本切分为分词：将文本（档）分为单个或多个分词。
- 分词过滤：使用分词过滤器转变每个分词。
- 分词索引：最终将分词存储在Lucene倒排索引中。

![20190824003615611858](/images/2020120405.png)



利用analyze api搜索

先建立索引

```
PUT /employee/_doc/1
{
  "name": "Eating an apple a day & keeps the doctor away", 
  "age": 30
}
```



然后搜索

```
GET /employee/_search
{
 "query":{
  "match": {"name":"eat"}
 }
}
```



没搜到后使用analyze api查看分析处理结果，可以看到没有分出eat，所以搜不到，改成用english分词器做索引

```
GET /employee/_analyze
{
 "field":"name",
 "text":"Eating an apple a day & keeps the doctor away"
}
```



重新创建索引

```
PUT /employee
{
  "settings" : {
   "number_of_shards" : 1,
   "number_of_replicas" : 1
  },
  "mappings" : {
    "properties" : {
      "name" : { 
        "type" : "text","analyzer": "english"},
        "age" : {"type":"integer"}
      }
  }
}
```



在用`analyze api`，可以看到eat

```
GET /employee/_analyze
{
 "field":"name",
 "text":"Eating an apple a day & keeps the doctor away"
}
```

## 相关性查询手段



类型：

> https://blog.csdn.net/chengyuqiang/article/details/79048800



