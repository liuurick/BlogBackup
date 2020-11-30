---
title: MySQL高级
date: 2020-11-20 13:55:41
tags: MySQL高级
categories: MySQL
---

MySQL的深入学习主要分为5个部分，分别是：

- MySQL的架构介绍
- 索引优化分析
- 查询截取分析
- MySQL锁机制
- 主从复制

<!--more-->

# 一.mysql的架构介绍

## 1. MysqlLinux版本的安装--mysql5.5

### 1.1 下载地址

https://dev.mysql.com/downloads/mysql/

### 1.2 检查当前系统是否安装过mysql

查询命令：`rpm -qa|grep -i mysql`

删除命令：`rpm -e RPM软件包名（改名字是上一个命令查出来的名字）`

### 1.3 安装mysql服务端(注意提示)与客户端

安装路径：

![image-20201122172644460](C:\Users\admin\Desktop\blog\source\images\2020112201.png)

上传并解压文件：

`tar -xvf MySQL-5.5.62-1.el7.x86_64.rpm-bundle.tar`

![image-20201122210059330](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201122210059330.png)



安装：

```
rpm -ivh MySQL-server-5.5.62-1.el7.x86_64.rpm
rpm -ivh MySQL-client-5.5.62-1.el7.x86_64.rpm
```

注意：这里可能会出现错误：

> conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_641

![image-20201122210437197](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201122210437197.png)

与mariadb冲突，删除mariadb即可！

```
rpm -e mariadb-libs-1:5.5.65-1.el7.x86_64 --nodeps
```



### 1.4 查看Mysql安装时创建的mysql用户和mysql组

![image-20201122204405232](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201122204405232.png)

或者也可以执行mysqladmin --version 命令，类似java -version，如果可以查看消息，说明成功

![image-20201122210850349](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201122210850349.png)



### 1.5 mysql服务的启动和停止

```
service mysql start
service mysql stop
```

查看进程：

![image-20201122211604756](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201122211604756.png)

**注意：**可能会报下面的错误

> Starting MySQL. ERROR! The server quit without updating PID file (/var/lib/mysql/localhost.localdomain.pid).

![image-20201122211244330](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201122211244330.png)

```
解决步骤：
yum -y install autoconf
/usr/bin/mysql_install_db --user=mysql
service mysql start
```

解答参考页面：https://www.cnblogs.com/weibanggang/p/11230528.html



### 1.6 mysql的ROOT密码设置

首次会直接连接成功，因为mysql默认没有密码。

修改密码可以按照Service中的提示：

```
/usr/bin/mysqladmin -u root password 123456
```

![image-20201122212503860](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201122212503860.png)



### 1.7 自启动mysql服务

```
chkconfig mysql on
```

或者

```
ntsysv
```



`chkconfig --list|grep mysql`

![image-20201122213038046](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201122213038046.png)



### 1.8 修改配置文件位置

5.5版本：`cp /usr/share/mysql/my-huge.cnf /etc/my.cnf`

5.6版本：`cp /usr/share/mysql/my-default.cnf /etc/my.cnf`



### 1.9 修改字符集和数据存储路径

查看字符集：

```
show variables like '%char%';
show variables like '%character%';
```

![image-20201122220007001](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201122220007001.png)

修改my.cnf文件：

```cnf
[client]
#password	= your_password
port		= 3306
socket		= /var/lib/mysql/mysql.sock

default-character-set = utf8mb4

# Here follows entries for some specific programs

# The MySQL server
[mysqld]
port		= 3306

character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci

socket		= /var/lib/mysql/mysql.sock
skip-external-locking
key_buffer_size = 384M
max_allowed_packet = 1M
table_open_cache = 512
sort_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size = 32M
# Try number of CPU's*2 for thread_concurrency
thread_concurrency = 8


log-bin=mysql-bin

# required unique id between 1 and 2^32 - 1
# defaults to 1 if master-host is not set
# but will not function as a master if omitted
server-id	= 1



[mysqldump]
quick
max_allowed_packet = 16M

[mysql]
no-auto-rehash

default-character-set = utf8mb4

# Remove the next comment character if you are not familiar with SQL
#safe-updates

[myisamchk]
key_buffer_size = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout
```

修改完成之后重启mysql服务

### 1.10 Mysql的安装位置

在linux下查看安装目录 ps -ef|grep mysql

![image-20201122212059724](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201122212059724.png)



## 2.Mysql配置文件   

### 2.1 二进制日志log-bin

主要用作主重复制

### 2.2 错误日志log-error

默认是关闭的,记录严重的警告和错误信息,每次启动和关闭的详细信息等.

### 2.3 查询日志log

默认关闭,记录查询的sql语句，如果开启会减低mysql的整体性能，因为记录日志也是需要消耗系统资源的

### 2.4 数据文件

在Windows下，D:\ProgramFiles\MySQL\MySQLServer5.5\data目录下可以挑选很多库；

在Linux下，看看当前系统中的全部库后再进去（默认路径：/var/lib/mysql）。	

**数据文件：**	

​	frm文件：存放表结构

​	myd文件：存放表数据

​	myi文件：存放表索引

### 2.5 如何配置

windows：my.ini文件

Linux：/etc/my.cnf文件



## 3.Mysql逻辑架构介绍

和其它数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎的架构上，插件式的存储引擎架构将查询处理和其它的系统任务以及数据的存储提取相分离。这种架构可以根据业务的需求和实际需要选择合适的存储引擎。

![image-20201123095017572](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201123095017572.png)



### 3.1 连接层

最上层是一些客户端和连接服务，包含本地sock通信和大多数基于客户端/服务端工具实现的类似于tcp/ip的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

**Connectors：**指的是不同语言中与SQL的交互

**Connection Pool**: 连接池
管理缓冲用户连接，线程处理等需要缓存的需求。
负责监听对 MySQL Server 的各种请求，接收连接请求，转发所有连接请求到线程管理模块。每一个连接上 MySQL Server 的客户端请求都会被分配（或创建）一个连接线程为其单独服务。而连接线程的主要工作就是负责 MySQL Server 与客户端的通信，接受客户端的命令请求，传递 Server 端的结果信息等。线程管理模块则负责管理维护这些连接线程。包括线程的创建，线程的 cache 等。

### 3.2 服务层

第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化及部分内置函数的执行，所有跨存储引擎的功能也在这一层实现，如过程，函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化，如确定查询表的顺序，是否利用索引等，最后生成相应的执行操作。如果是select语句，服务器还会查询内部的缓存，如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

**Management Serveices & Utilities： **系统管理和控制工具
**Connection Pool: 连接池**。管理缓冲用户连接，线程处理等需要缓存的需求。
负责监听对 MySQL Server 的各种请求，接收连接请求，转发所有连接请求到线程管理模块。每一个连接上 MySQL Server 的客户端请求都会被分配（或创建）一个连接线程为其单独服务。而连接线程的主要工作就是负责 MySQL Server 与客户端的通信，接受客户端的命令请求，传递 Server 端的结果信息等。线程管理模块则负责管理维护这些连接线程。包括线程的创建，线程的 cache 等。
**SQL Interface: SQL接口**
接受用户的SQL命令，并且返回用户需要查询的结果。比如select from就是调用SQL Interface
**Parser: 解析器**
SQL命令传递到解析器的时候会被解析器验证和解析。解析器是由Lex和YACC实现的，是一个很长的脚本。
在 MySQL中我们习惯将所有 Client 端发送给 Server 端的命令都称为 query ，在 MySQL Server 里面，连接线程接收到客户端的一个 Query 后，会直接将该 query 传递给专门负责将各种 Query 进行分类然后转发给各个对应的处理模块。

​		主要功能：
​		a . 将SQL语句进行语义和语法的分析，分解成数据结构，然后按照不同的操作类型进行分类，然后做出针对性的转发到后续步骤，以后SQL语句的传递和处理就是基于这个结构的。
​		b.  如果在分解构成中遇到错误，那么就说明这个sql语句是不合理的

**Optimizer: 查询优化器**

SQL语句在查询之前会使用查询优化器对查询进行优化。就是优化客户端请求的 query（sql语句） ，根据客户端请求的 query 语句，和数据库中的一些统计信息，在一系列算法的基础上进行分析，得出一个最优的策略，告诉后面的程序如何取得这个 query 语句的结果
他使用的是“**选取-投影-联接**”策略进行查询。
       用一个例子就可以理解： select uid,name from user where gender = 1;
       这个select 查询先根据where 语句进行选取，而不是先将表全部查询出来以后再进行gender过滤
       这个select查询先根据uid和name进行属性投影，而不是将属性全部取出以后再进行过滤
       将这两个查询条件联接起来生成最终查询结果

**Cache和Buffer： 查询缓存**
他的主要功能是将客户端提交 给MySQL 的 Select 类 query 请求的返回结果集 cache 到内存中，与该 query 的一个 hash 值 做一个对应。该 Query 所取数据的基表发生任何数据的变化之后， MySQL 会自动使该 query 的Cache 失效。在读写比例非常高的应用系统中， Query Cache 对性能的提高是非常显著的。当然它对内存的消耗也是非常大的。
如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等

### 3.3 引擎层
存储引擎层，存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取。

**存储引擎接口模块**：可以说是 MySQL 数据库中最有特色的一点了。目前各种数据库产品中，基本上只有 MySQL 可以实现其底层数据存储引擎的插件式管理。这个模块实际上只是 一个抽象类，但正是因为它成功地将各种数据处理高度抽象化，才成就了今天 MySQL 可插拔存储引擎的特色。
     从上图可以看出，MySQL区别于其他数据库的最重要的特点就是其插件式的表存储引擎。MySQL插件式的存储引擎架构提供了一系列标准的管理和服务支持，这些标准与存储引擎本身无关，可能是每个数据库系统本身都必需的，如SQL分析器和优化器等，而存储引擎是底层物理结构的实现，每个存储引擎开发者都可以按照自己的意愿来进行开发。
    **注意：**存储引擎是基于表的，而不是数据库。



### 3.4 存储层
数据存储层，主要是将数据存储在运行于裸设备的文件系统之上，并完成与存储引擎的交互。



**查询说明：**

**![img](https://img2018.cnblogs.com/blog/1491092/201906/1491092-20190604210526387-1627086664.png)**

首先，mysql的查询流程大致是：

- mysql客户端通过协议与mysql服务器建连接，发送查询语句，先检查查询缓存，如果命中(一模一样的sql才能命中)，直接返回结果，否则进行语句解析,也就是说，在解析查询之前，服务器会先访问查询缓存(query cache)——它存储SELECT语句以及相应的查询结果集。如果某个查询结果已经位于缓存中，服务器就不会再对查询进行解析、优化、以及执行。它仅仅将缓存中的结果返回给用户即可，这将大大提高系统的性能。

- 语法解析器和预处理：首先mysql通过关键字将SQL语句进行解析，并生成一颗对应的“解析树”。mysql解析器将使用mysql语法规则验证和解析查询；预处理器则根据一些mysql规则进一步检查解析数是否合法。

- 查询优化器当解析树被认为是合法的了，并且由优化器将其转化成执行计划。一条查询可以有很多种执行方式，最后都返回相同的结果。优化器的作用就是找到这其中最好的执行计划。。

- 然后，mysql默认使用的BTREE索引，并且一个大致方向是:无论怎么折腾sql，至少在目前来说，mysql最多只用到表中的一个索引。





## 4.Mysql存储引擎

常用命令：

查看mysql现在提供什么存储引擎

`show engines;`

查看mysql当前默认的存储引擎

`show variables like '%storage_engine%'`



MyISAM和InnoDB对比：

![image-20201123100931321](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201123100931321.png)



# 二.索引优化分析

## 1.性能下降SQL慢 ，执行时间长 ，等待时间长的原因分析

主要有以下原因：

- 查询语句写的烂

- 索引失效

  ​	单值索引/复合索引失效

- 关联查询太多join(设计缺陷或不得已的需求)

- 服务器调优及各个参数设置(缓冲\线程数等)



## 2.常见通用的join查询

### 2.1 SQL执行顺序

手写：

```sql
SELECT DISTINCT <query_list> FROM <left_table>
<join type> JOIN <right_table> ON <join_condition>
WHERE <wherer_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
ORDER BY <order_by_list>
LIMIT <limit number>
```



机读：

```sql
FROM <left_table> 
ON <join_condition>
<join_type> JOIN <right_table>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
SELECT
DISTINCT <query_list>
ORDER BY <order_by_condition>
LIMIT <limit_number>
```



总结：

![img](https://images2015.cnblogs.com/blog/1054685/201706/1054685-20170626154318664-711238691.jpg)

Join图

![img](https://images2015.cnblogs.com/blog/1054685/201706/1054685-20170626113414446-2016950911.jpg)



### 2.2 MySQL数据库实例

#### 1.创建数据库：

```
mysql> create database db_test;
Query OK, 1 row affected (0.01 sec)
```

####  2.使用数据库：

```
mysql> use db_test;
Database changed
```

#### 3.创建表、添加数据：

```sql
CREATE TABLE `tb_dept` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '部门主键',
  `deptName` varchar(30) DEFAULT NULL COMMENT '部门名称',
  `locAdd` varchar(40) DEFAULT NULL COMMENT '楼层',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;


CREATE TABLE `tb_emp` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '员工主键',
  `name` varchar(20) DEFAULT NULL COMMENT '员工姓名',
  `deptId` int(11) DEFAULT NULL COMMENT '部门外键',
  PRIMARY KEY (`id`),
  KEY `fk_dept_id` (`deptId`)
  #CONSTRAINT `fk_dept_id` FOREIGN KEY (`deptId`) REFERENCES `tb_dept` (`id`) COMMENT '部门外键设置, 已经注释掉。'
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

INSERT INTO `tb_dept` VALUES ('1', 'RD', '11');
INSERT INTO `tb_dept` VALUES ('2', 'HR', '12');
INSERT INTO `tb_dept` VALUES ('3', 'MK', '13');
INSERT INTO `tb_dept` VALUES ('4', 'MIS', '14');
INSERT INTO `tb_dept` VALUES ('5', 'FD', '15');

INSERT INTO `tb_emp` VALUES ('1', '张三', '1');
INSERT INTO `tb_emp` VALUES ('2', '李四', '1');
INSERT INTO `tb_emp` VALUES ('3', '王二', '1');
INSERT INTO `tb_emp` VALUES ('4', '麻子', '2');
INSERT INTO `tb_emp` VALUES ('5', '小马', '2');
INSERT INTO `tb_emp` VALUES ('6', '马旭', '3');
INSERT INTO `tb_emp` VALUES ('7', '小丁', '4');
INSERT INTO `tb_emp` VALUES ('8', '小西', '51');
```

**tb_emp表数据：**

```sql
mysql> select * from tb_emp;
+----+--------+--------+
| id | name   | deptId |
+----+--------+--------+
|  1 | 张三   |      1 |
|  2 | 李四   |      1 |
|  3 | 王二   |      1 |
|  4 | 麻子   |      2 |
|  5 | 小马   |      2 |
|  6 | 马旭   |      3 |
|  7 | 小丁   |      4 |
|  8 | 小西   |     51 |
+----+--------+--------+
8 rows in set (0.01 sec)
```



**tb_dept表数据：**

```sql
mysql> select * from tb_dept;
+----+----------+--------+
| id | deptName | locAdd |
+----+----------+--------+
|  1 | RD       | 11     |
|  2 | HR       | 12     |
|  3 | MK       | 13     |
|  4 | MIS      | 14     |
|  5 | FD       | 15     |
+----+----------+--------+
5 rows in set (0.01 sec)
```

 

**笛卡儿积：**

```sql
mysql> select * from tb_emp, tb_dept;
+----+--------+--------+----+----------+--------+
| id | name   | deptId | id | deptName | locAdd |
+----+--------+--------+----+----------+--------+
|  1 | 张三   |      1 |  1 | RD       | 11     |
|  1 | 张三   |      1 |  2 | HR       | 12     |
|  1 | 张三   |      1 |  3 | MK       | 13     |
|  1 | 张三   |      1 |  4 | MIS      | 14     |
|  1 | 张三   |      1 |  5 | FD       | 15     |
|  2 | 李四   |      1 |  1 | RD       | 11     |
|  2 | 李四   |      1 |  2 | HR       | 12     |
|  2 | 李四   |      1 |  3 | MK       | 13     |
|  2 | 李四   |      1 |  4 | MIS      | 14     |
|  2 | 李四   |      1 |  5 | FD       | 15     |
|  3 | 王二   |      1 |  1 | RD       | 11     |
|  3 | 王二   |      1 |  2 | HR       | 12     |
|  3 | 王二   |      1 |  3 | MK       | 13     |
|  3 | 王二   |      1 |  4 | MIS      | 14     |
|  3 | 王二   |      1 |  5 | FD       | 15     |
|  4 | 麻子   |      2 |  1 | RD       | 11     |
|  4 | 麻子   |      2 |  2 | HR       | 12     |
|  4 | 麻子   |      2 |  3 | MK       | 13     |
|  4 | 麻子   |      2 |  4 | MIS      | 14     |
|  4 | 麻子   |      2 |  5 | FD       | 15     |
|  5 | 小马   |      2 |  1 | RD       | 11     |
|  5 | 小马   |      2 |  2 | HR       | 12     |
|  5 | 小马   |      2 |  3 | MK       | 13     |
|  5 | 小马   |      2 |  4 | MIS      | 14     |
|  5 | 小马   |      2 |  5 | FD       | 15     |
|  6 | 马旭   |      3 |  1 | RD       | 11     |
|  6 | 马旭   |      3 |  2 | HR       | 12     |
|  6 | 马旭   |      3 |  3 | MK       | 13     |
|  6 | 马旭   |      3 |  4 | MIS      | 14     |
|  6 | 马旭   |      3 |  5 | FD       | 15     |
|  7 | 小丁   |      4 |  1 | RD       | 11     |
|  7 | 小丁   |      4 |  2 | HR       | 12     |
|  7 | 小丁   |      4 |  3 | MK       | 13     |
|  7 | 小丁   |      4 |  4 | MIS      | 14     |
|  7 | 小丁   |      4 |  5 | FD       | 15     |
|  8 | 小西   |     51 |  1 | RD       | 11     |
|  8 | 小西   |     51 |  2 | HR       | 12     |
|  8 | 小西   |     51 |  3 | MK       | 13     |
|  8 | 小西   |     51 |  4 | MIS      | 14     |
|  8 | 小西   |     51 |  5 | FD       | 15     |
+----+--------+--------+----+----------+--------+
40 rows in set (0.00 sec)
```

 

**查询tb_emp表和tb_dept中公共的数据：**

```sql
mysql> select * from tb_emp e
    -> inner join tb_dept d on e.deptId = d.id;
+----+--------+--------+----+----------+--------+
| id | name   | deptId | id | deptName | locAdd |
+----+--------+--------+----+----------+--------+
|  1 | 张三   |      1 |  1 | RD       | 11     |
|  2 | 李四   |      1 |  1 | RD       | 11     |
|  3 | 王二   |      1 |  1 | RD       | 11     |
|  4 | 麻子   |      2 |  2 | HR       | 12     |
|  5 | 小马   |      2 |  2 | HR       | 12     |
|  6 | 马旭   |      3 |  3 | MK       | 13     |
|  7 | 小丁   |      4 |  4 | MIS      | 14     |
+----+--------+--------+----+----------+--------+
7 rows in set (0.00 sec)
```

 

**查询tb_emp表中全部的数据：**

```sql
mysql> select * from tb_emp e
    -> left join tb_dept d on e.deptId = d.id;
    
#该条数据为 tb_emp 表中独有的数据, tb_dept 表中的字段用 null 占位
+----+--------+--------+------+----------+--------+
| id | name   | deptId | id   | deptName | locAdd |
+----+--------+--------+------+----------+--------+
|  1 | 张三   |      1 |    1 | RD       | 11     |
|  2 | 李四   |      1 |    1 | RD       | 11     |
|  3 | 王二   |      1 |    1 | RD       | 11     |
|  4 | 麻子   |      2 |    2 | HR       | 12     |
|  5 | 小马   |      2 |    2 | HR       | 12     |
|  6 | 马旭   |      3 |    3 | MK       | 13     |
|  7 | 小丁   |      4 |    4 | MIS      | 14     |
|  8 | 小西   |     51 | NULL | NULL     | NULL   |　　　　
+----+--------+--------+------+----------+--------+
8 rows in set (0.00 sec)
```

 

**查询tb_dept表中全部的数据：**

```sql
mysql> select * from tb_emp e
    -> right join tb_dept d on e.deptId = d.id;
    
#该条数据为 tb_dept 表中独有的数据, tb_emp 表中的字段用 null 占位
+------+--------+--------+----+----------+--------+
| id   | name   | deptId | id | deptName | locAdd |
+------+--------+--------+----+----------+--------+
|    1 | 张三   |      1 |  1 | RD       | 11     |
|    2 | 李四   |      1 |  1 | RD       | 11     |
|    3 | 王二   |      1 |  1 | RD       | 11     |
|    4 | 麻子   |      2 |  2 | HR       | 12     |
|    5 | 小马   |      2 |  2 | HR       | 12     |
|    6 | 马旭   |      3 |  3 | MK       | 13     |
|    7 | 小丁   |      4 |  4 | MIS      | 14     |
| NULL | NULL   |   NULL |  5 | FD       | 15     |     
+------+--------+--------+----+----------+--------+
8 rows in set (0.01 sec)
```

 

**查询tb_emp表中独有的数据：**

```sql
mysql> select * from tb_emp e
    -> left join tb_dept d on e.deptId = d.id
    -> where d.id is null;
+----+--------+--------+------+----------+--------+
| id | name   | deptId | id   | deptName | locAdd |
+----+--------+--------+------+----------+--------+
|  8 | 小西   |     51 | NULL | NULL     | NULL   |
+----+--------+--------+------+----------+--------+
1 row in set (0.00 sec)
```

 

**查询tb_dept表中独有的数据：**

```sql
mysql> select * from tb_emp e
    -> right join tb_dept d on e.deptId = d.id
    -> where e.deptId is null;
+------+------+--------+----+----------+--------+
| id   | name | deptId | id | deptName | locAdd |
+------+------+--------+----+----------+--------+
| NULL | NULL |   NULL |  5 | FD       | 15     |
+------+------+--------+----+----------+--------+
1 row in set (0.00 sec)
```

 

**查询tb_emp和tb_dept表中所有的数据：**

```sql
mysql> select * from tb_emp e
    -> full outer join tb_dept d on e.deptId = d.id;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'full outer join tb_dept d on e.deptId = d.id' at line 2
```

出错原因：mysql 中不支持这种连接方式。

**解决办法：**左连接数据和右连接数据相加，将公共的部分去重。

```
mysql> select * from tb_emp e 
    -> left join tb_dept d on e.deptId = d.id
    -> union　　　　　#去重
    -> select * from tb_emp e
    -> right join tb_dept d on e.deptId = d.id;
    
#该条数据为 tb_emp 表中独有的数据, tb_dept 表中的字段用 null 占位
#该条数据为 tb_dept 表中独有的数据, tb_emp 表中的字段用 null 占位
+------+--------+--------+------+----------+--------+
| id   | name   | deptId | id   | deptName | locAdd |
+------+--------+--------+------+----------+--------+
|    1 | 张三   |      1 |    1 | RD       | 11     |
|    2 | 李四   |      1 |    1 | RD       | 11     |
|    3 | 王二   |      1 |    1 | RD       | 11     |
|    4 | 麻子   |      2 |    2 | HR       | 12     |
|    5 | 小马   |      2 |    2 | HR       | 12     |
|    6 | 马旭   |      3 |    3 | MK       | 13     |
|    7 | 小丁   |      4 |    4 | MIS      | 14     |
|    8 | 小西   |     51 | NULL | NULL     | NULL   |　　　　
| NULL | NULL   |   NULL |    5 | FD       | 15     |　　　 
+------+--------+--------+------+----------+--------+
9 rows in set (0.00 sec)
```

 

**查询tb_emp和tb_dept表中独有的数据：**

```
mysql> select * from tb_emp e
    -> left join tb_dept d on e.deptId = d.id
    -> where d.id is null
    -> union
    -> select * from tb_emp e
    -> right join tb_dept d on e.deptId = d.id
    -> where e.deptId is null;
+------+--------+--------+------+----------+--------+
| id   | name   | deptId | id   | deptName | locAdd |
+------+--------+--------+------+----------+--------+
|    8 | 小西   |     51 | NULL | NULL     | NULL   |
| NULL | NULL   |   NULL |    5 | FD       | 15     |
+------+--------+--------+------+----------+--------+
2 rows in set (0.00 sec)
```

 



7种Join





## 3.索引简介

**索引概述：**

- 索引(Index) 是帮助MySQL高效获取数据的数据结构。索引的本质就是数据结构。 索引的目的在于提高查询效率，可以类比字典，可以简单的理解为“排好序的快速查找数据结构”。
- 在数据本身之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用(指向)数据，这样就可以在这些数据结构上实现高级查找算法。这种数据结构就是索引。
- 一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上。
- 我们平时所说的索引，如果没有特别指明，都是指B树结构组织的索引。

 

**二叉树算法图：**

**[![img](https://images2015.cnblogs.com/blog/1054685/201706/1054685-20170627085051383-985324526.png)](https://images2015.cnblogs.com/blog/1054685/201706/1054685-20170627085051383-985324526.png)**

​    为了加快Col2的查找，可以维护一个右边所示的二叉查找树，每个节点分别包含索引键值和一个指向对应数据记录物理地址的指针，这样就可以运用二叉查找在一定复杂度内获取到相应数据，从而快速的检索出符合条件的记录。

 

**索引的优势：**

1. 创建索引可以提高数据检索的效率，降低数据库的IO成本。
2. 通过索引列对数据进行排序，降低数据排序的成本，降低了CPU的消耗。

 

**索引的劣势：**

1. 索引其实也是一张表，该表保存了主键与索引字段，并指向实体的记录，所以索引列也是要占用空间的。
2. 索引虽然提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要调整因为更新所带来的键值变化后的索引信息。
3. 索引只是提高效率的一个因素，如果MySQL有大数据量的表，就需要花时间研究建立最优秀的索引，或优化查询语句。

 

**MySQL索引分类：**

1. 单值索引：一个索引只包含一个单列，一个表可以有多个单列索引。
2. 唯一索引：索引列的值必须唯一，但允许有空值。
3. 复合索引：一个索引包含多个列。

 

**索引的基本语法：**

- **创建：**

1. CREATE [UNIQUE] INDEX indexName ON tableName(colName(length));。——如果是CHAR、VARCHAR类型，length可以小于字段实际长度，若是BLOB和TEXT类型，必须指定length。
2. ALTER tableName ADD [UNIQUE] INDEX indexName ON (colName(length));。

- **删除：**DROP INDEX [indexName] ON tableName;　　
- **查看：**SHOW INDEX FROM tableName;

 

**使用ALTER命令四种方式添加数据表的索引：**

1. ALTER TABLE tableName ADD PRIMARY KEY(column_list); 该语句添加一个主键，这意味着索引值必须唯一，且不能为null。
2. ALTER TABLE tableName ADD UNIQUE indexName(column_list);该语句创建索引的值必须唯一(null除外，null可能会出现多次)。
3. ALTER TABLE tableName ADD INDEX indexName(column_list);该语句添加普通索引，索引值可出现多次。
4. AKTER TABLE tableName ADD FULLTEXT indexName(column_list);该语句指定索引为FULLTEXT，用于全文索引。

 

**MySQL索引结构：**

1. BTree索引
2. Hash索引
3. full-text全文索引
4. R-Tree索引

java工程师掌握BTree就可以。

 

**哪些情况需要创建索引：**

1. 主键自动建立唯一索引。
2. 频繁作为查询条件的字段应该创建索引。
3. 查询中与其他表关联的字段，外键关系建立索引。
4. 在高并发下倾向创建组合索引。
5. 查询中的排序字段适合创建索引，排序字段若通过索引去访问将大大提高排序速率。
6. 查询中统计或者分组字段适合创建索引。

 

**哪些情况不要创建索引：**

1. where条件里用不到的字段不创建索引。
2. 表记录太少。
3. 频繁更新的字段不不适合创建索引。——因为每次更新不仅更新了记录还会更新索引，加重了IO负担。
4. 经常增删改的表。
5. 数据重复且平均分录的表字段，建立索引没有太大的实际效果。



## 4.性能分析





## 5.索引优化











































# 三.查询截取分析





# 四.MySQL锁机制







# 五.主从复制

