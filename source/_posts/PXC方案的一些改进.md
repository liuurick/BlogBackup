---
title: PXC方案的一些改进
date: 2020-11-09 21:35:39
tags: [Docker,MySQL集群]
categories: PXC
---

之前MySQL集群搭建使用的是PXC方案，最后想想还是有很多地方需要改进：

主要分为两点：1.负载均衡的高可用方案  2.引入热备份数据

<!--more-->

### 1.负载均衡的高可用方案

虽然之前的MySQL集群也有负载均衡，但是haproxy是单节点的，不具备高可用。。。

#### 1.1 安装Keepalived
进入容器
`docker exec -it h1 bash`
在容器中安装
`apt-get update`
`apt-get install keepalived`

#### 1.2 Keepalived配置文件
在容器中
`vim /etc/keepalived/keepalived.conf`
如果容器没有安装vim，请输入命令：`apt-get install vim`
然后在文件里贴上配置

```xml
vrrp_instance  VI_1 {
    state  MASTER
    interface  eth0
    virtual_router_id  51
    priority  100
    advert_int  1
    authentication {
        auth_type  PASS
        auth_pass  123456
    }
    virtual_ipaddress {
        172.10.0.201
    }
}
```

- state 是keepalived的身份（MASTER主服务，BACKUP备服务器）。主服务要抢占虚拟IP，备用服务器不会抢占IP
- interface 网卡设备
- virtual_router_id 虚拟路由标识，MASTER和BACKUP的虚拟路由标识必须一致。标识可以是 0 ～ 255
- priority 权重
- advert_int 心跳检测秒，MASTER与BACKUP节点间同步检查的时间间。主备之间必须一致。
- authentication 主从服务器验证方式。主备必须使用相同的密码才能正常通信
- virtual_ipaddress 虚拟IP地址，可以设置多个虚拟IP地址，每行一个



#### 1.3 启动keepalived

在容器里输入命令：`service keepalived start`
因为不能直接停止容器，所以为了正常退出不关闭容器，使用Ctrl+P+Q进行退出容器
检验：在宿主机中，我们使用：ping 172.10.0.201，检测是否真正启动

![image-20201109224438458](/images/2020110901.png)



![image-20201109225321905](/images/2020110902.png)



### 2.MySQL集群 热备份数据

#### 2.1 冷备份：

冷备份是一种常见的备份方式，通常做法是先关闭数据库，然后在拷贝数据文件。

这种方式简单安全，mysqldump就是典型的冷备份技术。

冷备份的弊端也很大，比如大型网站无法做到关闭业务备份数据，所以冷备份不是最佳选择
当然在之前提到的PXC集群中，我们可以中断某个节点，单独备份数据，再上线。



#### 2.2 热备份：

热备份是在系统运行的状态下备份数据，也是难度最大的备份，Mysql常见的热备份有LVM和XtraBackup两种方案。
这里我使用的是XtraBackup方案实现热备份，XtraBackup是一款基于InnoDB的在线热备工具，具有开源免费，支持在线热备，占用磁盘空间小，能够非常快速地备份与恢复Mysql数据库等特点

**优势：**

- 备份过程中不锁表、快速可靠

- 备份过程中不会打断正在执行的事务

- 能够基于压缩等功能节约磁盘空间和流量

  

**每天进行一次增量备份，每周进行一次全量备份**

**全量备份：**备份全部数据。备份过程时间长，占用空间大

**增量备份：**只备份变化的数据。备份时间短，占用空间小



#### 2.3 全量数据热备份

1.创建数据卷

```undefined
docker volume create backup
```

2.选择数据节点映射数据卷

​	先停止节点

```undefined
docker stop node1
```

​	删除node1重新创建

```undefined
docker rm node1
```

​	创建node1

```kotlin
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456 -v v1:/var/lib/mysql -v backup:/data --privileged -e CLUSTER_JOIN=node2 --name=node1 --net=net1 --ip 172.10.0.2 pxc 
```



**PXC容器中安装XtraBackup，并执行备份**

进入节点

```bash
docker exec -it node1 bash
```

执行

```csharp
apt-get update
apt-get install percona-xtrabackup-24
```

全量备份

```kotlin
innobackupex --user=root --password=123456 /data/backup/full
```

容器内备份数据

```
/data/backup/full
退出容器
exit
查看数据卷目录
docker inspect backup
进入备份目录
```

#### 2.4 全量数据冷还原

数据库可以热备份，但是不能热还原。为了避免恢复过程中的数据同步。我们采用空白的MySQL还原数据，然后再建立PXC集群；还原数据前要将未提交的事务回滚，还原数据之后重启MySQL。

```csharp
rm -rf /var/lib/mysql/*
```

事务回滚

```kotlin
innobackupex --user=root --password=123456 --apply-back /data/backup/full/
```

全量数据冷还原

```go
innobackupex --user=root --password=123456 --copy-back /data/backup/full/
```
