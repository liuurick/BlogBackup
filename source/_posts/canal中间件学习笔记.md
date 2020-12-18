---
title: canal中间件学习笔记
date: 2020-12-09 15:05:40
tags: [ElasticSearch7,canal]
categories: canal
---

es准时性索引的构建

<!--more-->

# 增量索引构建的要求

- 准实时；
- 性能；
- 编程简化；

# canal 简介

>canal:https://github.com/alibaba/canal
>
>canal同步数据：https://mp.weixin.qq.com/s/zaV98KHVyXUW32DqypU1OQ

- 消息管道；
- source 为 MySQL 数据库；
- target 为其他存储，比如 ElasticSearch；
- Canal 借鉴了MySQL主从复制的原理，伪装成一个 MySQL 主库的备库，感知 MySQL 中 binlog 的变化，并同步出来一个结构化的数据，交给 Canal 管道的消费端；

![img](C:\Users\admin\Desktop\blog\source\images\2020121201.png)

# 工作原理

## MySQL主备复制原理

![img](C:\Users\admin\Desktop\blog\source\images\2020121202.png)

- MySQL master 将数据变更写入二进制日志( binary log, 其中记录叫做二进制日志事件binary log events，可以通过 show binlog events 进行查看)
- MySQL slave 将 master 的 binary log events 拷贝到它的中继日志(relay log)
- MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据

## canal 工作原理

- canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送dump 协议
- MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )
- canal 解析 binary log 对象(原始为 byte 流)



> canal快速启动：https://github.com/alibaba/canal/wiki/QuickStart



# Linux下安装canal

## 开启 MySQL 的 binglog

###### Ubuntu 用 APT Repository 安装 MySQL 的安装位置

```
/etc/mysql - 配置文件
/var/lib/mysql - 数据存放的位置
/usr/bin/mysql - 启动命令
/usr/lib/mysql -  插件
```



###### 查看 MySQL 几个变量的值

```
要保证 binlog 开启，并且格式是 ROW；

如果不是上述配置的话，需要修改 `/etc/mysql/mysql.conf.d/mysqld.cnf`，然后还要给 MySQL 实例起个 `server-id = 1`；MySQL 8 的话，两个参数默认就是这样的，然后 `server-id` 给个唯一值就行了；
```



```bash
# 是否开启 binlog
show variables like 'log_bin';
# binlog 的格式
show variables like 'binlog_format';
```



###### 创建复制用户

```csharp
create user 'canal'@'%' identified by 'canal';

grant replication slave on *.* to 'canal'@'localhost';
grant replication client on *.* to 'canal'@'localhost';
grant select on *.* to 'canal'@'localhost';

flush privileges;
```



## canal 配置 | 1.1.3

###### 修改 canal.properties

- 文件路径：`/canal/1.1.3/canal.deployer-1.1.3/conf`；
- 这一行注释掉；



```bash
#canal.instance.tsdb.spring.xml = classpath:spring/tsdb/h2-tsdb.xml
```

###### 修改 instance.properties

````
位置：/canal/1.1.3/canal.deployer-1.1.3/conf/example/instance.properties；
````

修改如下：

```undefined
canal.instance.mysql.slaveId=8

canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
```

###### 启动 canal

JDK 换成 8 才启动起来；

```undefined
bin/startup.sh
```

## canal.adapter | 1.1.4

###### 下载源码

```
在 IDEA 中打开 client-adapter module；

修改 /canal-canal-1.1.4/client-adapter/elasticsearch/pom.xml` 中的 ElasticSearch 的依赖版本，改成 7.3.0；
```

pom.xml:

```xml
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>7.3.0</version>
</dependency>
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>transport</artifactId>
    <version>7.3.0</version>
</dependency>
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>7.3.0</version>
</dependency>
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.3.0</version>
</dependency>
```

###### 在总 module 的目录下重新打包 canal

- 路径为：`canal-canal-1.1.4`；
- 打包命令：`mvn clean package -DskipTests`；
- 报错：`[ERROR] /canal-canal-1.1.4/client-adapter/elasticsearch/src/main/java/com/alibaba/otter/canal/client/adapter/es/support/ESConnection.java:[420,47] 无法将类 org.elasticsearch.client.RestHighLevelClient中的方法 bulk应用到给定类型;`
- 找到指定的 ESConnection.java 的 420 行，解决问题：`return restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);`
- 重新打包：`mvn clean package -DskipTests`；
- 报错：`/canal-canal-1.1.4/client-adapter/elasticsearch/src/main/java/com/alibaba/otter/canal/client/adapter/es/ESAdapter.java:[225,56] 不兼容的类型: org.apache.lucene.search.TotalHits无法转换为long`；
- 找到指定的 ESAdapter.java 的 225 行，解决问题：`long rowCount = response.getHits().getTotalHits().value;`；
- 重新打包：`mvn clean package -DskipTests`，终于成功了；

###### 进入编译后生成的目录

- `/canal-canal-1.1.4/client-adapter/launcher/target/canal-adapter`，里面的内容和直接从 Github 上下载二进制的包是一样的;
- 把这个目录拷到正经的目录下，并改个名字 `/canal/1.1.4/canal-adapter-es7`；

###### 修改配置文件 application.yml

- 文件路径：`/canal/1.1.4/canal-adapter-es7/conf/application.yml`；
- 这个配置主要是指明管道两端的 MySQL 和 ElasticSearch；



```ruby
server:
  port: 8081
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    default-property-inclusion: non_null

canal.conf:
  mode: tcp 
  canalServerHost: 127.0.0.1:11111
  batchSize: 500
  syncBatchSize: 1000
  retries: 0
  timeout:
  accessKey:
  secretKey:
  srcDataSources:
    defaultDS:
      url: jdbc:mysql://127.0.0.1:3306/dianping?useUnicode=true&useSSL=false
      username: canal
      password: canal
  canalAdapters:
  - instance: example 
    groups:
    - groupId: g1
      outerAdapters:
      - name: logger
      - name: es
        hosts: 127.0.0.1:9300
        properties:
          cluster.name: dianping-app
```

###### 创建文件 shop.yml

- 文件路径：`/canal/1.1.4/canal-adapter-es7/conf/es/shop.yml`；
- 其中 `defaultDS` 和   `example` 分别对应 application.yml 中的 `defaultDS` 和   `example`；



```objectivec
dataSourceKey: defaultDS
destination: example
groupId: 
esMapping:
  _index: shop
  _type: _doc
  _id: id
  upsert: true
  sql: "select a.id,a.name,a.tags,concat(a.latitude,',',a.longitude) as location,a.remark_score,a.price_per_man,a.category_id,b.name as category_name,a.seller_id,c.remark_score as seller_remark_score,c.disabled_flag as seller_disabled_flag from shop a inner join category b on a.category_id = b.id inner join seller c on c.id = a.seller_id"
  commitBatch: 3000
```

###### 启动 canal-adapter-es7

- MySQL 8 要把 `/canal/1.1.4/canal-adapter-es7/lib` 下的驱动换一下；
- 启动命令：`bin/startup.sh`；

###### 启动编译好的 canal-deployer-es7

- canal.adapter-1.1.4 和 canal.deployer-1.1.3 是不兼容的，所以要把之前启动的 canal.deployer-1.1.3 换成 Maven 编译好的 canal.deployer-1.1.4；
- 编译好的原路径：`/canal-canal-1.1.4/deployer/target/canal`，更改到新路径：`/canal/1.1.4/canal-deployer-es7`；
- 先关停 canal.deployer-1.1.3：`bin/stop.sh`；
- 把 canal.deployer-1.1.3 的 canal.properties 和 example/ 拷贝到 canal-deployer-es7 中；
- 启动 canal-deployer-es7：`bin/startup.sh`；

###### 更新 MySQL 中 dianping 库的 shop 表

- 日志文件 `canal-adapter-es7/logs/adapter/adapter.log` 立马可以感知到；
- 在 ElasticSearch 中查询，也可以立马感知到；



# Windows下安装canal

## 开启 MySQL 的 binglog

1.查看mysql的`log_bin`是否开启。

因为canal是通过`log_bin`监听更新的

```
show variables like '%log_bin%'
```

如果value为off需要修改my.ini文件

**注意**：可能会出现MySQL目录下找不到 my.ini 的问题

```
1.在Windows窗口栏查看-》勾选隐藏的项目
2.找到ProgramData目录
3.在ProgramData目录中找到MySQL
```



2.修改my.ini文件

```
server_id=1 ###代表集群模式第一台机器
binlog_format=ROW ###行模式
log_bin=mysql_bin.log ###binlog的文件名称
expire-logs-days=14 ###失效日期为14天（可不配置）
max-binlog-size=500M ###最大存储500MB（可不配置）
```



重启MySQL之后再次查询：

![image-20201212190859514](C:\Users\admin\Desktop\blog\source\images\2020121203.png)





## 编译canal-adapter

因为canal的es默认版本并不匹配,所以需要重新编译

**ESAdapter类中的类型错误**

```
long rowCount = response.getHits().getTotalHits().value;
```

**ESConnection**

```
public BulkResponse bulk() {
            if (mode == ESClientMode.TRANSPORT) {
                return bulkRequestBuilder.execute().actionGet();
            } else {
                try {
                    return restHighLevelClient.bulk(bulkRequest,RequestOptions.DEFAULT);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
```



## 安装canal-adapter

1.修改conf文件

```
srcDataSources:
    defaultDS:
    url: jdbc:mysql://127.0.0.1:3306/dianpingdb?useUnicode=true
    username: root
    password: 123456
```



```
- name: es
        hosts: 127.0.0.1:9300 # 127.0.0.1:9200 for rest mode
        properties:
          mode: transport # or rest
          # security.auth: test:123456 #  only used for rest mode
          cluster.name: elasticsearch
```



2.修改es目录下的文件

创建shop.yml文件

```
dataSourceKey: defaultDS
destination: example
groupId: 
esMapping:
  _index: shop
  _type: _doc
  _id: id
  upsert: true
  sql: "select a.id,a.name,a.tags,concat(a.latitude,',',a.longitude) as location,a.remark_score,a.price_per_man,a.category_id,b.name as category_name,a.seller_id,c.remark_score as seller_remark_score,c.disabled_flag as seller_disabled_flag from shop a inner join category b on a.category_id = b.id inner join seller c on c.id = a.seller_id"
  commitBatch: 3000
```

