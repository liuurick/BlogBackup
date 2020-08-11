---
title: Redis集群搭建
date: 2020-07-30 09:37:05
categories: Redis集群
tags: Redis集群
---

### 1、Redis集群方案比较

- **哨兵模式**

![img](/images/2020073001.jpg)

在redis3.0以前的版本要实现集群一般是借助哨兵sentinel工具来监控master节点的状态，如果master节点异常，则会做主从切换，将某一台slave作为master，哨兵的配置略微复杂，并且性能和高可用性等各方面表现一般，特别是在主从切换的瞬间存在访问瞬断的情况
<!--more-->
- **高可用集群模式**

![img](/images/2020073002.png)

redis集群是一个由多个主从节点群组成的分布式服务器群，它具有复制、高可用和分片特性。Redis集群不需要sentinel哨兵也能完成节点移除和故障转移的功能。需要将每个节点设置成集群模式，这种集群模式没有中心节点，可水平扩展，据官方文档称可以线性扩展到1000节点。redis集群的性能和高可用性均优于之前版本的哨兵模式，且集群配置非常简单。

### 2、redis高可用集群搭建

- **redis安装**

```
下载地址：http://redis.io/download
安装步骤：
# 安装gcc
yum install gcc

# 把下载好的redis-3.0.0-rc2.tar.gz放在/usr/local文件夹下，并解压
tar -zxvf redis-3.0.0-rc2.tar.gz

# 进入到解压好的redis-3.0.0目录下，进行编译
make

# 进入到redis-3.0.0/src目录下进行安装，安装完成验证src目录下是否已经生成了redis-server 、redis-cil
make install

# 建立俩个文件夹存放redis命令和配置文件
mkdir -p /usr/local/redis/etc
mkdir -p /usr/local/redis/bin

# 把redis-3.0.0下的redis.conf复制到/usr/local/redis/etc下
cp redis.conf /usr/local/redis/etc/

# 移动redis-3.0.0/src里的几个文件到/usr/local/redis/bin下
mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-dump redis-cli redis-server /usr/local/redis/bin

# 启动并指定配置文件
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf（注意要使用后台启动，所以修改redis.conf里的daemonize改为yes)

# 验证启动是否成功
ps -ef | grep redis 

# 查看是否有redis服务或者查看端口
netstat -tunpl | grep 6379

# 进入redis客户端 
/usr/local/redis/bin/redis-cli 

# 退出客户端
quit

# 退出redis服务： 
（1）pkill redis-server 
（2）kill 进程号                       
（3）/usr/local/redis/bin/redis-cli shutdown
```

- **redis集群搭建**

redis集群需要至少要三个master节点，我们这里搭建三个master节点，并且给每个master再搭建一个slave节点，总共6个redis节点，由于节点数较多，这里采用在一台机器上创建6个redis实例，并将这6个redis实例配置成集群模式，所以这里搭建的是伪分布式集群模式，当然真正的分布式集群的配置方法几乎一样，搭建伪分布式集群的步骤如下：

```
第一步：在/usr/local下创建文件夹redis-cluster，然后在其下面分别创建6个文件夾如下
（1）mkdir -p /usr/local/redis-cluster
（2）mkdir 8001、 mkdir 8002、 mkdir 8003、 mkdir 8004、 mkdir 8005、 mkdir 8006

第一步：把之前的redis.conf配置文件copy到8001下，修改如下内容：
（1）daemonize yes
（2）port 8001（分别对每个机器的端口号进行设置）
（3）bind 192.168.0.61（必须要绑定当前机器的ip，这里方便redis集群定位机器，不绑定可能会出现循环查找集群节点机器的情况）
（4）dir /usr/local/redis-cluster/8001/（指定数据文件存放位置，必须要指定不同的目录位置，不然会丢失数据）
（5）cluster-enabled yes（启动集群模式）
（6）cluster-config-file nodes-8001.conf（这里800x最好和port对应上）
（7）cluster-node-timeout 5000
（8）appendonly yes

第三步：把修改后的配置文件，分别 copy到各个文夹下，注意每个文件要修改第2、4、6项里的端口号
快捷复制命令：%s/原目标/目标地址/g    
第四步：由于 redis集群需要使用 ruby命令，所以我们需要安装 ruby
（1）yum install ruby
（2）yum install rubygems
（3）gem install redis --version 3.0.0（安装redis和 ruby的接囗）

第五步：分别启动6个redis实例，然后检查是否启动成功
（1）/usr/local/redis/bin/redis-server /usr/local/redis-cluster/800*/redis.conf
（2）ps -ef | grep redis 查看是否启动成功

第六步：在redis的安装目录下执行 redis-trib.rb命令
（1）cd /usr/local/redis-3.0.0/src
（2）./redis-trib.rb create --replicas 1 192.168.0.61:8001 192.168.0.61:8002 192.168.0.61:8003 192.168.0.61:8004 192.168.0.61:8005 192.168.0.61:8006
新版本：redis-cli --cluster create 192.168.200.10:8001 192.168.200.10:8002  192.168.200.10:8003 192.168.200.10:8004 192.168.200.10:8005 192.168.200.10:8006  --cluster-replicas 1

第七步：验证集群：
（1）连接任意一个客户端即可：./redis-cli -c -h -p (-c表示集群模式，指定ip地址和端口号）如：/usr/local/redis/bin/redis-cli -c -h 192.168.0.61 -p 800*
（2）进行验证： cluster info（查看集群信息）、cluster nodes（查看节点列表）
（3）进行数据操作验证
（4）关闭集群则需要逐个进行关闭，使用命令：
/usr/local/redis/bin/redis-cli -c -h 192.168.0.61 -p 800* shutdown

PS：当出现集群无法启动时，删除redis的临时数据文件，再次重新启动每一个redis服务，然后重新构造集群环境。
```



### 3、Java操作redis集群

借助redis的java客户端jedis可以操作以上集群，引用jedis版本的maven坐标如下：

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

Java编写访问redis集群的代码非常简单，如下所示：

```
import java.io.IOException;
import java.util.HashSet;
import java.util.Set;

import redis.clients.jedis.HostAndPort;
import redis.clients.jedis.JedisCluster;
import redis.clients.jedis.JedisPoolConfig;

/**
 * 访问redis集群
 * @author aaron.rao
 *
 */
public class RedisCluster 
{
    public static void main(String[] args) throws IOException
    {
        Set<HostAndPort> jedisClusterNode = new HashSet<HostAndPort>();
        jedisClusterNode.add(new HostAndPort("192.168.0.61", 8001));
        jedisClusterNode.add(new HostAndPort("192.168.0.61", 8002));
        jedisClusterNode.add(new HostAndPort("192.168.0.61", 8003));
        jedisClusterNode.add(new HostAndPort("192.168.0.61", 8004));
        jedisClusterNode.add(new HostAndPort("192.168.0.61", 8005));
        jedisClusterNode.add(new HostAndPort("192.168.0.61", 8006));
        
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(100);
        config.setMaxIdle(10);
        config.setTestOnBorrow(true);
        JedisCluster jedisCluster = new JedisCluster(jedisClusterNode, 6000, 10, config);
        System.out.println(jedisCluster.set("student", "aaron"));
        System.out.println(jedisCluster.set("age", "18"));
        
        System.out.println(jedisCluster.get("student"));
        System.out.println(jedisCluster.get("age"));
        
        jedisCluster.close();
    }
}
运行效果如下：
OK
OK
aaron
18
```