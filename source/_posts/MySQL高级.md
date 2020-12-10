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

最上层是一些客户端和连接服务，包含本地socket通信和大多数基于客户端/服务端工具实现的类似于tcp/ip的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

**Connectors：**指的是不同语言中与SQL的交互

**Connection Pool**: 连接池
管理缓冲用户连接，线程处理等需要缓存的需求。负责监听对 MySQL Server 的各种请求，接收连接请求，转发所有连接请求到线程管理模块。每一个连接上 MySQL Server 的客户端请求都会被分配（或创建）一个连接线程为其单独服务。而连接线程的主要工作就是负责 MySQL Server 与客户端的通信，接受客户端的命令请求，传递 Server 端的结果信息等。线程管理模块则负责管理维护这些连接线程。包括线程的创建，线程的 cache 等。

### 3.2 服务层

第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化及部分内置函数的执行，所有跨存储引擎的功能也在这一层实现，如过程，函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化，如确定查询表的顺序，是否利用索引等，最后生成相应的执行操作。如果是select语句，服务器还会查询内部的缓存，如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

**Management Serveices & Utilities： **系统管理和控制工具

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

**手写：**

```sql
SELECT DISTINCT <query_list> FROM <left_table>
<join type> JOIN <right_table> ON <join_condition>
WHERE <wherer_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
ORDER BY <order_by_list>
LIMIT <limit number>
```

**机读：**

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

**总结：**

![img](https://images2015.cnblogs.com/blog/1054685/201706/1054685-20170626154318664-711238691.jpg)

Join图

![img](https://images2015.cnblogs.com/blog/1054685/201706/1054685-20170626113414446-2016950911.jpg)



### 2.2 MySQL数据库实例

#### 1.创建数据库

```sql
mysql> create database db_test;
Query OK, 1 row affected (0.01 sec)
```

####  2.使用数据库

```sql
mysql> use db_test;
Database changed
```

####  3.查看数据库

```sql
mysql>show tables; //返回当前数据库下的所有表的名称
或者也可以直接用以下命令
mysql>show tables from databaseName;
```

#### 4.创建表、添加数据

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
INSERT INTO `tb_emp` VALUES ('8', '小西', '5');
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

```sql
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

```sql
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

 

## 3.索引简介

### 3.1 索引概述

>索引（在 MySQL 中也叫“键key”）是存储引擎快速找到记录的一种数据结构
>
>​																																							——《高性能MySQL》

- 索引(Index) 是帮助MySQL高效获取数据的数据结构。**索引的本质就是数据结构**。 索引的目的在于提高查询效率，可以类比字典，可以简单的理解为“排好序的快速查找数据结构”。
- 在数据本身之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用(指向)数据，这样就可以在这些数据结构上实现高级查找算法。这种数据结构就是索引。
- 一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上。
- 我们平时所说的索引，如果没有特别指明，都是指B树结构组织的索引。

 

**二叉树算法图：**

**[![img](https://images2015.cnblogs.com/blog/1054685/201706/1054685-20170627085051383-985324526.png)](https://images2015.cnblogs.com/blog/1054685/201706/1054685-20170627085051383-985324526.png)**

​    为了加快Col2的查找，可以维护一个右边所示的二叉查找树，每个节点分别包含索引键值和一个指向对应数据记录物理地址的指针，这样就可以运用二叉查找在一定复杂度内获取到相应数据，从而快速的检索出符合条件的记录。

 

### 3.2 索引的优势

1. 创建索引可以提高数据检索的效率，降低数据库的IO成本。
2. 通过索引列对数据进行排序，降低数据排序的成本，降低了CPU的消耗。

 

### 3.3 索引的劣势

1. 索引其实也是一张表，该表保存了主键与索引字段，并指向实体的记录，所以索引列也是要占用空间的。
2. 索引虽然提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要调整因为更新所带来的键值变化后的索引信息。
3. 索引只是提高效率的一个因素，如果MySQL有大数据量的表，就需要花时间研究建立最优秀的索引，或优化查询语句。

 

### 3.4 MySQL索引分类

1. 单值索引：一个索引只包含一个单列，一个表可以有多个单列索引。
2. 唯一索引：索引列的值必须唯一，但允许有空值。
3. 复合索引：一个索引包含多个列。

 

#### 1.主键索引 PRIMARY KEY

它是一种特殊的唯一索引，不允许有空值。一般是在建表的时候同时创建主键索引。

**注意：**一个表只能有一个主键

![mark](http://songwenjie.vip/blog/180802/1c7D2F0f76.png?imageslim)



#### 2.唯一索引 UNIQUE

唯一索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。

可以通过`ALTER TABLE table_name ADD UNIQUE (column);`创建唯一索引

![mark](http://songwenjie.vip/blog/180802/DBdFeKE8Fk.png?imageslim)

![mark](http://songwenjie.vip/blog/180802/L2jl91b6J6.png?imageslim)

可以通过`ALTER TABLE table_name ADD UNIQUE (column1,column2);`创建唯一组合索引

![mark](http://songwenjie.vip/blog/180802/mihd7Hm5i6.png?imageslim)

![mark](http://songwenjie.vip/blog/180802/bJbdFA9AcL.png?imageslim)

#### 3.普通索引 INDEX

最基本的索引，它没有任何限制。

可以通过`ALTER TABLE table_name ADD INDEX index_name (column);`创建普通索引

![mark](http://songwenjie.vip/blog/180802/17CmJIIJhD.png?imageslim)

![mark](http://songwenjie.vip/blog/180802/4fA7L6kBBm.png?imageslim)

#### 4.组合索引 INDEX

组合索引，即一个索引包含多个列。多用于避免回表查询。

可以通过`ALTER TABLE table_name ADD INDEX index_name(column1, column2, column3);`创建组合索引

![mark](http://songwenjie.vip/blog/180802/CLGIKiAC6J.png?imageslim)

![mark](http://songwenjie.vip/blog/180802/295B9bGi67.png?imageslim)

#### 5.全文索引 FULLTEXT

全文索引（也称全文检索）是目前搜索引擎使用的一种关键技术。

可以通过`ALTER TABLE table_name ADD FULLTEXT (column);`创建全文索引

![mark](http://songwenjie.vip/blog/180802/AjfLLkhdH1.png?imageslim)

![mark](http://songwenjie.vip/blog/180802/bA1a1m49cL.png?imageslim)

索引一经创建不能修改，如果要修改索引，只能删除重建。可以使用`DROP INDEX index_name ON table_name;`删除索引。



### 3.5 索引的基本语法

#### 创建：

```
CREATE [UNIQUE] INDEX indexName ON tableName(colName(length));。——如果是CHAR、VARCHAR类型，length可以小于字段实际长度，若是BLOB和TEXT类型，必须指定length。

ALTER tableName ADD [UNIQUE] INDEX indexName ON (colName(length));。
```

#### 删除：

```
DROP INDEX [indexName] ON tableName;　　
```

#### 查看：

```
SHOW INDEX FROM tableName;
```



#### 使用ALTER命令四种方式添加数据表的索引：

```
ALTER TABLE tableName ADD PRIMARY KEY(column_list); 该语句添加一个主键，这意味着索引值必须唯一，且不能为null。
ALTER TABLE tableName ADD UNIQUE indexName(column_list);该语句创建索引的值必须唯一(null除外，null可能会出现多次)。
ALTER TABLE tableName ADD INDEX indexName(column_list);该语句添加普通索引，索引值可出现多次。
AKTER TABLE tableName ADD FULLTEXT indexName(column_list);该语句指定索引为FULLTEXT，用于全文索引。
```

 

### 3.6 MySQL索引结构：

1. **BTree索引**

   [BTree详解](https://blog.csdn.net/tongdanping/article/details/79878302#3、BTree索引和B%2BTree索引)

2. Hash索引

3. full-text全文索引

4. R-Tree索引 



### 3.7 哪些情况需要创建索引：

1. 主键自动建立唯一索引。
2. 频繁作为查询的条件的字段应该创建索引查询中与其他表关联的字段，外键关系建立索引。
3. 查询中与其他表关联的字段，外键关系建立索引
4. 查询中的排序字段适合创建索引，排序字段若通过索引去访问将大大提高排序速率。
5. 查询中统计或者分组字段适合创建索引。
6. 频繁更新的字段不适合创建索引，因为每次更新不单单是更新了记录还会更新索引，加重IO负担
7. Where条件里用不到的字段不创建索引
8. 单值/组合索引的选择问题，who？（在高并发下倾向创建组合索引）

 

### 3.8 哪些情况不要创建索引：

1. 表记录太少。

2. 经常增删改的表。

3. 数据重复且分布平均的表字段，因此应该只为经常查询和经常排序的数据列建立索引。
   注意，如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。

   ![image-20201129201618523](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201129201618523.png)



## 4.性能分析

### 4.1 MySQL查询优化器(MySQL Query Optimizer)

1.Mysql中有专门负责优化SELECT语句的优化器模块，主要工作：通过技术分析系统中收集到的统计信息，为客户端请求的Query提供他认为最优的执行计划（他认为最优的数据检索方式，但不见得是DBA认为是最优的，这部分最耗费时间）

2.当客户端向MySQL请求一条Query，命令解析器模块完成请求分类，区别出是SELECT并转发给MySQL Query Optimizer时，MySQL Query Optimizer首先会对整条Query进行优化，处理掉一些常量表达式的预算，直接换算成常量值。并对Query中的查询条件进行简化和转换，如去掉一些无用或显而易见的条件，结构调整等。然后分析Query中的Hint信息（如果有），看显示Hint信息是否可以完全确定该Query的执行计划。如果没有Hint或Hint信息还不足以完全确定执行计划，则会读取所涉及对象的统计信息，根据Query进行写相应的计算分析，然后再得出最后的执行计划。



### 4.2 MySQL常见瓶颈

- CPU:CPU在饱和的时候一般发生在数据装入在内存或从磁盘上读取数据时候
- IO:磁盘I/O瓶颈发生在装入数据远大于内存容量时
- 服务器硬件的性能瓶颈：top,free,iostat和vmstat来查看系统的性能状态



### 4.3 Explain

> https://segmentfault.com/a/1190000008131735



#### 是什么（查看执行计划）

使用EXPLAIN关键字可以模拟优化器执行SQL语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是结构的性能瓶颈。



#### 能干嘛

表的读取顺序

数据读取操作的操作类型

哪些索引可以使用

哪些索引被实际使用

表之间的引用

每张表有多少行被优化器查询



#### 怎么玩

Explain+SQL语句

执行计划包含的信息

![img](https://images2018.cnblogs.com/blog/512541/201808/512541-20180803142201303-545775900.png)

expain出来的信息有10列，分别是id、select_type、table、type、possible_keys、key、key_len、ref、rows、Extra

##### 概要描述：

```
id:选择标识符
select_type:表示查询的类型。
table:输出结果集的表
partitions:匹配的分区
type:表示表的连接类型
possible_keys:表示查询时，可能使用的索引
key:表示实际使用的索引
key_len:索引字段的长度
ref:列与索引的比较
rows:扫描出的行数(估算的行数)
filtered:按表条件过滤的行百分比
Extra:执行情况的描述和说明
```

**下面对这些字段出现的可能进行解释：**

##### 1.id

SELECT识别符。这是SELECT的查询序列号

**我的理解是SQL执行的顺序的标识，SQL从大到小的执行**

​		1.id相同时，执行顺序由上至下

![image-20201202131914359](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201202131914359.png)

​		2.如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

![image-20201202131931919](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201202131931919.png)

​		3.id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行

![image-20201202131954051](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201202131954051.png)



##### 2.select_type

   ***\*示查询中每个select子句的类型\****

(1) SIMPLE(简单SELECT，不使用UNION或子查询等)

(2) PRIMARY(子查询中最外层查询，查询中若包含任何复杂的子部分，最外层的select被标记为PRIMARY)

(3) UNION(UNION中的第二个或后面的SELECT语句)

(4) DEPENDENT UNION(UNION中的第二个或后面的SELECT语句，取决于外面的查询)

(5) UNION RESULT(UNION的结果，union语句中第二个select开始后面所有select)

(6) SUBQUERY(子查询中的第一个SELECT，结果不依赖于外部查询)

(7) DEPENDENT SUBQUERY(子查询中的第一个SELECT，依赖于外部查询)

(8) DERIVED(派生表的SELECT, FROM子句的子查询)

(9) UNCACHEABLE SUBQUERY(一个子查询的结果不能被缓存，必须重新评估外链接的第一行)

 

##### 3.table

显示这一步所访问数据库中表名称（显示这一行的数据是关于哪张表的），有时不是真实的表名字，可能是简称，例如上面的e，d，也可能是第几步执行的结果的简称

 

##### 4.type

对表访问方式，表示MySQL在表中找到所需行的方式，又称**访问类型**。

![image-20201202132947918](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201202132947918.png)

性能从最好到最差依次是：**system>const>eq_ref>ref>range>index>ALL**



system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，这个也可以忽略不计

const：表示通过索引一次就找到了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快。如将主键至于where列表中，MySQL就能将该查询转换为一个常量

eq_ref：唯一性索引，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引扫描

ref：非唯一索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体

NULL：MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

range：只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。一般就是在你的where语句中出现了between、<、>、in等的查询，这种范围扫描索引扫描比全表扫描要好，因为他只需要开始索引的某一点，而结束语另一点，不用扫描全部索引。

index：Full Index Scan,index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）

ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行

**备注：**一般来说，得保证查询只是达到range级别，最好达到ref



##### 5.possible_keys

**指出MySQL能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用（该查询可以利用的索引，如果没有任何索引显示 null）**

该列完全独立于EXPLAIN输出所示的表的次序。这意味着在possible_keys中的某些键实际上不能按生成的表次序使用。如果该列是NULL，则没有相关的索引。在这种情况下，可以通过检查WHERE子句看是否它引用某些列或适合索引的列来提高你的查询性能。如果是这样，创造一个适当的索引并且再次用EXPLAIN检查查询



##### 6.Key

**key列显示MySQL实际决定使用的键（索引），必然包含在possible_keys中**

如果没有选择索引，键是NULL。要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。



##### 7.key_len

**表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的）**

不损失精确性的情况下，长度越短越好 



##### 8.ref

**列与索引的比较，表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值**

 

##### 9.rows

 **估算出结果集行数，表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数**

 

##### 10.Extra

**该列包含MySQL解决查询的详细信息。**

**有以下几种情况：**

1.Using filesort：说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成排序操作成为“文件排序”

2.Using temporary：使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序order by 和分组查询 group by

3.Using index：表示相应的select操作中使用了覆盖索引（Coveing Index）,避免访问了表的数据行，效率不错！
						如果同时出现using where，表明索引被用来执行索引键值的查找；
						如果没有同时出现using where，表面索引用来读取数据而非执行查找动作。

​						**覆盖索引（Covering Index）**

![image-20201202134700929](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201202134700929.png)

4.Using where:不用读取表中所有信息，仅通过索引就可以获取所需数据，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示mysql服务器将在存储引擎检索行后再进行过滤

5.Using join buffer：改值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进能。

6.Impossible where：这个值强调了where语句会导致没有符合条件的行（通过收集统计信息不可能存在结果）。

7.Select tables optimized away：这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行

8.distinct：优化distinct，在找到第一匹配的元组后即停止找同样值的工作

9.No tables used：Query语句中使用from dual 或不含任何from子句

```
-- explain select now() from dual;
```

 

**总结：**
• EXPLAIN不会告诉你关于触发器、存储过程的信息或用户自定义函数对查询的影响情况
• EXPLAIN不考虑各种Cache
• EXPLAIN不能显示MySQL在执行查询时所作的优化工作
• 部分统计信息是估算的，并非精确值
• EXPALIN只能解释SELECT操作，其他操作要重写为SELECT后查看执行计划。**

通过收集统计信息不可能存在结果



热身case：

![image-20201202134800435](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201202134800435.png)

![image-20201202134808051](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201202134808051.png)



## 5.索引优化



### 索引分析



### 索引失效（应该避免）





### 一般性建议









# 三.查询截取分析

> 联表查询时始终以小结果集驱动大结果集：https://www.cnblogs.com/shamo89/p/8733025.html
>
> MySQL性能调优——Query 的优化：https://www.cnblogs.com/shamo89/p/8733139.html
>
> MySQL索引原理及慢查询优化：https://tech.meituan.com/2014/06/30/mysql-index.html

## 查询优化



## 慢查询日志



## 批量数据脚本



## Show profiles



## 全局查询日志







# 四.MySQL锁机制

## 概述





## 三锁





# 五.主从复制

> https://www.jianshu.com/p/faf0127f1cb2