---
title: ElasticSearch搜索优化
date: 2020-12-09 17:21:11
tags: ElasticSearch7
categories: ElasticSearch
---

ElasticSearch搜索优化主要通过三个方面：

1. 定制化分词器
2. 同义词扩展
3. 相关性重塑

<!--more-->

# 定制化分词器

对于中文分词器，我们一般使用的是`elasticsearch-analysis-ik`，当然也可以在现成的分词器上加一些定制化的改造，让分词器更加“智能”。

## 1.创建dic文件

在es的node1节点创建`myword.dic`文件

然后将文件复制到其余的两个节点



## 2.修改es配置文件

`IKAnalyzer.cfg.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">myword.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>

```

将文件复制到其余的两个节点





## 3.热更新词库

```xml
<entry key="ext_dict">http://yoursite.com/getCustomDic</entry>
```

>Http请求需要返回两个头部，last-modified和etag
>
>两者任何一个发生变化会重新更新，ik一分钟检测一次





# 同义词扩展

同义词：

> 语义相近或相同
>
> 品牌类目关联
>
> 搜索它等于搜索它



## 1.创建同义词文件

在集群的各节点上的该路径`elasticsearch\elasticsearch-node3\config\analysis-ik`加上同义词文件

![image-20201209164003792](/images/202012091601.png)



## 2.建立es索引

```
#加入同义词
PUT /myshop
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1,
    "analysis": {
      "filter": {
        "my_synonym_filter":{
          "type":"synonym",
          "synonyms_path":"analysis-ik/my_synonyms.txt"
        }
      },
      "analyzer":{
        "ik_syno":{
          "type":"custom",
          "tokenizer":"ik_smart",
          "filter":["my_synonym_filter"]
        },
        "ik_syno_max":{
          "type":"custom",
          "tokenizer":"ik_max_word",
          "filter":["my_synonym_filter"]
        }
      }
    }
  },
    
  "mappings": {
    "properties": {
      "id":{
        "type": "integer"
      },
      "name":{
        "type": "text",
        "analyzer": "ik_syno_max",
        "search_analyzer": "ik_syno"
      },
      "tags":{
        "type": "text",
        "analyzer": "whitespace","fielddata": true
      },
      "location":{
        "type": "geo_point"
      },
      "remark_score":{
        "type": "double"
      },
      "price_per_man":{
        "type": "integer"
      },
      "category_id":{
        "type": "integer"
      },
      "category_name":{
        "type": "text"
      },
      "seller_id":{
        "type": "integer"
      },
      "seller_name":{
        "type": "text"
      },
      "seller_remark_score":{
        "type": "double"
      },
      "seller_disabled_flag":{
        "type": "integer"
      }
    }
  }
}

```

## 3.同步数据

通过logstash将数据同步到es中

```
logstash -f mysql/jdbc.conf
```



## 4.测试

分词测试：

```
GET /shop/_analyze
{
  "field": "name"
  , "text": "xxx"
}
```



索引测试：

```
GET /shop/_search
{
  "query": {
    "match": {
      "name": "xxx"
    }
  }
}
```







# 相关性重塑

> 什么是相关性：https://www.elastic.co/guide/cn/elasticsearch/guide/current/relevance-intro.html



相关性搜索

让搜索引擎理解语义

影响召回及排序



## 重新构建es索引

采取词性影响召回策略模型 通过category_id



## 引入Java代码：

构造分类：

```java
private Integer getCategoryIdByToken(String token) {
    for (Integer key : categoryWorkMap.keySet()) {
        List<String> tokenList = categoryWorkMap.get(key);
        if (tokenList.contains(token)) {
            return key;
        }
    }
    return null;
}

private Map<Integer, List<String>> categoryWorkMap = new HashMap<>();

@PostConstruct
public void init() {
    categoryWorkMap.put(1, new ArrayList<>());
    categoryWorkMap.put(2, new ArrayList<>());

    categoryWorkMap.get(1).add("吃饭");
    categoryWorkMap.get(1).add("下午茶");

    categoryWorkMap.get(2).add("休息");
    categoryWorkMap.get(2).add("睡觉");
    categoryWorkMap.get(2).add("住宿");

}
```

在构造分词函数识别器时调用`getCategoryIdByToken`方法

```java
/**
 * 构造分词函数识别器
 * @param keyword
 * @return
 * @throws IOException
 */
private Map<String, Object> analyzeCategoryKeyword(String keyword) throws IOException {
    Map<String, Object> res = new HashMap<>();

    Request request = new Request("GET", "/shop/_analyze");
    request.setJsonEntity("{" + "  \"field\": \"name\"," + "  \"text\":\"" + keyword + "\"\n" + "}");
    Response response = highLevelClient.getLowLevelClient().performRequest(request);
    String responseStr = EntityUtils.toString(response.getEntity());
    JSONObject jsonObject = JSONObject.parseObject(responseStr);
    JSONArray jsonArray = jsonObject.getJSONArray("tokens");
    for (int i = 0; i < jsonArray.size(); i++) {
        String token = jsonArray.getJSONObject(i).getString("token");
        Integer categoryId = getCategoryIdByToken(token);
        if (categoryId != null) {
            res.put(token, categoryId);
        }
    }
    return res;
}
```