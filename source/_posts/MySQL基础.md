---
title: MySQL基础
date: 2018-06-18 09:33:11
tags: MySQL
categories: [编程基础,MySQL]
---

mysql基础

<!--more-->

> https://www.cnblogs.com/lms520/p/5427685.html

# 1.课前准备！

开启mysql服务：1).配置环境变量;2).`net start mysql`

将该sql文件导入到你的数据库中，以下所有操作都是基于该数据库表操作的！！！

建表：

```sql
create table goods (
  goods_id mediumint(8) unsigned primary key auto_increment,
  goods_name varchar(120) not null default '',
  cat_id smallint(5) unsigned not null default '0',
  brand_id smallint(5) unsigned not null default '0',
  goods_sn char(15) not null default '',
  goods_number smallint(5) unsigned not null default '0',
  shop_price decimal(10,2) unsigned not null default '0.00',
  market_price decimal(10,2) unsigned not null default '0.00',
  click_count int(10) unsigned not null default '0'
) engine=myisam default charset=utf8;

insert into `goods` values (1,'kd876',4,8,'ecs000000',1,1388.00,1665.60,9),
(4,'诺基亚n85原装充电器',8,1,'ecs000004',17,58.00,69.60,0),
(3,'诺基亚原装5800耳机',8,1,'ecs000002',24,68.00,81.60,3),
(5,'索爱原装m2卡读卡器',11,7,'ecs000005',8,20.00,24.00,3),
(6,'胜创kingmax内存卡',11,0,'ecs000006',15,42.00,50.40,0),
(7,'诺基亚n85原装立体声耳机hs-82',8,1,'ecs000007',20,100.00,120.00,0),
(8,'飞利浦9@9v',3,4,'ecs000008',1,399.00,478.79,10),
(9,'诺基亚e66',3,1,'ecs000009',4,2298.00,2757.60,20),
(10,'索爱c702c',3,7,'ecs000010',7,1328.00,1593.60,11),
(11,'索爱c702c',3,7,'ecs000011',1,1300.00,0.00,0),
(12,'摩托罗拉a810',3,2,'ecs000012',8,983.00,1179.60,13),
(13,'诺基亚5320 xpressmusic',3,1,'ecs000013',8,1311.00,1573.20,13),
(14,'诺基亚5800xm',4,1,'ecs000014',1,2625.00,3150.00,6),
(15,'摩托罗拉a810',3,2,'ecs000015',3,788.00,945.60,8),
(16,'恒基伟业g101',2,11,'ecs000016',0,823.33,988.00,3),
(17,'夏新n7',3,5,'ecs000017',1,2300.00,2760.00,2),
(18,'夏新t5',4,5,'ecs000018',1,2878.00,3453.60,0),
(19,'三星sgh-f258',3,6,'ecs000019',12,858.00,1029.60,7),
(20,'三星bc01',3,6,'ecs000020',12,280.00,336.00,14),
(21,'金立 a30',3,10,'ecs000021',40,2000.00,2400.00,4),
(22,'多普达touch hd',3,3,'ecs000022',1,5999.00,7198.80,16),
(23,'诺基亚n96',5,1,'ecs000023',8,3700.00,4440.00,17),
(24,'p806',3,9,'ecs000024',100,2000.00,2400.00,35),
(25,'小灵通/固话50元充值卡',13,0,'ecs000025',2,48.00,57.59,0),
(26,'小灵通/固话20元充值卡',13,0,'ecs000026',2,19.00,22.80,0),
(27,'联通100元充值卡',15,0,'ecs000027',2,95.00,100.00,0),
(28,'联通50元充值卡',15,0,'ecs000028',0,45.00,50.00,0),
(29,'移动100元充值卡',14,0,'ecs000029',0,90.00,0.00,0),
(30,'移动20元充值卡',14,0,'ecs000030',9,18.00,21.00,1),
(31,'摩托罗拉e8 ',3,2,'ecs000031',1,1337.00,1604.39,5),
(32,'诺基亚n85',3,1,'ecs000032',4,3010.00,3612.00,9);

create table category (
cat_id smallint unsigned auto_increment primary key,
cat_name varchar(90) not null default '',
parent_id smallint unsigned
)engine myisam charset utf8;

INSERT INTO `category` VALUES
(1,'手机类型',0),
(2,'CDMA手机',1),
(3,'GSM手机',1),
(4,'3G手机',1),
(5,'双模手机',1),
(6,'手机配件',0),
(7,'充电器',6),
(8,'耳机',6),
(9,'电池',6),
(11,'读卡器和内存卡',6),
(12,'充值卡',0),
(13,'小灵通/固话充值卡',12),
(14,'移动手机充值卡',12),
(15,'联通手机充值卡',12);

CREATE TABLE `result` (
  `name` varchar(20) DEFAULT NULL,
  `subject` varchar(20) DEFAULT NULL,
  `score` tinyint(4) DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

insert into result
values
('张三','数学',90),
('张三','语文',50),
('张三','地理',40),
('李四','语文',55),
('李四','政治',45),
('王五','政治',30);

create table a (
id char(1),
num int
)engine myisam charset utf8;

insert into a values ('a',5),('b',10),('c',15),('d',10);
create table b (
id char(1),
num int
)engine myisam charset utf8;

insert into b values ('b',5),('c',15),('d',20),('e',99);

create table m(
     mid int,
     hid int,
     gid int,
     mres varchar(10),
     matime date
)engine myisam charset utf8;

create table t (
     tid int,
     tname varchar(20)
)engine myisam charset utf8;

insert into m
     values
     (1,1,2,'2:0','2006-05-21'),
     (2,2,3,'1:2','2006-06-21'),
     (3,3,1,'2:5','2006-06-25'),
     (4,2,1,'3:2','2006-07-21');

insert into t
     values
     (1,'国安'),
     (2,'申花'),
     (3,'布尔联队');

create table mian ( num int) engine myisam;

insert into mian values
(3),
(12),
(15),
(25),
(23),
(29),
(34),
(37),
(32);

create table user (
uid int primary key auto_increment,
name varchar(20) not null default '',
age smallint unsigned not null default 0
) engine myisam charset utf8;

create table boy (
    hid char(1),
     bname varchar(20)
)engine myisam charset utf8;

insert into boy (bname,hid)
     values
     ('屌丝','A'),
     ('杨过','B'),
     ('陈冠希','C');

create table girl (
    hid char(1),
     gname varchar(20)
     )engine myisam charset utf8;

insert into girl(gname,hid)
     values
     ('小龙女','B'),
     ('张柏芝','C'),
     ('死宅女','D');
```



**1.数据库查询的重要思想:将查询的结果集当成一张新关系二维"表"**

**2.数据表==>二维多列的一个表结构**

**注意**：此处只是告诉你一些在校期间MySQL需要掌握的知识！但是如何用PHP来操作这些知识，需要不断练习



## 1.1 数据库--客户端 

**mysqld --服务器端==>安装mysql之后，内存中就有这个这个服务了!**

mysql -- 客户端 ==>连通服务所使用的软件 `mysql -h localhost -uroot -p` ==>客户端有很多【例如:网络服务(服务器端)==浏览器(IE/Firefox)】

表==(多个表)===>数据库===(多个数据库)==>数据库服务 



## 1.2 phpmyadmin允许空密码登录配置

==>config.sample.inc.php-->复制(config.inc.php)==>$cfg['Servers'][$i]['AllowNoPassword'] = true;



## 1.3 什么是SQL(Structured Query Language) 结构查询语句

-----SQL语句是一种what型语言【想要什么，给你】，而非how语言【要我怎么做才能给你】--php....编程语言

## 1.4 SQL语言分类:

1)DML is Data Manipulation Languages statements .Some examples:数据库操作语言，SQL中处理数据【使用者角度--接触率占据80%】--相当于"员工" 

2)DDL is Data Definition Language statements.Some example:数据定义语言，用于定义管理SQL数据库中所有对象--建表，建库，建视图....等【建设的角度--接触率15%】--相当于"总经理" 

3)DCL is Data Control Language statements.Some example:数据控制语言，用于授予或回收访问数据库的某种权限，并控制数据库操作事务发生的时间及效果，对数据库实行监视...等【管理者角度--接触率5%】---相当于"董事长"



## 1.5 常用的表操作语句:

1. `mysql -h localhost -uroot -p123456` -- 以root用户连接本地数据库

2. `show databases;` -- 查看MySQL服务中所有的数据库
3. use database; -- 更改操作的数据库对象
4. c --取消执行当前未输入mysql语句
5. show tables; -- 查看该操作数据库对象中所有的数据表名和视图名
6. desc table_name/view_name;--查看表/视图结构；
7. truncate table_name; --清空表数据【表结构依然不变】-- 和delete from table_name;是不同的
8. show create table table_name/view; --查看建表/视图过程
9. show table status [\G]; -- 查看数据库中所有表信息【\G:以竖行显示信息】
10. show table status where name = table_name [\G]; -- 查看数据库中指定表信息【\G:以竖行显示信息】
11. rename table_name; --改表名
12. drop table table_name; --删除表
13. drop view view_name; -- 删除视图



# 2 SQL语言之DML部分@数据库操作语言【搬运数据】--"员工"

## 2.1 常用操作:增[insert] 删[delete] 改[update] 查[select]

### 1.INSERT:

`insert into table_name (col1, col2,....) values (value1, value2,....)---`**"插入值"与"列"要一一对应**

### 2.DELETE

`delete from 表名 where条件【不加条件删除整个表】--对于关系型数据库:”增"和"删"都是相对整个一行数据来说的`

### 3.UPDATE

`update 表名 set 列1=新值1,列2=新值2...where条件---修改指定列(修改所有就不用加where)`

### 4.★★★SELECT★★★

`select (列1,列2,列3,....) from 表名 where 条件 limit 0,100;[时间函数：select uid,userid,username,email,FROM_UNIXTIME(addtime,'%Y年%m月%d') from members]`

**【更新和删除操作要注意:where条件记得要加，除非对生活心灰意冷了否则还是加上比较好--不加影响的将是整个表的数据】**

------

`select的5种子句:`

2. `where子句；--条件查询`
3. `groupby子句；--分组查询`
4. `having 子句；--筛选查询`
5. `order by子句；--排序查询`
6. `limit 子句；--范围查询`

**5种子句写的时候要有严格的顺序：where | group by | having | order by | limit**

------

## 2.2 SELECT条件查询模型深入理解【重点】

**====列是"变量"=====变量就可以计算=====**

**select uid, name, age+1 from user;--从user表中查找所有uid, name,age三列，并给age列所在值+1**

**==where是"表达式"==值为真【true】假【false】==**

**select \* from user where id=5;--从user表中查找所有列，当id为5**

**【判断所在行id=5?==>返回true则输出】**

**select \* from user where 1;--从user表中查找所有列，当条件恒真--【输出所有】
select \* from user where 0;--从user表中查找所有列，当条件恒假--【返回Empty】**

**select 语句还可以配合算数运算符、逻辑运算符和位运算符以及相关函数写出更高效率的查询语句**

【当然要注意运算符的优先级】

查询的实质：**对磁盘上的数据文件进行查询得到结果集，并将结果集存放到内存中，其余就是对内存结果集的操作**

### 2.2.1 查询练习:

查询出第4和第11列的信息:
select goods_id, goods_name, shop_price from goods where goods_id =4 or goods_id=11;
select goods_id, goods_name, shop_price from goods where goods_id in(4,11);
查询出第4到第11列间的信息:
select goods_id, goods_name, shop_price from goods where goods_id>4 and goods_id < 11;
select goods_id, goods_name, shop_price from goods where goods_id between 4 and 11;

#### 1.模糊查询(like)--%通配任意字符; _ 通配单一字符

**取出名字以"诺基亚"开头的商品
select goods_id,cat_id,goods_name,shop_price from e cs_goods where goods_name like '诺基亚%';**

**取出名字为"诺基亚Nxx"的手机
select goods_id,cat_id,goods_name,shop_price from ecs_goods where goods_name like '诺基亚N__';**

**取出名字不以"诺基亚"开头的商品**

**select goods_id,cat_id,goods_name,shop_price from ecs_goos where goods_name not like '诺基亚%';**

**当涉及到多重条件查询需要用到运算符,and , or ,not,...之类的来修饰条件时候：
1)一定要先弄清楚条件之间的分类
2)使用( ) 将其分类--避免因为优先级问题**

------

**一道关于查询的面试题**

```
有如下表和数组
把num值处于[20,29]之间,改为20:update main set num=20 where num between 20 and 29;
num值处于[30,39]之间的,改为30:mian表
+------+
| num |
+------+
| 3 |
| 12 |
| 15 |
| 25 |
| 23 |
| 29 |
| 34 |
| 37 |
| 32 |
| 45 |
| 48 |
| 52 |
+------+
floor(X):返回一个不大于X的最大整数值
update mian set num=( floor(num/10)*10 ) where num between 20 and 39;

练习题:
把good表中商品名为'诺基亚xxxx'的商品,改为'HTCxxxx',
提示:大胆的把列看成变量,参与运算,甚至调用函数来处理 .
substring(),--concat()

select goods_id, concat( 'LMS', substring(goods_name,4) ,shop_price 
from goods 
where goods_name like '诺基亚%';
```



------



#### 2.奇怪的NULL查询

**对于NULL=NULL==>返回假;==>NULL是什么都没有,所以不能比较！使用is null 才能查询
select \* from user where name is not null --查询出user表中name字段不为空的信息
【对于数据表中，null不利于数据表优化操作，所以数据表中一般都对字段设置not null】**

------



#### 3.GOUP BY分组与统计函数

**group by -- 当出现group by分组中不能配对的情况，该字段取查询时候第一次出现的值
统计函数：**

```
max()--最大值；
min()--取最小值；
avg()--求平均值;
sum()--求和；
count()--计算行数/条数；
distinct()--求有多少种不同解；
```

**【时间是以时间戳的形式存放的，是int型，max() --最新商品; min() -- 最旧商品】**

------

#### 4.having筛选结果集

**1.查询goods表中商品比市场价低出多少？**
**select goods_id, goods_name,(market_price-shop_price) from goods**
**2.查询goods表中商品比市场价低出至少200的商品？**
**select goods_id, goods_name,(market_price-shop_price) from goods where (market_price - shop_price) > 200;**
**error：查询goods表中商品比市场价低出至少200的商品？**
**select goods_id, goods_name,(market_price-shop_price) as 'min' from goods where min > 200;**
**报错:不识别min这个列！**
**【where子句针对的对象是磁盘上的数据表文件去select的，而select出来后的数据是存放在内存中的一个零时"结果集"】**

**--因此：当使用where min >200 ；去筛选结果集的时候是不能识别出min字段的**
**having--针对的对象是内存表结构中的"结果集"**
**3.查询goods表中商品比市场价低出至少200的商品？**
**select goods_id, goods_name,(market_price-shop_price) as '节省' from goods where 1 having '节省' >200;**

**【如果同时写了where和having子句，where子句肯定要写在having子句前面，因为having子句是针对where子句查询出来的结果集来操作的】** 

------

★★★where-having-group综合练习题

```sql
有如下表及数据
+------+---------+-------+
| name | subject | score |
+------+---------+-------+
| 张三 | 数学 | 90 |
| 张三 | 语文 | 50 |
| 张三 | 地理 | 40 |
| 李四 | 语文 | 55 |
| 李四 | 政治 | 45 |
| 王五 | 政治 | 30 |
+------+---------+-------+
要求:查询出2门及2门以上不及格者的平均成绩
## 一种错误做法【错在:对count和比较运算两者结合的理解错误】
mysql> select name,count(score < 60) as k,avg(score) from stu group by name having k>=2;
+------+---+------------+
| name | k | avg(score) |
+------+---+------------+
| 张三 | 3 | 60.0000 |
| 李四 | 2 | 50.0000 |
+------+---+------------+
2 rows in set (0.00 sec)
 
mysql> select name,count(score < 60) as k,avg(score) from stu group by name;
+------+---+------------+
| name | k | avg(score) |
+------+---+------------+
| 张三 | 3 | 60.0000 |
| 李四 | 2 | 50.0000 |
| 王五 | 1 | 30.0000 |
+------+---+------------+
3 rows in set (0.00 sec)
 
mysql> select name,count(score < 60) as k,avg(score) from stu group by name having k>=2;
+------+---+------------+
| name | k | avg(score) |
+------+---+------------+
| 张三 | 3 | 60.0000 |
| 李四 | 2 | 50.0000 |
+------+---+------------+
2 rows in set (0.00 sec)
 
#加上赵六后错误暴露
mysql> insert into stu -> values -> ('赵六','A',100), -> ('赵六','B',99), -> ('赵六','C',98);
Query OK, 3 rows affected (0.05 sec)
Records: 3 Duplicates: 0 Warnings: 0
 
#错误显现
mysql> select name,count(score < 60) as k,avg(score) from stu group by name having k>=2;
+------+---+------------+
| name | k | avg(score) |
+------+---+------------+
| 张三 | 3 | 60.0000 |
| 李四 | 2 | 50.0000 |
| 赵六 | 3 | 99.0000 |
+------+---+------------+
3 rows in set (0.00 sec)
#正确思路,先查看每个人的平均成绩
mysql> select name,avg(score) from stu group by name;
+------+------------+
| name | avg(score) |
+------+------------+
| 张三 | 60.0000 |
| 李四 | 50.0000 |
| 王五 | 30.0000 |
| 赵六 | 99.0000 |
+------+------------+
4 rows in set (0.00 sec)mysql> 

# 1.看每个人挂科情况
mysql> select name,score < 60 from stu;
+------+------------+
| name | score < 60 |
+------+------------+
| 张三 | 0 |
| 张三 | 1 |
| 张三 | 1 |
| 李四 | 1 |
| 李四 | 1 |
| 王五 | 1 |
| 赵六 | 0 |
| 赵六 | 0 |
| 赵六 | 0 |
+------+------------+
9 rows in set (0.00 sec)
mysql> select name,sum(score < 60) from stu group by name; 

#2.计算每个人的挂科科目
+------+-----------------+
| name | sum(score < 60) |
+------+-----------------+
| 张三 | 2 |
| 李四 | 2 |
| 王五 | 1 |
| 赵六 | 0 |
+------+-----------------+
4 rows in set (0.00 sec)
mysql> select name,sum(score < 60),avg(score) as pj from stu group by name;

#3.同时计算每人的平均分
+------+-----------------+---------+
| name | sum(score < 60) | pj |
+------+-----------------+---------+
| 张三 | 2 | 60.0000 |
| 李四 | 2 | 50.0000 |
| 王五 | 1 | 30.0000 |
| 赵六 | 0 | 99.0000 |
+------+-----------------+---------+
4 rows in set (0.00 sec)
mysql> select name,sum(score < 60) as gk ,avg(score) as pj from stu group by name having gk >=2;

#4.利用having筛选挂科2门以上的.
+------+------+---------+
| name | gk | pj |
+------+------+---------+
| 张三 | 2 | 60.0000 |
| 李四 | 2 | 50.0000 |
+------+------+---------+
2 rows in set (0.00 sec)
```

**错误原因【以下几点】:**

COUNT：是对记录进行汇总，即计数

SUM：是对符合条件的数值列字段进行求和

> https://www.cnblogs.com/bravesunforever/p/11722995.html

**① score < 60 ☛ 比较运算 ☛ 返回值:真(1)、假(0)
②count() ☛ 返回的结果是总行数 ☛ count(socre)和count( score < 60 )得到结果是一样的
③对sum()和count()函数理解的不到位☛想要计算至少有2门课挂了的人，使用count()函数，结果是 ✘
④sum()和score< 60 结合的理解:sum( score < 60) >=2 ==>计算出至少挂了2门课的人**

------

#### 5.order by排序查询【在内存中排序】 与 limit范围查询【--经典应用:分页类】

1:按价格由高到低排序
select goods_id,goods_name,shop_price from goods order by shop_price desc;
2:按发布时间由早到晚排序
select goods_id,goods_name,add_time from goods order by add_time;
3:接栏目由低到高排序,栏目内部按价格由高到低排序【有冲突时，顺序决定优先】
select goods_id,cat_id,goods_name,shop_price from goods order by cat_id ,shop_price desc;
4:取出价格最高的前三名商品
select goods_id,goods_name,shop_price from goods order by shop_price desc limit 3;
5:取出点击量前三名到前5名的商品
select goods_id,goods_name,click_count from goods order by click_count desc limit 2,3;

------

#### 6.子句的查询陷阱

**如何：查询goods表中，每个栏目(cat_id) 下最新(goods_id最大)的那件商品？**

**错误示范:【正确答案见下】
思路：1)最新的商品 -- max(good_id)
2)每个栏目--group by cat_id
mysql> select max(goods_id), goods_name, cat_id, shop_price from goods group by cat_id;**

```
+---------------+-----------------------+--------+------------+
| max(goods_id) | goods_name | cat_id | shop_price |
+---------------+-----------------------+--------+------------+
| 16 | 恒基伟业g101 | 2 | 823.33 |
| 32 | 飞利浦9@9v | 3 | 399.00 |

**除了数据goods_id对了其他 ✘**

| 18 | kd876 | 4 | 1388.00 |
| 23 | 诺基亚n96 | 5 | 3700.00 |
| 7 | 诺基亚n85原装充电器 | 8 | 58.00 |
| 6 | 索爱原装m2卡读卡器 | 11 | 20.00 |
| 26 | 小灵通/固话50元充值卡 | 13 | 48.00 |
| 30 | 移动100元充值卡 | 14 | 90.00 |
| 28 | 联通100元充值卡 | 15 | 95.00 |
+---------------+-----------------------+--------+------------+
```

**这里错在：“先查询在排序”==>group by cat_id ,但goods_name, shop_price，我们应该取谁的呢？--解决思路:用到“子查询”/连接查询==>先排序再查询**

------

------

#### 7.子查询 之 where子查询[以内层查询结果作为外层的比较条件]

**1:查找出goods表中最新的那件商品信息?
思考问题：1.如何保证每次更新商品后，取得都是最新的呢？☛ 涉及到了"变量"☛"列"就是变量
2.查询的条件可以是个表达式☛但是表示得到的要是一个“明确”的量才可以查询
3.数据库查询☛"投影式"查询[要那列查那列,查的那列和其他列没关系]
--第3点典型错误: select max(goods_id), goods_name, shop_price from goods;--除了goods_id对,其余都是错的！这是个有语义缺陷的语句
子语句查询:select goods_id,goods_name,shop_price from goods where goods_id =
( select max(goods_id) from goods );
以查询select max( goods_id ) from user;的返回结果【存放在内存中，且无论如何该结果都是一个"定值"】作为对前方查询语句的条件**

**2.如何：查询goods表中，每个栏目(cat_id) 下最新(goods_id最大)的那件商品？**

**思路整理:(从上面的错误范例已可以得到正确思路==>先"排序" 再"查询")
1.排序==>有题目可知,排序的变量应该是cat_id字段,通过排序找到每一个cat_id下中goods_id最大的那个商品ID号
2.查询==>用排序得到的那个最大ID号作为条件表达式的对比条件，查找出商品信息**

**1.先"排序:"mysql> select max(goods_id), cat_id, shop_price from goods group by cat_id;

```
+---------------+--------+------------+
| max(goods_id) | cat_id | shop_price |
+---------------+--------+------------+
| 16 | 2 | 823.33 |
| 32 | 3 | 399.00 |
| 18 | 4 | 1388.00 |
| 23 | 5 | 3700.00 |
| 7 | 8 | 58.00 |
| 6 | 11 | 20.00 |
| 26 | 13 | 48.00 |
| 30 | 14 | 90.00 |
| 28 | 15 | 95.00 |
+---------------+--------+------------+
9 rows in set (0.00 sec)
```


2.再"查询":mysql> select good_id, goods_name, shop_price from goods where goods_id in (select max(goods_id) from goods group by cat_id);

```
+----------+------------------------------+------------+
| goods_id | goods_name | shop_price |
+----------+------------------------------+------------+
| 6 | 胜创kingmax内存卡 | 42.00 |
| 7 | 诺基亚n85原装立体声耳机hs-82 | 100.00 |
| 16 | 恒基伟业g101 | 823.33 |
| 18 | 夏新t5 | 2878.00 |
| 23 | 诺基亚n96 | 3700.00 |
| 26 | 小灵通/固话20元充值卡 | 19.00 |
| 28 | 联通50元充值卡 | 45.00 |
| 30 | 移动20元充值卡 | 18.00 |
| 32 | 诺基亚n85 | 3010.00 |
+----------+------------------------------+------------+
```



**2.查询出编号为19的商品的栏目名称[栏目名称放在category表中](用左连接查询和子查询分别)**

**WHERE型子查询：**
1.先找出外层条件的内层结果--goods表中第19号商品的cat_id：select cat_id from goods where goods_id = 19;
2.查询:select cat_name from category where cat_id = ( select cat_id from goods where goods_id = 19 );

------

**子查询 之 from子查询【将查询出来的结果集当成一个新"表"来操作】**

**2.如何：查询goods表中，每个栏目(cat_id) 下最新(goods_id最大)的那件商品？--使用from子查询**
**同样的思路==>先排序再查询
排序：mysql> select goods_id, goods_name, shop_price from order by cat_id asc, goods_id DESC;
得到一张优先按照cat_id升序,再goods_id降序的"表"-----同一个cat_id的商品，它在"表"里出现的位置是第一个
排序：mysql> select goods_id, goods_name, shop_price from order by cat_id asc, goods_id DESC;
查询：mysql> select goods_id, goods_name,shop_price from
(select goods_id,cat_id, goods_name, shop_price from goods order by cat_id ) as tmp
group by cat_id;**



------

**子查询 之 exists子查询【"存在"】**

**1.用exists型子查询，查出所有商品的栏目下有商品的栏目**
mysql> **select \* from category where exists (select \* from goods where goods.cat_id = category.cat_id);**
查找category这个表，如果select * from goods where goods.cat_id = category.cat_id这个"表"中对应的数据存在则查询

```
+--------+-------------------+-----------+
| cat_id | cat_name | parent_id |
+--------+-------------------+-----------+
| 2 | CDMA手机 | 1 |
| 3 | GSM手机 | 1 |
| 4 | 3G手机 | 1 |
| 5 | 双模手机 | 1 |
| 8 | 耳机 | 6 |
| 11 | 读卡器和内存卡 | 6 |
| 13 | 小灵通/固话充值卡 | 12 |
| 14 | 移动手机充值卡 | 12 |
| 15 | 联通手机充值卡 | 12 |
+--------+-------------------+-----------+
9 rows in set (0.00 sec)
```

------

------

#### 8.内连接查询[inner join]、左连接[left join]、右连接[right join]

**【MySQL中没有外连接】**

> **详解：http://www.dedecms.com/knowledge/data-base/sql-server/2012/0709/2872.html
> 内连接：select xxxx from table1 inner join table2 on table1.xx=table2.xx ☛ 交集
> 左连接：select xxxx from table1 left join table2 on table1.xx=table2.xx ☛ 左表为基础的查询
> 右连接：select xxxx from table1 right join table2 on table1.xx=table2.xx ☛ 右表为基础的查询**

**1.查询价格大于2000元的商品及其栏目名称
思路：
--涉及到两个表；--基础表为goods表,连接表为category表，条件为shop_price > 2000
--goods表cat_id中的和category表中的cat_id对应
mysql > select goods.goods_id, category.cat_name, goods.goods_name, goods.shop price from
\- > goods left join category
\- > on goods.cat_id = category.cat_id
\- > where goods.shop_price > 2000;
2.取出第4个栏目下的商品的商品名,栏目名,与品牌名
select goods_name,cat_name,shop_price from goods left join category on goods.cat_id=category.cat_id where goods.cat_id = 4**

------

**用友面试题**:

**根据给出的表结构按要求写出SQL语句。**

Match 赛程表

| 字段名称    | 字段类型    | 描述                |
| ----------- | ----------- | ------------------- |
| matchID     | int         | 主键                |
| hostTeamID  | int         | 主队的ID            |
| guestTeamID | int         | 客队的ID            |
| matchResult | varchar(20) | 比赛结果，如（2:0） |
| matchTime   | date        | 比赛开始时间        |

Team 参赛队伍表

| 字段名称 | 字段类型    | 描述     |
| -------- | ----------- | -------- |
| teamID   | int         | 主键     |
| teamName | varchar(20) | 队伍名称 |

**Match的hostTeamID与guestTeamID都与Team中的teamID关联**
查出 2006-6-1 到2006-7-1之间举行的所有比赛，并且用以下形式列出：
拜仁 2：0 不来梅 2006-6-21

**mysql> select \* from m;**
+-----+------+------+------+------------+
| mid | hid | gid | mres | matime |
+-----+------+------+------+------------+
| 1 | 1 | 2 | 2:0 | 2006-05-21 |
| 2 | 2 | 3 | 1:2 | 2006-06-21 |
| 3 | 3 | 1 | 2:5 | 2006-06-25 |
| 4 | 2 | 1 | 3:2 | 2006-07-21 |
+-----+------+------+------+------------+
4 rows in set (0.00 sec)

**mysql> select \* from t;**
+------+----------+
| tid | tname |
+------+----------+
| 1 | 国安 |
| 2 | 申花 |
| 3 | 传智联队 |
+------+----------+
3 rows in set (0.00 sec)

**思路：--使用Team表中tname代替Match表中对应的hid和gid,然后对时间用between做出做出范围限制查询**
**1.先代替hid:**
**mysql> select m.\*, t.tname as htname
    -> from m inner join t
    -> on m.hid = t.tid;**

+------+------+------+------+------------+----------+

| mid | hid | gid | mres | matime | htname |

+------+------+------+------+------------+----------+

| 1 | 1 | 2 | 2:0 | 2006-05-21 | 国安 |

| 2 | 2 | 3 | 1:2 | 2006-06-21 | 申花 |

| 3 | 3 | 1 | 2:5 | 2006-06-25 | 布尔联队|

| 4 | 2 | 1 | 3:2 | 2006-07-21 | 申花 |

+------+------+------+------+------------+----------+

**2.再将查询出来的结果集当做是一张新的表对Team表再来一次内连接查询**
**mysql> select m.\*, t.tname as htname,t1.tname as gtname from m inner join t on m.hid = t.tid
    ->inner join t as t1
**    **->on m.gid=t1.tid;**
+------+------+------+------+------------+----------+----------+
| mid | hid | gid | mres | matime | htname | gtname |
+------+------+------+------+------------+----------+----------+
| 1 | 1 | 2 | 2:0 | 2006-05-21 | 国安 | 申花 |
| 2 | 2 | 3 | 1:2 | 2006-06-21 | 申花 | 布尔联队|
| 3 | 3 | 1 | 2:5 | 2006-06-25 | 布尔联队 | 国安 |
| 4 | 2 | 1 | 3:2 | 2006-07-21 | 申花 | 国安 |
+------+------+------+------+------------+----------+----------+
4 rows in set (0.00 sec)

==>**3.替换后结果如下：**
**mysql> select hid,t1.tname as hname ,mres,gid,t2.tname as gname,matime
    -> from
    -> m left join t as t1
    -> on m.hid = t1.tid
    -> left join t as t2
**    **-> on m.gid = t2.tid;**
+------+----------+------+------+----------+------------+
| hid | hname | mres | gid | gname | matime |
+------+----------+------+------+----------+------------+
| 1 | 国安 | 2:0 | 2 | 申花 | 2006-05-21 |
| 2 | 申花 | 1:2 | 3 | 传智联队 | 2006-06-21 |
| 3 | 传智联队 | 2:5 | 1 | 国安 | 2006-06-25 |
| 2 | 申花 | 3:2 | 1 | 国安 | 2006-07-21 |
+------+----------+------+------+----------+------------+
4 rows in set (0.00 sec)

**==>3.最终结果如下：**
**mysql> select hid,t1.tname as hname ,mres,gid,t2.tname as gname,matime
    -> from
    -> m left join t as t1
    -> on m.hid = t1.tid
    -> left join t as t2
    -> on m.gid = t2.tid
    -> where matime between "2006-06-01" and "2006-07-01";**

+------+----------+------+------+----------+------------+
| hid | hname | mres | gid | gname | matime |
+------+----------+------+------+----------+------------+
| 2 | 申花 | 1:2 | 3 | 布尔联队 | 2006-06-21 |
| 3 | 布尔联队 | 2:5 | 1 | 国安 | 2006-06-25 |
+------+----------+------+------+----------+------------+
2 rows in set (0.00 sec)

------

#### 9.union查询:将2条或多条SQL的查询结果合并成1个结果集

**注意：**
1)取的两个表投影查找的字段列数要相同，列名可不一致(默认使用第一个表的列名)否则
2)如果碰到完全相同的行，将会被合并【合并是非常耗时的☛使用 union all 就不需要比较字段合并了】
3)union查询的内部子句中不用写order by子句，意义不大！但是可以对查询合并后id结果集进行排列

**1.同时查询goods表中cat_id为2和4的商品**

**select goods_id,cat_id,goods_name from goods where cat_id =2
union
select goods_id, cat_id,goods_name from goods where cat_id = 4 (order by )**

------

**union查询面试题**

**将A、B表中id值相同的两个num值相加**

**A表:
+------+------+
| id | num |
+------+------+
| a | 5 |
| b | 10 |
| c | 15 |
| d | 10 |
+------+------+ B表:
+------+------+
| id | num |
+------+------+
| b | 5 |
| c | 15 |
| d | 20 |
| e | 99 |
+------+------+mysql> # 合并 ,注意all的作用
mysql> select \* from ta
    -> union all
    -> select \* from tb;
+------+------+
| id | num |
+------+------+
| a | 5 |
| b | 10 |
| c | 15 |
| d | 10 |
| b | 5 |
| c | 15 |
| d | 20 |
| e | 99 |
+------+------+**

**将上面查询的"结果集"当做是一个新表**

**参考答案:**

**mysql> # sum,group求和**

**mysql> select id,sum(num)**

​    **->from**

​    **->(select \* from ta union all select \* from tb) as tmp**

​    **->group by id;**

+------+----------+

| id | sum(num) |

+------+----------+

| a | 5 |

| b | 15 |

| c | 25 |

| d | 30 |

| e | 99 |

+------+----------+

5 rows in set (0.00 sec)

------

------

# 3 SQL语言之DDL部分@数据定义语言【建库、建表】--"总经理"

------

------

## 3.1 创建表table

**1)建"表"过程 ☛ 申明数据库中各个"列"的过程
☛ creat table_name ( 列名 列类型 [列属性 列默认值]) ENGINE = 存储引擎 default charset=字符集;
★★★
2)设计"表"结构☛对"列"的优化☛"列"选什么类型?列选什么属性最好?**

------

## 3.2 列类型知识:

**数值型：整型、浮点型、定点型 字符串：char varchar text,... 日期时间：datetime, time,**

**一种类型，占得字节越多，存储越大，也越浪费**

### 2_1:整型列

**bigint 8个字节
int 4个字节【1个字节=8位☛4个字节=32位--也就是"1"这个int型只占了32位中1个位】
mediumint 3个字节
smallint 2个字节
tinyint 1个字节 【8位==> 0-255 或 -128 - 127】**

**1)像tinyint中，默认数值型都是对半正负分配的==>即:正常情况下tinyint是不能存储大于128的数字的!**

**那么，如何让tinyint存储0-255之间的数呢？**
使用unsigned属性【无符号】修饰；
zerofill==>用0填充至固定宽度【学号:1->0001;255 ->0255】
M -> 宽度 tinyint(5)-->宽度为5;varchar(10)->宽度为10
**注意：**①zerofill属性就已经代表了该类型为是unsigned属性了==>负数不需要用0填充
②M属性只有和zerofill配合使用才有意义！宽度是指0填充的宽度，而不是指该列存储的宽度【如:tinyint(1) 可以存储111】

------

### 2_2.浮点列[float/doule]与定点列[decimal]

**浮点列:float/double (M,D) [UNSIGNED] [ZEROFILL] -- M表示精度【总位数】,D表示小数点后面的位数**
如:float(3,2)--存10==>错误:其实这里有4位了10.00;
float(3,2)==>存9.99正确
**定点列decimal[整数部分和小数部分分开来存储的]
浮点数是有精度损失的！定点列更准确**

------

### 2_3字符型列[char/varchar]

#### ①char(M)--定长；varchar(M) -- 变长

**例如:
char(10) -- 放10个字符长度,但是存放1个字符，在内存中依然是占10个字符长度
--char(M) 在磁盘上就占M个字节，磁盘空间利用率可能达到100%
varchar(10) -- 放10个字符长度，但是存放1个字符，在内存中就占了1个字符长度的空格键
--varchar(M) 在内存表中存储时，在表头会增加1-2说明字节存储该字符串长度==>那么内存寻址的时候就能准确找到每一行数据==>实际varchar占M+[1/2]字节**

**小技巧：一般对于M较小的，都用char!**

**1).因为varchar的利用率是不可能达到100%！**

**2).内存的定长寻址会快很多
****3).char型，如果不够M个宽度，内存存储时候会用空格在字符右边补齐，取出时候把右侧空格删除**
**如果用char存储' hello '，取出之后' hello'；用varchar存，取出时候' hello '**

#### ②text -- 大文本类型；blob -- 二进制类型

**例如：论文、博客...等大段文本text
图像、音频等二进制信息用blob类型来存储
意义：blob是使用二进制来存储信息的，因此不需要考虑字符集的问题！
例如0xFF这个字节，在ASCII字符集中被认为是非法的，在入库的时候就会被过滤掉！如果使用blob来存储则不会被过滤**

####  

#### ③enum('value1','value2',...) -- 枚举类型；set('value1','value2',...) -- 集合类型

**例如：
enum('男','女') ☛ 该列所存储的值就只能是'男'或'女' ☛ 是个单选值存储
set('value1','value2',...) ☛ 是个复选值存储，但值也只能在列举的元素中选取
注意：set()最多只能列举64个值！**

------

### 2_4日期时间型列[char/varchar]

**year 年 [1个字节] 范围:[1901-2155] ☛ 在insert是，可以简写年后面对两位，但是这样不推荐
【00-69】+2000；【70-99】+1900 ☛ 填写两位，表示1970-2069年✘不要只写后面2个数字
Date 日期 1994-10-29
☛ 以'YYYY-MM-DD HH:MM:SS'格式检索和显示DATETIME值。支持的范围为'1000-01-01 00:00:00'到'9999-12-31 23:59:59'
time 时间 13:02:29
☛ 用'YYYY-MM-DD'格式检索和显示DATE值。支持的范围是'1000-01-01'到 '9999-12-31'
datetime 日期
☛ 以'YYYY-MM-DD HH:MM:SS'格式检索和显示DATETIME值。支持的范围为'1000-01-01 00:00:00'到'9999-12-31 23:59:59'
int unsigned 时间戳 1970-01-01 00:00:00 到当前的秒数
☛ 一般存注册时间，商品发布时间等，并不是用datetime，而是用时间戳存储，因为datetime存储虽然直观，但不便计算**

------

### 2_5 列属性 ☛ 默认值[ default ]&& not null

**1.NULL不便于查询【注意：空字数串,0都不是NULL--NULL是什么都没有，是不存在】
not null default xxxx**

### 2_6 列属性 ☛ 主键[ primary key ] && 自增[ auto_increment ]

**1.此列不重复，能够区分每一行==>列名 primary key auto_increment
一般主键和自增是一起使用的[int类型]，不一定一要一起使用！一张表中只能有一个自增的列！
小技巧：
1.很多时候都是用tinyint存储☛性别：0/1 --> 男/女;体重：tinyint 【0-255】....
2.定长存储寻址快，效率高--常用的字段建议定长存储【对于一张表，只有一个变长大字段其他都是定长字段情况下，可考虑将变长单独分出来】
3.一般mysql的列名都用小写**

------

### 2_7 列的增add/删/改 ☛ 这是对表结构的修改

**增：alter table 表名 add 列名 列类型 [列属性] -- 默认该列是存放在表最后的【使用 after 列名 --放在指定列】
删：alter table 表名 drop column 列名 列类型 [列属性]
改：alter table 表名 change 旧列名 新列名 [新列类型] [新列属性]
改：alter table 表名 modify 列名 [新列类型] [新列属性] --modify 不能修改列名**

------

## 4  视图

### 1).什么是视图？

**view 又称虚拟表，view其实就一条查询SQL语句的结果集==>将常用的SQL查询结果集虚拟为一张表存放在内存中
create view as 视图名 (查询SQL语句结果集);--当再次使用时:select \* from 视图名**

### 2).视图有什么用？【视图实际上存储的就是SQL语句】

**①权限的控制!比如：某几个列允许用户查询，而其他列不允许，可以通过视图开放其中的一部分列，达到权限的控制
②简化复杂的查询!比如：查询每个栏目下的商品的平均价格并按平均价格排序，然后查出平均价格前3高的栏目
①create view v as select cat_id, avg(shop_price) as pj from goods group by cat_id
②select \* from v order by pj limit 0,3**

### 3).视图能不能更新删除修改

**①视图【虚拟表】☛ 是物理表的一个"投影",两者是相互影响的☛更改物理表，虚拟表也会更改，同理，更改虚拟表，物理表也会更改！**

**但是：如果虚拟表中含有函数(经过计算...)，则不能修改!【即物理表和虚拟表的列能一一对应，则虚拟表中该列能修改--改一行影响一行】**

**①create view as v select cat_id, avg(shop_price) as pj from goods group by cat_id**

**②update v set pj = 80 where cat_id=11;--报错！因为修改结果不能正确映射回到goods表中所有shop_price中**

**同理：增加和删除操作也是和修改一样**

### 4).视图放在什么地方？

**①对于VIEW存储的SQL语句是简单的select语句，所以当对视图查询时候就是对SQL语句的拼接==>对物理表的间接拼接查询(合并：merge)
②对于VIEW存储的SQL语句已经是逻辑复杂的select语句了，这时对视图的拼接查询会更麻烦！
==>这时候mysql会先执行视图的创建语句，把结果集形成一张临时表，再对临时表(temptable)进行操作
MySQL数据库中可以通过algorithm(算法)定义对视图的处理情况 create algorithm = merge/temptalbe view v_name as ...
[不写该属性，则由MySQL自行判断]**

------

## 5 存储引擎[ENGINE]

### 1).什么是存储引擎？

**即：保存"数据"的形式【格式】
MYISAM:【处理快-相对不安全-不支持事务】
good.frm--说明书[声明表结构的表具体语句]
good.MYD--数据内容
goods.MYI--目录[索引文件]
InnoDB【安全-处理慢-支持事务】--只有.frm文件，其余表的其余全部内容存放在了一个文件中
Memory【存放在内存中--一关机就没有了】**

------

## 5.字符集与乱码问题

### 1.什么是乱码？

对计算机来说，没有"乱码"，只有0/1==>乱码：人看不懂！

### 2.为什么会乱码？

**①导致原因：文字本来的字符集与展示的字符集不一致
==>一般统一utf8；
②服务器和客户端字符集不一致！
客户端[GBK提交数据]==>连接器处理[转换为数据库字符集]==>数据库[UTF8存放数据]【无论连接器转不转，最终存放到数据库中都是UTF8】
数据库[UTF8存放数据]==>连接器处理[转换为客户端字符集]==>客户端[GBK显示数据]

☛由于客户端和数据库字符集不同导致的乱码==>在提交和显示数据的时候，要"说清楚"字符集
==>"我"要什么字符集？==>客户端：set character_set_client =gbk;【谁连接服务器谁就是客户端，客户端字符集是多变的】
==>"你"接受什么字符集？==>数据库：set character_set_results=utf8;
==>"转换"用什么字符集？==>连接器：set character_set_connection = gbk/utf8[都可以]**
**只需要将3者的字符集设置一致不会乱码了！==>set names gbk/utf8 ==> 1句好比3句强
UTF8:包含中文，韩文，日文，英语，繁体字...国际化
GBK:只有简体中文
****gbk2123:简体中文+英文**

### 3.怎么能不乱码？

**①文件保存时是否保存为utf8格式
②HTML页面显示时候 ：
③创建数据表的时候： create table () charset utf8;
****④查询数据的时候：set names utf8;**

### 4.什么是字符校对集？

**[utf8-bin==>bin : 二进制排序]==>校对集就是查询排序的标准**

######  

------

## 6. 索引[ index ]

### 1).查询方式？

**当表中有大量记录时，若要对表进行查询：
①全表搜索，是将所有记录一一取出，和查询条件进行一一对比，然后返回满足条件的记录
☛消耗大量数据库系统时间，并造成大量磁盘I/O操作
****②第二种就是在表中建立索引，然后在索引中找到符合查询条件的索引值，最后通过保存在索引中的ROWID（相当于页码）快速找到表中对应的记录**

### 2).什么是索引？

**索引是对数据库表中一列或多列的值进行排序的一种结构，使用索引可快速访问数据库表中的特定信息！**

**--相当于图书的"目录",根据目录，迅速定位查找内容的位置**

### 3).索引优/缺点？

**优点：**
**①加快了查询时对数据的检索速度
②创建唯一性索引，保证数据库表中每一行数据的唯一性
③加速表和表之间的连接
****④在使用分组和排序子句进行数据检索时，可以显著减少查询中分组和排序的时间**
**缺点：**
①索引是另外独立于数据外存放的一个二进制文件==>需要占物理空间( .MYI )
②对表数据进行增、删和改的维护操作时，索引也要动态的变化==>降低了数据增、删、改的维护速度
☛在创建索引之前，您必须确定要使用哪些列以及要创建的索引类型!
☛索引不是越多越好==>一般在查询频率多、且重复度小的列上加!

**例如：性别和身份号都需要频繁查询,且表数据量大
==>性别：就只有男和女，定位的时候有太多重复的了，添加索引反而是占用了空间！
****==>身份证号：添加索引，身份证号是唯一的,只要快速找到索引就能快速定位**

### 4).索引类型

**①key 列名(索引名)==> 普通索引==>纯粹提高查询速度
②unique key 列名(索引名)==> 唯一索引 ==>提高速度，且约束数据唯一性
③primary key 列名 ==> 主键索引==>唯一主键
****④fulltext ==> 全文索引 ==> 在中文环境下，基本不起作用,要分词索引，一般用第三方解决方案(如：sphinx)** 

### 5).索引长度：

**[在建立索引时，对列中一部分字符进行索引]
①unique key / key 列名(索引名 (索引长度) )
****例如：对于唯一的Email,形式都是.....@qq.com**

### 6).多列索引：

**[在建立索引时，对2个或多个列进行索引]**

### 7).冗余索引：

**[索引存在覆盖]==>冗余索引有时候在开发中是必要的**

### 8).操作索引：

**①查看索引：show index table_name;
②添加索引：alter table table_name add index column( index_name )
③删除索引:alter table table_name
添加主键索引: alter table table_name add primary key column
删除主键索引: alter table table_name drop primary key;
1.explain select .... ==>查看该语句执行信息==>可以查看使用到的索引
2.索引有一个左前缀查找原则==> ".......xxx"这样对xxx发挥不了作用**

------

## 7. 常用九大类函数==>看一次就好！要用的时候至少知道

**数据库是用来存储管理数据的，能够少用函数来处理尽量少用==>效率慢**

### 1)、数学函数

abs(x) 返回x的绝对值
bin(x) 返回x的二进制（oct返回八进制，hex返回十六进制）
ceiling(x) 返回大于x的最小整数值==>向上取整
exp(x) 返回值e（自然对数的底）的x次方
floor(x) 返回小于x的最大整数值==>向下取整
greatest(x1,x2,...,xn)返回集合中最大的值
least(x1,x2,...,xn) 返回集合中最小的值
ln(x) 返回x的自然对数
log(x,y)返回x的以y为底的对数
mod(x,y) 返回x/y的模（余数）
pi()返回pi的值（圆周率）
rand()返回０或１的随机值,可以通过提供一个参数(种子)使rand()生成器生成1.
round(x,y)返回参数x的四舍五入的有y位小数的值
sign(x) 返回代表数字x的符号的值
sqrt(x) 返回一个数的平方根
truncate(x,y) 返回数字x截短为y位小数的结果

------

### 2)、聚合函数(常用于group by从句的select查询中)

avg(col)返回指定列的平均值
count(col)返回指定列中非null值的个数
min(col)返回指定列的最小值
max(col)返回指定列的最大值
sum(col)返回指定列的所有值之和
group_concat(col) 返回由属于一组的列值连接组合而成的结果

------

### 3)、字符串函数

ascii(char)返回字符的ascii码值
bit_length(str)返回字符串的比特长度
concat(s1,s2...,sn)将s1,s2...,sn连接成字符串
concat_ws(sep,s1,s2...,sn)将s1,s2...,sn连接成字符串，并用sep字符间隔
insert(str,x,y,instr) 将字符串str从第x位置开始，y个字符长的子串替换为字符串instr，返回结果
find_in_set(str,list)分析逗号分隔的list列表，如果发现str，返回str在list中的位置
lcase(str)或lower(str) 返回将字符串str中所有字符改变为小写后的结果
left(str,x)返回字符串str中最左边的x个字符
length(s)返回字符串str中的字符数
ltrim(str) 从字符串str中切掉开头的空格
position(substr,str) 返回子串substr在字符串str中第一次出现的位置
quote(str) 用反斜杠转义str中的单引号
repeat(str,srchstr,rplcstr)返回字符串str重复x次的结果
reverse(str) 返回颠倒字符串str的结果
right(str,x) 返回字符串str中最右边的x个字符
rtrim(str) 返回字符串str尾部的空格
strcmp(s1,s2)比较字符串s1和s2
trim(str)去除字符串首部和尾部的所有空格
ucase(str)或upper(str) 返回将字符串str中所有字符转变为大写后的结果

------

### 4)、日期和时间函数

curdate()或current_date() 返回当前的日期
curtime()或current_time() 返回当前的时间
date_add(date,interval int keyword)返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：selectdate_add(current_date,interval 6 month);
date_format(date,fmt) 依照指定的fmt格式格式化日期date值
date_sub(date,interval int keyword)返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：selectdate_sub(current_date,interval 6 month);
dayofweek(date) 返回date所代表的一星期中的第几天(1~7)
dayofmonth(date) 返回date是一个月的第几天(1~31)
dayofyear(date) 返回date是一年的第几天(1~366)
dayname(date) 返回date的星期名，如：select dayname(current_date);
from_unixtime(ts,fmt) 根据指定的fmt格式，格式化unix时间戳ts
hour(time) 返回time的小时值(0~23)
minute(time) 返回time的分钟值(0~59)
month(date) 返回date的月份值(1~12)
monthname(date) 返回date的月份名，如：select monthname(current_date);
now() 返回当前的日期和时间
quarter(date) 返回date在一年中的季度(1~4)，如select quarter(current_date);
week(date) 返回日期date为一年中第几周(0~53)
year(date) 返回日期date的年份(1000~9999)
一些示例：
获取当前系统时间：select from_unixtime(unix_timestamp());
select extract(year_month from current_date);
select extract(day_second from current_date);
select extract(hour_minute from current_date);
返回两个日期值之间的差值(月数)：select period_diff(200302,199802);
在mysql中计算年龄：
select date_format(from_days(to_days(now())-to_days(birthday)),'%y')+0 as age from employee;
这样，如果brithday是未来的年月日的话，计算结果为0。
下面的sql语句计算员工的绝对年龄，即当birthday是未来的日期时，将得到负值。
select date_format(now(), '%y') - date_format(birthday, '%y') -(date_format(now(), '00-%m-%d') < date_format(birthday, '00-%m-%d')) as age from employee

------

### 5)、加密函数

**aes_encrypt(str,key) 返回用密钥key对字符串str利用高级加密标准算法加密后的结果，调用aes_encrypt的结果是一个二进制字符串，以blob类型存储
aes_decrypt(str,key) 返回用密钥key对字符串str利用高级加密标准算法解密后的结果
decode(str,key) 使用key作为密钥解密加密字符串str
encrypt(str,salt) 使用unixcrypt()函数，用关键词salt(一个可以惟一确定口令的字符串，就像钥匙一样)加密字符串str
encode(str,key) 使用key作为密钥加密字符串str，调用encode()的结果是一个二进制字符串，它以blob类型存储
md5() 计算字符串str的md5校验和
password(str) 返回字符串str的加密版本，这个加密过程是不可逆转的，和unix密码加密过程使用不同的算法。
sha() 计算字符串str的安全散列算法(sha)校验和
示例：
select encrypt('root','salt');
select encode('xufeng','key');
select decode(encode('xufeng','key'),'key');#加解密放在一起
select aes_encrypt('root','key');
select aes_decrypt(aes_encrypt('root','key'),'key');
select md5('123456');
select sha('123456');**

------

### 6)、控制流函数

```
mysql有4个函数是用来进行条件操作的，这些函数可以实现sql的条件逻辑，允许开发者将一些应用程序业务逻辑转换到数据库后台。
mysql控制流函数：
case when[test1] then [result1]...else [default] end如果testn是真，则返回resultn，否则返回default
case [test] when[val1] then [result]...else [default]end  如果test和valn相等，则返回resultn，否则返回default
if(test,t,f)   如果test是真，返回t；否则返回f
ifnull(arg1,arg2) 如果arg1不是空，返回arg1，否则返回arg2
nullif(arg1,arg2) 如果arg1=arg2返回null；否则返回arg1
这些函数的第一个是ifnull()，它有两个参数，并且对第一个参数进行判断。
    ==>如果第一个参数不是null，函数就会向调用者返回第一个参数；如果是null,将返回第二个参数。
如：select ifnull(1,2), ifnull(null,10),ifnull(4*null,'false');
nullif()函数将会检验提供的两个参数是否相等，如果相等，则返回null，如果不相等，就返回第一个参数。
如：select nullif(1,1),nullif('a','b'),nullif(2+3,4+1);
和许多脚本语言提供的if()函数一样，mysql的if()函数也可以建立一个简单的条件测试，这个函数有三个参数:
    ==>第一个是要被判断的表达式，如果表达式为真，if()将会返回第二个参数，如果为假，if()将会返回第三个参数。
如：selectif(1<10,2,3),if(56>100,'true','false');
if()函数在只有两种可能结果时才适合使用。然而，在现实世界中，我们可能发现在条件测试中会需要多个分支。
   ---在这种情况下，mysql提供了case函数，它和php及perl语言的switch-case条件例程一样。
case函数的格式有些复杂，通常如下所示：
case [expression to be evaluated]
when [val 1] then [result 1]
when [val 2] then [result 2]
when [val 3] then [result 3]
......
when [val n] then [result n]
else [default result]
end
    这里，第一个参数是要被判断的值或表达式，接下来的是一系列的when-then块，每一块的第一个参数指定要比较的值，如果为真，就返回结果。
    所有的when-then块将以else块结束，当end结束了所有外部的case块时
    ==>如果前面的每一个块都不匹配就会返回else块指定的默认结果。如果没有指定else块，而且所有的when-then比较都不是真，mysql将会返回null。
case函数还有另外一种句法，有时使用起来非常方便，如下：
case
when [conditional test 1] then [result 1]
when [conditional test 2] then [result 2]
else [default result]
end
这种条件下，返回的结果取决于相应的条件测试是否为真。
示例：
mysql>select case 'green'
     when 'red' then 'stop'
     when 'green' then 'go' end;
select case 9 when 1 then 'a' when 2 then 'b' else 'n/a' end;
select case when (2+2)=4 then 'ok' when(2+2)<>4 then 'not ok' end asstatus;
select name,if((isactive = 1),'已激活','未激活') as result fromuserlogininfo;
select fname,lname,(math+sci+lit) as total,
case when (math+sci+lit) < 50 then 'd'
when (math+sci+lit) between 50 and 150 then 'c'
when (math+sci+lit) between 151 and 250 then 'b'
else 'a' end
as grade from marks;
select if(encrypt('sue','ts')=upass,'allow','deny') as loginresultfrom users where uname = 'sue';#一个登陆验证
```

 

------

### 7)、格式化函数

date_format(date,fmt) 依照字符串fmt格式化日期date值
format(x,y) 把x格式化为以逗号隔开的数字序列，y是结果的小数位数
inet_aton(ip) 返回ip地址的数字表示
inet_ntoa(num) 返回数字所代表的ip地址
time_format(time,fmt) 依照字符串fmt格式化时间time值
其中最简单的是format()函数，它可以把大的数值格式化为以逗号间隔的易读的序列。
示例：
select format(34234.34323432,3);
select date_format(now(),'%w,%d %m %y %r');
select date_format(now(),'%y-%m-%d');
select date_format(19990330,'%y-%m-%d');
select date_format(now(),'%h:%i %p');
select inet_aton('10.122.89.47');
select inet_ntoa(175790383);

------

### 8)、类型转化函数

为了进行数据类型转化，mysql提供了cast()函数，它可以把一个值转化为指定的数据类型。类型有：binary,char,date,time,datetime,signed,unsigned 示例：
select cast(now() as signed integer),curdate()+0;
select 'f'=binary 'f','f'=cast('f' as binary);

------

### 9)、系统信息函数

database() 返回当前数据库名
benchmark(count,expr) 将表达式expr重复运行count次
connection_id() 返回当前客户的连接id
found_rows() 返回最后一个select查询进行检索的总行数
user()或system_user() 返回当前登陆用户名
version() 返回mysql服务器的版本
示例：
select database(),version(),user();
selectbenchmark(9999999,log(rand()*pi()));#该例中,mysql计算log(rand()*pi())表达式9999999次。

------

## 8. 事务的概念

### 1.什么是事务？

将一个业务下的SQL语句作为一个单元统一操作==>"同生共死"!【MyISAM不支持事务】
例如：A"打账"500给B，打完之后A减少500，B增加500！如果这两个动作有一个没完成则整个打账过程取消失败--[**原子性**]

### 2.如何启用事务？

**start transaction;**

### 3.如何结束事务？

**commit;**

### 4.如何撤销事务？【回滚事务】

**rollback;**

 

事务的中间状态是不可见的--隔离性

事务发生结束了之后是不能恢复的--持久性

事务之前和之后它们的业务逻辑上要保持一致！两人总账额度9000，相互转帐后依然是9000 -- 一致性



**对于MySQL索引优化，DCL...等部分，工作之后就会遇到！现在可以不必要学！****现在学了也不一定会，会了也不一定能用得上，用得上也不一定能记得！**