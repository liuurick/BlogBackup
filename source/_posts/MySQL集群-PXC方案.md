---
xmltitle: MySQL集群-PXC方案
date: 2020-8-21 20:24:44
tags: [MySQL集群,Docker]
categories: MySQL
---

常见的集群方案包括：Replication，Percona XtraDB Cluster（PXC）

<!--more-->

| Replication                                                  | PXC                                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 数据同步是单向的，master负责写，然后异步复制给slave；如果slave写入数据，不会复制给master。 | 数据同步时双向的，任何一个mysql节点写入数据，都会同步到集群中其它的节点。 |
| 异步复制，从和主无法保证数据的一致性                         | 同步复制，事务在所有集群节点要么同时提交，要么同时不提交     |

Replication方案适用于日志，博客这样的网站，存储一些价值较低的内容。

而PXC具备强一致性，数据同步是双向的特点，另外PerconaServer是MySQL改进版，性能提升很大。所以这里我选择的是PXC方案搭建Mysql集群。

### 一、搭建准备

这里我是尝试通过Docker搭建的。Docker的安装与配置非常简单。

#### 1.安装docker

```
yum -y update
yum install -y docker
```

#### 2.配置docker镜像加速器

这里我使用的阿里云的加速器

#### 3.docker常用命令

```
docker search   #查找镜像
docker images	#查看所有镜像
docker pull		#下载镜像
docker rmi		#删除镜像
docker ps -a	#查看正在运行的实例
```



### 二、Mysql PXC集群环境部署

#### 1.下载镜像

```
docker pull docker.io/percona/percona-xtradb-cluster
```

**注意：**可以使用`docker tag`改名，这里我将 docker.io/percona/percona-xtradb-cluster更名为pxc

#### 2.创建内部网络

出于安全考虑，需要给PXC集群实例创建Docker内部网络

这里我搭建集群使用的是**5节点**，网段可以自己规定

```
docker network create --subnet=172.10.0.0/24 net1
docker network inspect net1
docker network rm net1
```

#### 3.创建Docker卷

容器中的PXC节点映射数据目录的解决方法：

```
docker volume create --name v1
docker volume create --name v2
docker volume create --name v3
docker volume create --name v4
docker volume create --name v5
```

查看docker卷信息：`docker inspect v1`

#### 4.创建PXC容器

5个PXC容器

```
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456 -v v1:/var/lib/mysql --privileged --name=node1 --net=net1 --ip 172.10.0.2 pxc 

docker run -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456 -e CLUSTER_JOIN=node1 -v v2:/var/lib/mysql --privileged --name=node2 --net=net1 --ip 172.10.0.3 pxc

docker run -d -p 3308:3306 -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456 -e CLUSTER_JOIN=node1 -v v3:/var/lib/mysql --privileged --name=node3 --net=net1 --ip 172.10.0.4 pxc 

docker run -d -p 3309:3306 -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456 -e CLUSTER_JOIN=node1 -v v4:/var/lib/mysql --privileged --name=node4 --net=net1 --ip 172.10.0.5 pxc

docker run -d -p 3310:3306 -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456 -e CLUSTER_JOIN=node1 -v v5:/var/lib/mysql --privileged --name=node5 --net=net1 --ip 172.10.0.6 pxc
```

### 三、验证是否创建成功PCX集群

在数据库中创建5个DB：

![image-20201102100706388](/images/2020110201.png)

在DB1中新建test数据库并创建student表：

![image-20201102100820614](/images/2020110202.png)

刷新之后，可以看到DB2,DB3,DB4,DB5全部同步到与DB1一致

![image-20201102100858654](/images/2020110203.png)



### 四、数据库负载的配置

数据库负载均衡的必要性：

虽然搭建了集群，但是不使用数据库负载均衡，单节点处理所有请求，会暴露负载高，性能差的问题

这里可以使用**Haproxy**做负载均衡，将请求均匀的分发给每个节点，单节点负载会轻松很多。

#### 1.haproxy的下载

`docker pull haproxy`

#### 2.创建haproxy配置文件

配置参考：http://zhangge.net/5125.html

在/home/soft/haproxy/目录下创建haproxy.cfg文件

haproxy.cfg

```xml
global
	#工作目录
	chroot /usr/local/etc/haproxy
	#日志文件，使用rsyslog服务中local5日志设备（/var/log/local5），等级info
	log 127.0.0.1 local5 info
	#守护进程运行
	daemon

defaults
	log global
	mode http
	#日志格式
	option httplog
	#日志中不记录负载均衡的心跳检测记录
	option dontlognull
	#连接超时（毫秒）
	timeout connect 5000
	#客户端超时（毫秒）
	timeout client 50000
	#服务器超时（毫秒）
	timeout server 50000

#监控界面
listen admin_stats
	#监控界面的访问的IP和端口
	bind 0.0.0.0:8888
	#访问协议
	mode http
	#URI相对地址
	stats uri /dbs
	#统计报告格式
	stats realm     Global\ statistics
	#登陆帐户信息
	stats auth  admin:123456
 #数据库负载均衡
 listen  proxy-mysql
	#访问的IP和端口
	bind  0.0.0.0:3306
	#网络协议
	mode  tcp
	#负载均衡算法（轮询算法）
	#轮询算法：roundrobin
	#权重算法：static-rr
	#最少连接算法：leastconn
	#请求源IP算法：source
	balance  roundrobin
	#日志格式
	option  tcplog
   	#在MySQL中创建一个没有权限的haproxy用户，密码为空。Haproxy使用这个账户对MySQL数据库心跳检测
        option  mysql-check user haproxy
        server  MySQL_1 172.10.1.2:3306 check weight 1 maxconn 2000
        server  MySQL_2 172.10.1.3:3306 check weight 1 maxconn 2000
        server  MySQL_3 172.10.1.4:3306 check weight 1 maxconn 2000
        server  MySQL_4 172.10.1.5:3306 check weight 1 maxconn 2000
        server  MySQL_5 172.10.1.6:3306 check weight 1 maxconn 2000
	#使用keepalive检测死链
	option  tcpka
```



#### 4.实例化haproxy

```xml
docker run -it -d -p 4001:8888 -p 4002:3306 -v /home/soft/haproxy:/usr/local/etc/haproxy --name h1 --privileged --net=net1 --ip 172.10.0.7 haproxy
```

#### 5.登陆到交互容器里

```
docker exec -it h1 bash
haproxy -f /usr/local/etc/haproxy/haproxy.cfg
```

#### 6.测试

![image-20201102120345130](/images/2020110204.png)

首页展示：

![image-20201102120404832](/images/2020110205.png)