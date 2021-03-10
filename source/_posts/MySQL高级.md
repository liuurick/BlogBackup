---
title: MySQL高级
date: 2020-11-22 13:55:41
tags: MySQL高级
categories: [编程基础,MySQL]
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

![image-20201122172644460](/images/2020112201.png)

上传并解压文件：

`tar -xvf MySQL-5.5.62-1.el7.x86_64.rpm-bundle.tar`

![image-20201122172644460](/images/2020112202.png)



安装：

```bash
rpm -ivh MySQL-server-5.5.62-1.el7.x86_64.rpm
rpm -ivh MySQL-client-5.5.62-1.el7.x86_64.rpm
```

注意：这里可能会出现错误：

> conflicts with file from package mariadb-libs-1:5.5.56-2.el7.x86_641

![image-20201122210437197](/images/2020112203.png)

与mariadb冲突，删除mariadb即可！

```bash
rpm -e mariadb-libs-1:5.5.65-1.el7.x86_64 --nodeps
```



### 1.4 查看Mysql安装时创建的mysql用户和mysql组

![image-20201122204405232](/images/2020112204.png)

或者也可以执行mysqladmin --version 命令，类似java -version，如果可以查看消息，说明成功

![image-20201122210850349](/images/2020112205.png)



### 1.5 mysql服务的启动和停止

```
service mysql start
service mysql stop
```

查看进程：

![image-20201122211604756](/images/2020112206.png)

**注意：**可能会报下面的错误

> Starting MySQL. ERROR! The server quit without updating PID file (/var/lib/mysql/localhost.localdomain.pid).

![image-20201122211244330](/images/2020112207.png)

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

![image-20201122212503860](/images/2020112208.png)



### 1.7 自启动mysql服务

```
chkconfig mysql on
```

或者

```
ntsysv
```

`chkconfig --list|grep mysql`

![image-20201122213038046](/images/2020112209.png)



### 1.8 修改配置文件位置

5.5版本：`cp /usr/share/mysql/my-huge.cnf /etc/my.cnf`

5.6版本：`cp /usr/share/mysql/my-default.cnf /etc/my.cnf`



### 1.9 修改字符集和数据存储路径

查看字符集：

```
show variables like '%char%';
show variables like '%character%';
```

![image-20201122220007001](/images/2020112210.png)

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

在linux下查看安装目录 `ps -ef|grep mysql`

![image-20201122212059724](/images/2020112211.png)



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

![image-20201123095017572](/images/2020112212.png)



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

**![img](/images/2020112213.png)**

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

![image-20201123100931321](/images/2020112214.png)



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

![img](/images/2020112215.png)

Join图

**内连接**：结果仅包含符合连接条件的两表中的行。

**外连接**：结果包含符合条件的行，同时包含不符合条件的行（分为左外连接、右外连接和全外连接）

![img](/images/2020112216.png)



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

- 索引(Index) 是帮助MySQL高效获取数据的数据结构。**索引的本质就是数据结构**。 **索引的目的在于提高查询效率**，可以类比字典，可以简单的理解为“排好序的快速查找数据结构”。
- 在数据本身之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用(指向)数据，这样就可以在这些数据结构上实现高级查找算法。这种数据结构就是索引。
- 一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上。
- 我们平时所说的索引，如果没有特别指明，都是指B树结构组织的索引。

 

**二叉树算法图：**

**[![img](/images/2020112217.png)](https://images2015.cnblogs.com/blog/1054685/201706/1054685-20170627085051383-985324526.png)**

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
   1. 最简单，通用的一种索引
2. 唯一索引：索引列的值必须唯一，但允许有空值。
   1. 银行卡号
   2. 身份证号
3. 复合索引：一个索引包含多个列。

 

#### 1.主键索引 PRIMARY KEY

它是一种特殊的唯一索引，不允许有空值。一般是在建表的时候同时创建主键索引。

**注意：**一个表只能有一个主键

![mark](/images/2020112218.png)



#### 2.唯一索引 UNIQUE

唯一索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。

可以通过`ALTER TABLE table_name ADD UNIQUE (column);`创建唯一索引

![mark](/images/2020112219.png)

![mark](/images/2020112220.png)

可以通过`ALTER TABLE table_name ADD UNIQUE (column1,column2);`创建唯一组合索引

![mark](/images/2020112221.png)

![mark](/images/2020112222.png)

#### 3.普通索引 INDEX

最基本的索引，它没有任何限制。

可以通过`ALTER TABLE table_name ADD INDEX index_name (column);`创建普通索引

![mark](/images/2020112223.png)

![mark](/images/2020112224.png)

#### 4.组合索引 INDEX

组合索引，即一个索引包含多个列。多用于避免回表查询。

可以通过`ALTER TABLE table_name ADD INDEX index_name(column1, column2, column3);`创建组合索引

![mark](/images/2020112225.png)

![mark](/images/2020112226.png)

#### 5.全文索引 FULLTEXT

全文索引（也称全文检索）是目前搜索引擎使用的一种关键技术。

可以通过`ALTER TABLE table_name ADD FULLTEXT (column);`创建全文索引

![mark](/images/2020112227.png)

![mark](/images/2020112228.png)

索引一经创建不能修改，如果要修改索引，只能删除重建。可以使用`DROP INDEX index_name ON table_name;`删除索引。



### 3.5 索引的基本语法

#### 创建

```sql
CREATE [UNIQUE] INDEX indexName ON tableName(colName(length));。——如果是CHAR、VARCHAR类型，length可以小于字段实际长度，若是BLOB和TEXT类型，必须指定length。

ALTER tableName ADD [UNIQUE] INDEX indexName ON (colName(length));。
```

#### 删除

```sql
DROP INDEX [indexName] ON tableName;　　
```

#### 查看

```sql
SHOW INDEX FROM tableName;
```

#### 使用ALTER命令四种方式添加数据表的索引

```sql
ALTER TABLE tableName ADD PRIMARY KEY(column_list); 该语句添加一个主键，这意味着索引值必须唯一，且不能为null。
ALTER TABLE tableName ADD UNIQUE indexName(column_list);该语句创建索引的值必须唯一(null除外，null可能会出现多次)。
ALTER TABLE tableName ADD INDEX indexName(column_list);该语句添加普通索引，索引值可出现多次。
AKTER TABLE tableName ADD FULLTEXT indexName(column_list);该语句指定索引为FULLTEXT，用于全文索引。
```

 

### 3.6 MySQL索引结构

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

   ![image-20201129201618523](/images/2020112229.png)



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

![img](/images/2020112230.png)

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

![image-20201202131914359](/images/2020112231.png)

​		2.如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

![image-20201202131931919](/images/2020112232.png)

​		3.id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行

![image-20201202131954051](/images/2020112233.png)



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

![image-20201202132947918](/images/2020112234.png)

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

![image-20201202134700929](/images/2020112235.png)

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



**热身case**：

![image-20201202134800435](/images/2020112236.png)



![image-20201202134808051](/images/2020112237.png)



## 5.索引优化

索引分析之后进行索引优化

### 索引分析



### 索引失效（应该避免）





### 一般性建议









# 三.查询截取分析

> 联表查询时始终以小结果集驱动大结果集:https://www.cnblogs.com/shamo89/p/8733025.html
>
> MySQL性能调优——Query 的优化:https://www.cnblogs.com/shamo89/p/8733139.html

## 3.1 查询优化

### 3.1.1 分析方法

1. 慢查询的开启并捕获
2. explain+慢SQL分析
3. `show profile`查询SQL在MySQL服务器里面的执行细节和生命周期情况
4. SQL数据库服务器的参数调优



### 3.1.2优化原则

**小表驱动大表**（即小的数据集驱动大的数据集）

in和exists的使用：

- B表的数据集必须小于A表数据集的时候，in优于exists

```sql
select * from A where id in (select id from B)
```

- A表的数据集小于B表数据集的时候，exists优于in

```sql
select * from A where exists (select 1 from B where A.id = B.id)
```

**注意**：A表与B表的ID字段应建立索引



**EXISTS的使用**

```sql
select ... from table where exists (子查询)
```

理解为：将主查询的数据，放到子查询中做条件验证，将验证的结果(true或false)来决定主查询的数据结果是否得以保留

**注意：**

1. EXISTS（subquery）只返回TRUE或FALSE，因此子查询中的SELECT * 也可以是SELECT 1或SELECT 'X'，官方说法是实际执行会忽略SELECT清单，因此没有区别。
2. EXISTS子查询的实际查询执行过程经过了优化而不是我们理解上的逐条对比，如果担忧效率问题，可进行实际检验以确定是否有效率问题。
3. EXISTS子查询往往也可以用条件表达式、其他子查询或者JOIN来替代，何种最优需要具体问题具体分析。



### 3.1.3 ORDER BY关键字优化

SQL支持两种方式的排序，`FileSort`和`Index`

- ORDER BY子句，尽量使用index方式排序，避免使用FileSort方式排序
  　MySQL支持二种方式的排序，FileSort和Index，Index效率高，它指MySQL扫描索引本身完成排序。FileSort方式效率较低。(用explain可以在extra字段里看到Using index/filesort)
    　Order By满足两种情况，会使用Index方式排序
    　　　1.Order by语句使用索引最左前列
    　　　2.使用where子句与Order by子句条件组合满足索引最左前列。
  
- 尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀
- 如果不在索引列上，fileSort有两种算法：mysql就要启动**双路排序**和**单路排序**
  　　**双路排序**：MySQL4.1之前是使用双路排序，字面意思就是 **两次扫描磁盘**，最终得到数据，读取行指针和orderby列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出。从磁盘取排序字段，在buffer进行排序，再从磁盘取其他字段。取一批数据，要对磁盘进行了两次扫描，众所周知，I/O是很耗时的，所以在mysql4.1后，出现了改进算法，就是单路排序
    　　**单路排序**：从磁盘读取查询需要的所有列，按照order by列在buffer对它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO，但是它会使用更多的空间，因为它把每一行都保存在内存中了。
    　　结论及引申出的问题：由于单路是后出的，总体而言好过双路，但是单路也有问题。

> 单路的问题
> 　　在sort_buffer中，方法B比方法A要多占用很多空间，因为方法B是把所有字段都取出，所以有可能取出的数据的总大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据，进行排序(创建tmp文件，多路合并),排完再取sort_buffer容量大小，再排…..从而多次IO.
> 　　本来想省一次IO操作，反而导致了大量的I/O操作，反而得不偿失。

- 总结
  **MySql两种排序方式：文件排序(filesort)或扫描有序索引排序(index)**
  **MySql能为排序与查询使用相同的索引**

```sql
KEY a_b_c(a, b, c)
 
order by 能使用索引最左前缀
- order by a
- order by a, b
- order by a desc, c desc
 
如果where使用索引的最左前缀定义为常量，则order by 能使用索引
- where a = const order by b, c
- where a = const and b = const order by c
- where a = const order by b,c
- where a = const and b > const order by b, c
 
不能使用索引进行排序
- order by a asc, b desc, c desc /* 排序不一致 */
- where g = const order by b, c /* 丢失a索引 */
- where a = const order by c /* 丢失b索引 */
- where a = const order by a, d /* d不是索引的一部分 */
- where a in (..) order by b, c /* 对于排序来说，多个相等条件也是范围查询(in 也是范围查询)！！ */
```

### 3.1.4 GROUP BY关键字优化

基本与 order by 优化一致

1. group by 实质是先排序后分组，遵照索引建的最佳左前缀
2. 当无法使用索引列，增大 `max_length_for_sort_data` 参数的设置 + 增大`sort_buffer_size`参数的设置
3. where高于having，能写在where限定的条件就不要去having限定了。

## 3.2 慢查询日志

### 3.2.1 是什么

　　MySql的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中
　　long_query_time 的默认值为10，意思是运行10秒以上的语句。
　　由它来查看哪些SQL超出了我们的最大忍耐时间值，比如一条sql执行超过5秒种，我们就算慢SQL，希望能收集超过5秒的sql，结合之前explain进行全面分析。 　　

### 3.2.2 怎么用

1.说明
　　**默认情况下，MySQL数据库没有开启慢查询日志**，需要我们手动来设置这个参数。
　　**当然，如果不是调优需要的话，一般不建议启动该参数**，因为开启慢查询日志会或多或少带来一定的性能影响。慢查询日志支持将日志记录写入文件。

2.查看是否开启及如何开启
　　默认：`show variables like '%slow_query_log%';`
　　开启：`set global show_query_log=1;`，这个 **只对当前数据库生效**，如果MySQL重启后则会失效。如果要永久生效，必须修改配置文件my.cnf(其他系统变量也是如此)

3.开启慢查询后，什么样的SQL才会记录到慢查询日志里面呢？
　　这个是由参数long_query_time控制，默认情况下long_query_time的值为10秒，
　　命令：`show variables like ‘long_query_time%;’`。可以使用命令修改，也可以在my.cnf参数里面修改。
　　假如运行时间正好等于 `long_query_time `的情况，并不会被记录。也就是说，在mysql源码里是 **判断>long_query_time，而非>=**.

### 3.2.3 日志分析工具 mysqldumpslow

　　在生产环境中，如果要手工分析日志，查找、分析SQL，显然是个体力活，MySQL提供了日志分析工具 mysqldumpslow
查看 mysqldumpslow 的帮助信息：`mysqldumpslow --help`
　　s: 表示按何种方式排序
　　c: 访问次数
　　l: 锁定时间
　　r: 返回记录
　　t: 查询时间
　　al: 平均锁定时间
　　ar: 平均返回记录数
　　at: 平均查询时间
　　t: 返回前面多少条的数据
　　g: 后边搭配一个正则匹配模式，大小写不敏感的。

```sql
得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/show.log
 
得到访问次数最多的10个SQL
mysqldumpslow -s c -t 10 /var/lig/mysql/show.log
 
得到按照时间排序的前10条里面含有左连接的查询语句
mysqldumpslow -s t -t 10 -g "left join" /var/lig/mysql/show.log
 
另外建议在使用这些命令时结构 | 和more使用，否则有可能出现爆屏情况
mysqldumpslow -s r -t 10 /var/lig/mysql/show.log | more
```

## 3.3 批量数据脚本

往表里插入1000W数据

### 3.3.1 建表

```sql
# 新建库
create database bigData;
use bigData;
 
#1 建表dept
CREATE TABLE dept(  
    id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,  
    deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,   
    dname VARCHAR(20) NOT NULL DEFAULT "",  
    loc VARCHAR(13) NOT NULL DEFAULT ""  
) ENGINE=INNODB DEFAULT CHARSET=UTF8 ;  

#2 建表emp
CREATE TABLE emp  
(  
    id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,  
    empno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0, /*编号*/  
    ename VARCHAR(20) NOT NULL DEFAULT "", /*名字*/  
    job VARCHAR(9) NOT NULL DEFAULT "",/*工作*/  
    mgr MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,/*上级编号*/  
    hiredate DATE NOT NULL,/*入职时间*/  
    sal DECIMAL(7,2) NOT NULL,/*薪水*/  
    comm DECIMAL(7,2) NOT NULL,/*红利*/  
    deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 /*部门编号*/  
)ENGINE=INNODB DEFAULT CHARSET=UTF8 ;
```

### 3.3.2 设置参数`log_trust_function_createors`

​		当开启二进制日志后，如果变量`log_bin_trust_function_creators`为OFF，那么创建或修改存储函数就会报“ERROR 1418 (HY000): This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)”这样的错误。因为二进制日志的一个重要功能是用于`主从复制`，而存储函数有可能导致主从的数据不一致。

​		所以当开启二进制日志后，参数log_bin_trust_function_creators就会生效，限制存储函数的创建、修改、调用。

```
show variables like 'log_bin_trust_function_creators';
set global log_bin_trust_function_creators=1;
```

这样添加了参数以后，如果mysqld重启，上述参数又会消失，永久方法：

```
windows下my.ini[mysqld]加上log_bin_trust_function_creators=1
linux下 /etc/my.cnf下my.cnf[mysqld]加上log_bin_trust_function_creators=1
```

### 3.3.3 创建函数保证每条数据都不同

- 随机产生字符串

  ```sql
  DELIMITER $$
  CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
  BEGIN    ##方法开始
   DECLARE chars_str VARCHAR(100) DEFAULT   'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ'; 
   ##声明一个 字符窜长度为 100 的变量 chars_str ,默认值 
   DECLARE return_str VARCHAR(255) DEFAULT '';
   DECLARE i INT DEFAULT 0;
  ##循环开始
   WHILE i < n DO  
   SET return_str =CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1));
  ##concat 连接函数  ，substring(a,index,length) 从index处开始截取
   SET i = i + 1;
   END WHILE;
   RETURN return_str;
  END $$
   
  #假如要删除
  #drop function rand_string;
  ```

- 随机产生部门编号

  ```sql
  #用于随机产生部门编号
  DELIMITER $$
  CREATE FUNCTION rand_num( ) 
  RETURNS INT(5)  
  BEGIN   
   DECLARE i INT DEFAULT 0;  
   SET i = FLOOR(100+RAND()*10);  
  RETURN i;  
   END $$
   
   
  #假如要删除
  #drop function rand_num;
  ```

### 3.3.4 创建存储过程

- 创建往emp表中插入数据的存储过程

  ```sql
  DELIMITER $$
  CREATE PROCEDURE insert_emp10000(IN START INT(10),IN max_num INT(10))  
  BEGIN  
  DECLARE i INT DEFAULT 0;   
  #set autocommit =0 把autocommit设置成0  ；提高执行效率
   SET autocommit = 0;    
   REPEAT  ##重复
   SET i = i + 1;  
   INSERT INTO emp10000 (empno, ename ,job ,mgr ,hiredate ,sal ,comm ,deptno ) VALUES ((START+i) ,rand_string(6),'SALESMAN',0001,CURDATE(),FLOOR(1+RAND()*20000),FLOOR(1+RAND()*1000),rand_num());  
   UNTIL i = max_num   ##直到  上面也是一个循环
   END REPEAT;  ##满足条件后结束循环
   COMMIT;   ##执行完成后一起提交
   END $$
   
  #删除
  # DELIMITER ;
  # drop PROCEDURE insert_emp;
  ```

  

- 创建往dept表中插入数据的存储过程

  ```sql
  #执行存储过程，往dept表添加随机数据
  DELIMITER $$
  CREATE PROCEDURE insert_dept(IN START INT(10),IN max_num INT(10))  
  BEGIN  
  DECLARE i INT DEFAULT 0;   
   SET autocommit = 0;    
   REPEAT  
   SET i = i + 1;  
   INSERT INTO dept (deptno ,dname,loc ) VALUES (START +i ,rand_string(10),rand_string(8));  
   UNTIL i = max_num  
   END REPEAT;  
   COMMIT;  
   END $$ 
   
  #删除
  # DELIMITER ;
  # drop PROCEDURE insert_dept;
  ```

### 3.3.5 调用存储过程

```sql
DELIMITER ;
CALL insert_dept(100,10); 

#执行存储过程，往emp表添加50万条数据
DELIMITER ;    #将 结束标志换回 ;
CALL insert_emp(100001,500000); 
CALL insert_emp10000(100001,10000);
```

## 3.4 show profile

这个是sql分析最强大的
默认情况下，参数处于关闭状态，并保存最近15次的运行结果

### 3.4.1 是什么
是mysql提供可以用来分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优的测量

### 3.4.2 分析步骤

1. 是否支持，看看当前的mysql版本是否支持
   `show variables like 'profiling%'`

2. 开启功能，默认是关闭，使用前需要开启
   `set profiling=on`

3. 运行SQL
   `select * from emp group by id%10 limit 1500000`
   `select * from emp group by id%20 order by 5`

4. 查看结果，`show profiles`

5. 诊断SQL，`show profile cpu, block io for query 2` 后的数字是 show profiles 里的query_id
   **参数备注**：

   ```
   all: 显示所有的开销信息
   block io: 显示块IO相关开销
   context switches: 上下文切换相关开销
   cpu: 显示CPU相关开销信息
   ipc: 显示发送和接收相关开销信息
   memory: 显示内存相关开销信息
   page faults: 显示页面错误相关开销信息
   source: 显示和source_function, source_file, souce_line相关的开销信息
   swaps: 显示交换次数相关开销的信息
   ```

6. 日常开发需要注意的结论

   ```
   converting HEAP to MyISAM 查询结果太长，内存都不够用了往磁盘上搬了。
   Creating tmp table 创建临时表：copy数据到临时表，用完再删除
   Copying to tmp table on disk 把内存中临时表复制到磁盘，很危险！！！
   locaked
   ```

## 3.5 全局查询日志

永远不要在生产环境上打开，测试时可以

### 3.5.1 配置启用

在mysql的my.cnf中，设置如下：

```sql
# 开启
general_log=1 

# 记录日志文件的路径
general_log_file=/path/logfile 

# 输出格式
log_output=FILE
```


### 3.5.2 编码启用

```sql
set global general_log=1;
set global log_output='TABLE';
```

此后，你所编写的sql语句，将会记录到mysql库里的general_log表。 
可以用下面的命令查看

```sql
select * from mysql.general_log;
```



# 四.MySQL锁机制

## 4.1 概述

### 4.1.1 锁的定义

1. 锁是计算机协调**多个进程或线程并发访问某一资源的机制**。
2. 在数据库中，除传统的计算资源（如CPU、RAM、I/O等）的争用以外，数据也是一种供许多用户共享的资源。
3. 如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。
4. 从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

### 4.1.2 锁的分类

1. 从数据操作的类型（读、写）分
   - **读锁**（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响
   - **写锁**（排它锁）：当前写操作没有完成前，它会阻断其他写锁和读锁。
2. 从对数据操作的颗粒度
   - 表锁
   - 行锁

## 4.2 表锁

**表锁的特点**:

**偏向MyISAM存储引擎**，开销小，加锁快，无死锁，锁定粒度大，**发生锁冲突的概率最高，并发最低**

### 4.2.1 表锁案例分析

**创建表**

- 建表 SQL：**引擎选择 myisam**

```sql
create table mylock (
    id int not null primary key auto_increment,
    name varchar(20) default ''
) engine myisam;

insert into mylock(name) values('a');
insert into mylock(name) values('b');
insert into mylock(name) values('c');
insert into mylock(name) values('d');
insert into mylock(name) values('e');

select * from mylock;
```

- mylock 表中的测试数据

```sql
mysql> select * from mylock;
+----+------+
| id | name |
+----+------+
|  1 | a    |
|  2 | b    |
|  3 | c    |
|  4 | d    |
|  5 | e    |
+----+------+
5 rows in set (0.00 sec)
```

**手动加锁和释放锁**

- 查看当前数据库中表的上锁情况：`show open tables;`，0 表示未上锁

```sql
mysql> show open tables;
+--------------------+----------------------------------------------------+--------+-------------+
| Database           | Table                                              | In_use | Name_locked |
+--------------------+----------------------------------------------------+--------+-------------+
| performance_schema | events_waits_history                               |      0 |           0 |
| performance_schema | events_waits_summary_global_by_event_name          |      0 |           0 |
| performance_schema | setup_timers                                       |      0 |           0 |
| performance_schema | events_waits_history_long                          |      0 |           0 |
| performance_schema | events_statements_summary_by_digest                |      0 |           0 |
| performance_schema | mutex_instances                                    |      0 |           0 |
| performance_schema | events_waits_summary_by_instance                   |      0 |           0 |
| performance_schema | events_stages_history                              |      0 |           0 |
| mysql              | db                                                 |      0 |           0 |
| performance_schema | events_waits_summary_by_host_by_event_name         |      0 |           0 |
| mysql              | user                                               |      0 |           0 |
| mysql              | columns_priv                                       |      0 |           0 |
| performance_schema | events_statements_history_long                     |      0 |           0 |
| performance_schema | performance_timers                                 |      0 |           0 |
| performance_schema | file_instances                                     |      0 |           0 |
| performance_schema | events_stages_summary_by_user_by_event_name        |      0 |           0 |
| performance_schema | events_stages_history_long                         |      0 |           0 |
| performance_schema | setup_actors                                       |      0 |           0 |
| performance_schema | cond_instances                                     |      0 |           0 |
| mysql              | proxies_priv                                       |      0 |           0 |
| performance_schema | socket_summary_by_instance                         |      0 |           0 |
| performance_schema | events_statements_current                          |      0 |           0 |
| mysql              | event                                              |      0 |           0 |
| performance_schema | session_connect_attrs                              |      0 |           0 |
| mysql              | plugin                                             |      0 |           0 |
| performance_schema | threads                                            |      0 |           0 |
| mysql              | time_zone_transition_type                          |      0 |           0 |
| mysql              | time_zone_name                                     |      0 |           0 |
| performance_schema | file_summary_by_event_name                         |      0 |           0 |
| performance_schema | events_waits_summary_by_user_by_event_name         |      0 |           0 |
| performance_schema | socket_summary_by_event_name                       |      0 |           0 |
| performance_schema | users                                              |      0 |           0 |
| mysql              | servers                                            |      0 |           0 |
| performance_schema | events_waits_summary_by_account_by_event_name      |      0 |           0 |
| db01               | tbl_emp                                            |      0 |           0 |
| performance_schema | events_statements_summary_by_host_by_event_name    |      0 |           0 |
| db01               | tblA                                               |      0 |           0 |
| performance_schema | table_io_waits_summary_by_index_usage              |      0 |           0 |
| performance_schema | events_waits_current                               |      0 |           0 |
| db01               | user                                               |      0 |           0 |
| mysql              | procs_priv                                         |      0 |           0 |
| performance_schema | events_statements_summary_by_thread_by_event_name  |      0 |           0 |
| db01               | emp                                                |      0 |           0 |
| db01               | tbl_user                                           |      0 |           0 |
| db01               | test03                                             |      0 |           0 |
| mysql              | slow_log                                           |      0 |           0 |
| performance_schema | file_summary_by_instance                           |      0 |           0 |
| db01               | article                                            |      0 |           0 |
| performance_schema | objects_summary_global_by_type                     |      0 |           0 |
| db01               | phone                                              |      0 |           0 |
| performance_schema | events_waits_summary_by_thread_by_event_name       |      0 |           0 |
| performance_schema | setup_consumers                                    |      0 |           0 |
| performance_schema | socket_instances                                   |      0 |           0 |
| performance_schema | rwlock_instances                                   |      0 |           0 |
| db01               | tbl_dept                                           |      0 |           0 |
| performance_schema | events_statements_summary_by_user_by_event_name    |      0 |           0 |
| db01               | staffs                                             |      0 |           0 |
| db01               | class                                              |      0 |           0 |
| mysql              | general_log                                        |      0 |           0 |
| performance_schema | events_stages_summary_global_by_event_name         |      0 |           0 |
| performance_schema | events_stages_summary_by_account_by_event_name     |      0 |           0 |
| performance_schema | events_statements_summary_by_account_by_event_name |      0 |           0 |
| performance_schema | table_lock_waits_summary_by_table                  |      0 |           0 |
| performance_schema | hosts                                              |      0 |           0 |
| performance_schema | setup_objects                                      |      0 |           0 |
| performance_schema | events_stages_current                              |      0 |           0 |
| mysql              | time_zone                                          |      0 |           0 |
| mysql              | tables_priv                                        |      0 |           0 |
| performance_schema | table_io_waits_summary_by_table                    |      0 |           0 |
| mysql              | time_zone_leap_second                              |      0 |           0 |
| db01               | book                                               |      0 |           0 |
| performance_schema | session_account_connect_attrs                      |      0 |           0 |
| db01               | mylock                                             |      0 |           0 |
| mysql              | func                                               |      0 |           0 |
| performance_schema | events_statements_summary_global_by_event_name     |      0 |           0 |
| performance_schema | events_statements_history                          |      0 |           0 |
| performance_schema | accounts                                           |      0 |           0 |
| mysql              | time_zone_transition                               |      0 |           0 |
| db01               | dept                                               |      0 |           0 |
| performance_schema | events_stages_summary_by_host_by_event_name        |      0 |           0 |
| performance_schema | events_stages_summary_by_thread_by_event_name      |      0 |           0 |
| mysql              | proc                                               |      0 |           0 |
| performance_schema | setup_instruments                                  |      0 |           0 |
| performance_schema | host_cache                                         |      0 |           0 |
+--------------------+----------------------------------------------------+--------+-------------+
84 rows in set (0.00 sec)

```

- 添加锁

```sql
lock table 表名1 read(write), 表名2 read(write), ...;
```

- 释放表锁

```sql
unlock tables;
```

#### 读锁示例

- 在 session 1 会话中，给 mylock 表加个读锁

```sql
mysql> lock table mylock read;
Query OK, 0 rows affected (0.00 sec)
```

- 在 session1 会话中能不能读取 mylock 表：可以读

```sql
################# session1 中的操作 #################

mysql> select * from mylock;
+----+------+
| id | name |
+----+------+
|  1 | a    |
|  2 | b    |
|  3 | c    |
|  4 | d    |
|  5 | e    |
+----+------+
5 rows in set (0.00 sec)
```

- 在 session1 会话中能不能读取 book 表：并不行。。。

```sql
################# session1 中的操作 #################

mysql> select * from book;
ERROR 1100 (HY000): Table 'book' was not locked with LOCK TABLES
```

- 在 session2 会话中能不能读取 mylock 表：可以读

```sql
################# session2 中的操作 #################

mysql> select * from mylock;
+----+------+
| id | name |
+----+------+
|  1 | a    |
|  2 | b    |
|  3 | c    |
|  4 | d    |
|  5 | e    |
+----+------+
5 rows in set (0.00 sec)
```

- 在 session1 会话中能不能修改 mylock 表：并不行。。。

```sql
################# session1 中的操作 #################

mysql> update mylock set name='a2' where id=1;
ERROR 1099 (HY000): Table 'mylock' was locked with a READ lock and can't be updated
```

- 在 session2 会话中能不能修改 mylock 表：阻塞，一旦 mylock 表锁释放，则会执行修改操作

```sql
################# session2 中的操作 #################

mysql> update mylock set name='a2' where id=1;
# 在这里阻塞着呢~~~
```

**结论**

1. 当前 session 和其他 session 均可以读取加了读锁的表
2. 当前 session 不能读取其他表，并且不能修改加了读锁的表
3. 其他 session 想要修改加了读锁的表，必须等待其读锁释放

#### 写锁示例

- 在 session 1 会话中，给 mylock 表加个写锁

```sql
mysql> lock table mylock write;
Query OK, 0 rows affected (0.00 sec)
```

- 在 session1 会话中能不能读取 mylock 表：阔以

```sql
################# session1 中的操作 #################

mysql> select * from mylock;
+----+------+
| id | name |
+----+------+
|  1 | a2   |
|  2 | b    |
|  3 | c    |
|  4 | d    |
|  5 | e    |
+----+------+
5 rows in set (0.00 sec)
```

- 在 session1 会话中能不能读取 book 表：不阔以

```sql
################# session1 中的操作 #################

mysql> select * from book;
ERROR 1100 (HY000): Table 'book' was not locked with LOCK TABLES
```

- 在 session1 会话中能不能修改 mylock 表：当然可以啦，加写锁就是为了修改呀

```
################# session1 中的操作 #################

mysql> update mylock set name='a2' where id=1;
Query OK, 0 rows affected (0.00 sec)
Rows matched: 1  Changed: 0  Warnings: 0
```

- 在 session2 会话中能不能读取 mylock 表：

```sql
################# session2 中的操作 #################

mysql> select * from mylock;
# 在这里阻塞着呢~~~
```

**结论**

1. 当前 session 可以读取和修改加了写锁的表
2. 当前 session 不能读取其他表
3. 其他 session 想要读取加了写锁的表，必须等待其读锁释放

> **案例结论**

1. MyISAM在执行查询语句（SELECT）前，会自动给涉及的所有表加**读锁**，在执行增删改操作前，会自动给涉及的表加**写锁**。
2. MySQL的表级锁有两种模式：
   - 表共享读锁（Table Read Lock）
   - 表独占写锁（Table Write Lock）

![image-20200805154049814](/images/2020112238.png)

------

结论：结合上表，所以对MyISAM表进行操作，会有以下情况：

1. 对MyISAM表的读操作（加读锁），不会阻塞其他进程对同一表的读请求，但会阻塞对同一表的写请求。只有当读锁释放后，才会执行其它进程的写操作。
2. 对MyISAM表的写操作（加写锁），会阻塞其他进程对同一表的读和写操作，只有当写锁释放后，才会执行其它进程的读写操作
3. 简而言之，就是读锁会阻塞写，但是不会堵塞读。而写锁则会把读和写都堵塞。

### 4.2.2 表锁分析

- 查看哪些表被锁了，0 表示未锁，1 表示被锁

```sql
show open tables;
```

------

【如何分析表锁定】可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定，通过 `show status like 'table%';` 命令查看

1. Table_locks_immediate：产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1；
2. Table_locks_waited：出现表级锁定争用而发生等待的次数（不能立即获取锁的次数，每等待一次锁值加1），此值高则说明存在着较严重的表级锁争用情况；

```sql
mysql> show status like 'table%';
+----------------------------+--------+
| Variable_name              | Value  |
+----------------------------+--------+
| Table_locks_immediate      | 500440 |
| Table_locks_waited         | 1      |
| Table_open_cache_hits      | 500070 |
| Table_open_cache_misses    | 5      |
| Table_open_cache_overflows | 0      |
+----------------------------+--------+
5 rows in set (0.00 sec)
```

- 此外，Myisam的读写锁调度是写优先，这也是myisam不适合做写为主表的引擎。因为写锁后，其他线程不能做任何操作，**大量的更新会使查询很难得到锁**，从而造成永远阻塞

## 4.3 行锁

**行锁的特点:**

1. **偏向InnoDB存储引擎**，开销大，加锁慢；会出现死锁；**锁定粒度最小，发生锁冲突的概率最低，并发度也最高**。
2. InnoDB与MyISAM的最大不同有两点：**一是支持事务（TRANSACTION）；二是采用了行级锁**

### 4.3.1 事务复习

> **行锁支持事务，复习下老知识**

**事务（Transation）及其ACID属性**

事务是由一组SQL语句组成的逻辑处理单元，事务具有以下4个属性，通常简称为事务的ACID属性。

1. **原子性**（Atomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。
2. **一致性**（Consistent）：在事务开始和完成时，数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改，以保持数据的完整性；事务结束时，所有的内部数据结构（如B树索引或双向链表）也都必须是正确的。
3. **隔离性**（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然。
4. **持久性**（Durability）：事务院成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

------

**并发事务处理带来的问题**

1. 更新丢失

   （Lost Update）：

   - 当两个或多个事务选择同一行，然后基于最初选定的值更新该行时，由于每个事务都不知道其他事务的存在，就会发生丢失更新问题一一最后的更新覆盖了由其他事务所做的更新。
   - 例如，两个程序员修改同一java文件。每程序员独立地更改其副本，然后保存更改后的副本，这样就覆盖了原始文档。最后保存其更改副本的编辑人员覆盖前一个程序员所做的更改。
   - 如果在一个程序员完成并提交事务之前，另一个程序员不能访问同一文件，则可避免此问题。

2. 脏读

   （Dirty Reads）：

   - 一个事务正在对一条记录做修改，在这个事务完成并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做”脏读”。
   - 一句话：事务A读取到了事务B已修改但尚未提交的的数据，还在这个数据基础上做了操作。此时，如果B事务回滚，A读取的数据无效，不符合一致性要求。

3. 不可重复读

   （Non-Repeatable Reads）：

   - 一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现其读出的数据已经发生了改变、或某些记录已经被删除了！这种现象就叫做“不可重复读”。
   - 一句话：事务A读取到了事务B已经提交的修改数据，不符合隔离性

4. 幻读

   （Phantom Reads）

   - 一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读一句话：事务A读取到了事务B体提交的新增数据，不符合隔离性。
   - 多说一句：幻读和脏读有点类似，脏读是事务B里面修改了数据，幻读是事务B里面新增了数据。

------

**事物的隔离级别**

1. 脏读”、“不可重复读”和“幻读”，其实都是数据库读一致性问题，必须由数据库提供一定的事务隔离机制来解决。
2. 数据库的事务隔离越严格，并发副作用越小，但付出的代价也就越大，因为事务隔离实质上就是使事务在一定程度上“串行化”进行，这显然与“并发”是矛盾的。
3. 同时，不同的应用对读一致性和事务隔离程度的要求也是不同的，比如许多应用对“不可重复读”和“幻读”并不敏感，可能更关心数据并发访问的能力。
4. 查看当前数据库的事务隔离级别：`show variables like 'tx_isolation';` mysql 默认是可重复读

![image-20200805155415247](/images/2020112239.png)

### 4.3.2 行锁案例

**行锁案例分析：**

**创建表**

- 建表 SQL

```sql
CREATE TABLE test_innodb_lock (a INT(11),b VARCHAR(16))ENGINE=INNODB;

INSERT INTO test_innodb_lock VALUES(1,'b2');
INSERT INTO test_innodb_lock VALUES(3,'3');
INSERT INTO test_innodb_lock VALUES(4, '4000');
INSERT INTO test_innodb_lock VALUES(5,'5000');
INSERT INTO test_innodb_lock VALUES(6, '6000');
INSERT INTO test_innodb_lock VALUES(7,'7000');
INSERT INTO test_innodb_lock VALUES(8, '8000');
INSERT INTO test_innodb_lock VALUES(9,'9000');
INSERT INTO test_innodb_lock VALUES(1,'b1');

CREATE INDEX test_innodb_a_ind ON test_innodb_lock(a);
CREATE INDEX test_innodb_lock_b_ind ON test_innodb_lock(b);
```

- test_innodb_lock 表中的测试数据

```
mysql> select * from test_innodb_lock;
+------+------+
| a    | b    |
+------+------+
|    1 | b2   |
|    3 | 3    |
|    4 | 4000 |
|    5 | 5000 |
|    6 | 6000 |
|    7 | 7000 |
|    8 | 8000 |
|    9 | 9000 |
|    1 | b1   |
+------+------+
9 rows in set (0.00 sec)
```



```sql
mysql> SHOW INDEX FROM test_innodb_lock;
+------------------+------------+------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table            | Non_unique | Key_name               | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+------------------+------------+------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| test_innodb_lock |          1 | test_innodb_a_ind      |            1 | a           | A         |           9 |     NULL | NULL   | YES  | BTREE      |         |               |
| test_innodb_lock |          1 | test_innodb_lock_b_ind |            1 | b           | A         |           9 |     NULL | NULL   | YES  | BTREE      |         |               |
+------------------+------------+------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.00 sec)

```

**操作同一行数据：**

- session1 开启事务，修改 test_innodb_lock 中的数据

```sql
mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_innodb_lock set b='4001' where a=4;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

- session2 开启事务，修改 test_innodb_lock 中同一行数据，将导致 session2 发生阻塞，一旦 session1 提交事务，session2 将执行更新操作

```sql
mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_innodb_lock set b='4002' where a=4;
# 在这儿阻塞着呢~~~

# 时间太长，会报超时错误哦
mysql> update test_innodb_lock set b='4001' where a=4;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

**操作不同行数据：**

- session1 开启事务，修改 test_innodb_lock 中的数据

```sql
mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_innodb_lock set b='4001' where a=4;
Query OK, 0 rows affected (0.00 sec)
Rows matched: 1  Changed: 0  Warnings: 0
```

- session2 开启事务，修改 test_innodb_lock 中不同行的数据
- 由于采用行锁，session2 和 session1 互不干涉，所以 session2 中的修改操作没有阻塞

```sql
mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_innodb_lock set b='9001' where a=9;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

**无索引导致行锁升级为表锁：**

- session1 开启事务，修改 test_innodb_lock 中的数据，varchar 不用 ’ ’ ，导致系统自动转换类型，导致索引失效

```sql
mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_innodb_lock set a=44 where b=4000;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

- session2 开启事务，修改 test_innodb_lock 中不同行的数据
- 由于发生了自动类型转换，索引失效，导致行锁变为表锁

```sql
mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_innodb_lock set b='9001' where a=9;
# 在这儿阻塞着呢~~~
```

### 4.3.3 间隙锁

**什么是间隙锁：**

1. 当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP）”
2. InnoDB也会对这个“间隙”加锁，这种锁机制是所谓的间隙锁（Next-Key锁）

**间隙锁的危害**：

1. 因为Query执行过程中通过过范围查找的话，他会锁定整个范围内所有的索引键值，即使这个键值并不存在。
2. 间隙锁有一个比较致命的弱点，就是当锁定一个范围键值之后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据。在某些场景下这可能会对性能造成很大的危害

------

**间隙锁示例**：

- test_innodb_lock 表中的数据

```sql
mysql> select * from test_innodb_lock;
+------+------+
| a    | b    |
+------+------+
|    1 | b2   |
|    3 | 3    |
|    4 | 4000 |
|    5 | 5000 |
|    6 | 6000 |
|    7 | 7000 |
|    8 | 8000 |
|    9 | 9000 |
|    1 | b1   |
+------+------+
9 rows in set (0.00 sec)
```

- session1 开启事务，执行修改 a > 1 and a < 6 的数据，这会导致 mysql 将 a = 2 的数据行锁住（虽然表中并没有这行数据）

```sql
mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_innodb_lock set b='Heygo' where a>1 and a<6;
Query OK, 3 rows affected (0.00 sec)
Rows matched: 3  Changed: 3  Warnings: 0
```

- session2 开启事务，修改 test_innodb_lock 中不同行的数据，也会导致阻塞，直至 session1 提交事务

```sql
mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_innodb_lock set b='9001' where a=9;
# 在这儿阻塞着呢~~~
```

### 4.3.4 手动行锁

**如何锁定一行:**

- `select xxx ... for update` 锁定某一行后，其它的操作会被阻塞，直到锁定行的会话提交
- session1 开启事务，手动执行 for update 锁定指定行，待执行完指定操作时再将数据提交

```sql
mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from test_innodb_lock  where a=8 for update;
+------+------+
| a    | b    |
+------+------+
|    8 | 8000 |
+------+------+
1 row in set (0.00 sec)
```

- session2 开启事务，修改 session1 中被锁定的行，会导致阻塞，直至 session1 提交事务

```sql
mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_innodb_lock set b='XXX' where a=8;
# 在这儿阻塞着呢~~~
```

### 4.3.5 行锁分析

**案例结论:**

1. Innodb存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会要更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表级锁定的。
2. **当系统并发量较高的时候，Innodb的整体性能和MyISAM相比就会有比较明显的优势了**。
3. 但是，Innodb的行级锁定同样也有其脆弱的一面，当我们使用不当的时候（索引失效，导致行锁变表锁），可能会让Innodb的整体性能表现不仅不能比MyISAM高，甚至可能会更差。

**行锁分析**:

**如何分析行锁定**

- 通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况

```sql
show status like 'innodb_row_lock%';

mysql> show status like 'innodb_row_lock%';
+-------------------------------+--------+
| Variable_name                 | Value  |
+-------------------------------+--------+
| Innodb_row_lock_current_waits | 0      |
| Innodb_row_lock_time          | 212969 |
| Innodb_row_lock_time_avg      | 42593  |
| Innodb_row_lock_time_max      | 51034  |
| Innodb_row_lock_waits         | 5      |
+-------------------------------+--------+
5 rows in set (0.00 sec)
```

**对各个状态量的说明如下：**

1. Innodb_row_lock_current_waits：当前正在等待锁定的数量；
2. Innodb_row_lock_time：从系统启动到现在锁定总时间长度；
3. Innodb_row_lock_time_avg：每次等待所花平均时间；
4. Innodb_row_lock_time_max：从系统启动到现在等待最常的一次所花的时间；
5. Innodb_row_lock_waits：系统启动后到现在总共等待的次数；

------

**对于这5个状态变量，比较重要的主要是**

1. Innodb_row_lock_time_avg（等待平均时长）
2. Innodb_row_lock_waits（等待总次数）
3. Innodb_row_lock_time（等待总时长）

------

尤其是当等待次数很高，而且每次等待时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根据分析结果着手指定优化计划。

### 4.3.6 行锁优化

**优化建议:**

1. 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁
2. 合理设计索引，尽量缩小锁的范围
3. 尽可能较少检索条件，避免间隙锁
4. 尽量控制事务大小，减少锁定资源量和时间长度
5. 尽可能低级别事务隔离

## 4.4 页锁

1. 开销和加锁时间界于表锁和行锁之间：会出现死锁；
2. 锁定粒度界于表锁和行锁之间，并发度一般。
3. 了解即可



# 五.主从复制

> https://www.jianshu.com/p/faf0127f1cb2