---
title: ElasticSearch集群搭建
date: 2020-10-11 10:55:50
tags: ElasticSearch集群
categories: [后端,搜索引擎,ElasticSearch]
---

ElasticSearch诞生于2010年，它采用的是分布式模式，这也印证了它名字中的“elastic”。谈到分布式，当然离不开集群，这里记录一下自己搭建集群的经验。

<!--more-->

### 一、版本

- elasticsearch-7.7.1-windows-x86_64

- elasticsearch-head-5.0.0



### 二、搭建

搭建主要分为三步：解压文件--》修改配置文件--》启动

#### 1.解压文件

这里我是将文件解压到一个新的文件夹，注意文件路径名不要带有中文名。

![image-20201011112800198](/images/2020101101.png)

#### 2.修改配置文件

修改config文件中的elasticsearch.yml

```yml
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
cluster.name: XXX
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
node.name: node-1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
#path.data: /path/to/data
#
# Path to log files:
#
#path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: 127.0.0.1
#
# Set a custom port for HTTP:
#
http.port: 9200

#集群之间的通信
transport.tcp.port: 9300 

#允许前端做跨域访问
http.cors.enabled: true
http.cors.allow-origin: "*"
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.seed_hosts: ["127.0.0.1:9300", "127.0.0.1:9301","127.0.0.1:9302"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
cluster.initial_master_nodes: ["127.0.0.1:9300", "127.0.0.1:9301","127.0.0.1:9302"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true


```

因为是单机搭建集群，这里是做的伪分布式配置，集群的区分主要是通过端口

```yml
#集群名字
cluster.name

#当前配置所在机器的节点名
node.name

#设置其它节点和该节点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址
network.host

#设置对外服务的http端口
http.port

#设置节点之间交互的tcp端口，默认是9300。
transport.tcp.port

# 在启动此节点时传递要执行发现的主机的初始列表
discovery.seed_hosts

# 使用初始的一组符合主节点条件的节点引导集群
cluster.initial_master_nodes
```

另外两个节点更改一下端口即可

#### 3.启动集群

node1:

![node1](/images/2020101102.png)

node2:

![node2](/images/2020101103.png)

node3:

![node3](/images/2020101104.png)

总览：

![nodes](/images/2020101105.png)

