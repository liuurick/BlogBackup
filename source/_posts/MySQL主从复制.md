---
title: MySQL主从复制
date: 2020-11-30 20:51:32
tags: MySQL主从复制
categories: MySQL
---

### MySQL Replication

主从复制（也称 AB 复制）允许将来自一个MySQL数据库服务器（主服务器）的数据复制到一个或多个MySQL数据库服务器（从服务器）。

> 复制是异步的 从站不需要永久连接以接收来自主站的更新。

根据配置，您可以复制数据库中的所有数据库，所选数据库甚至选定的表。

MySQL中复制的优点包括：

- 横向扩展解决方案 - 在多个从站之间分配负载以提高性能。在此环境中，所有写入和更新都必须在主服务器上进行。但是，读取可以在一个或多个从设备上进行。该模型可以提高写入性能（因为主设备专用于更新），同时显着提高了越来越多的从设备的读取速度。
- 数据安全性 - 因为数据被复制到从站，并且从站可以暂停复制过程，所以可以在从站上运行备份服务而不会破坏相应的主数据。
- 分析 - 可以在主服务器上创建实时数据，而信息分析可以在从服务器上进行，而不会影响主服务器的性能。
- 远程数据分发 - 您可以使用复制为远程站点创建数据的本地副本，而无需永久访问主服务器。

#### Replication 的原理

![img](https:////upload-images.jianshu.io/upload_images/11414906-1e1d8aaa7a86af96.png?imageMogr2/auto-orient/strip|imageView2/2/w/799/format/webp)

image.png

> 前提是作为主服务器角色的数据库服务器必须开启二进制日志

1. 主服务器上面的任何修改都会通过自己的 I/O tread(I/O 线程)保存在二进制日志 `Binary log` 里面。
2. 从服务器上面也启动一个 I/O thread，通过配置好的用户名和密码, 连接到主服务器上面请求读取二进制日志，然后把读取到的二进制日志写到本地的一个`Realy log`（中继日志）里面。
3. 从服务器上面同时开启一个 SQL thread 定时检查 `Realy log`(这个文件也是二进制的)，如果发现有更新立即把更新的内容在本机的数据库上面执行一遍。

每个从服务器都会收到主服务器二进制日志的全部内容的副本。

从服务器设备负责决定应该执行二进制日志中的哪些语句。

除非另行指定，否则主从二进制日志中的所有事件都在从站上执行。

如果需要，您可以将从服务器配置为仅处理一些特定数据库或表的事件。

**重要: 您无法将主服务器配置为仅记录特定事件。**

每个从站(从服务器)都会记录二进制日志坐标：

- 文件名
- 文件中它已经从主站读取和处理的位置。

由于每个从服务器都分别记录了自己当前处理二进制日志中的位置，因此可以断开从服务器的连接，重新连接然后恢复继续处理。

##### 一主多从

如果一主多从的话，这时主库既要负责写又要负责为几个从库提供二进制日志。此时可以稍做调整，将二进制日志只给某一从，这一从再开启二进制日志并将自己的二进制日志再发给其它从。或者是干脆这个从不记录只负责将二进制日志转发给其它从，这样架构起来性能可能要好得多，而且数据之间的延时应该也稍微要好一些。工作原理图如下：

![img](https:////upload-images.jianshu.io/upload_images/11414906-5e787d256338774f.png?imageMogr2/auto-orient/strip|imageView2/2/w/946/format/webp)

image.png

#### 关于二进制日志

[**mysqld**](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fmysqld.html)将数字扩展名附加到二进制日志基本名称以生成二进制日志文件名。每次服务器创建新日志文件时，该数字都会增加，从而创建一系列有序的文件。每次启动或刷新日志时，服务器都会在系列中创建一个新文件。服务器还会在当前日志大小达到[`max_binlog_size`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Freplication-options-binary-log.html%23sysvar_max_binlog_size)参数设置的大小后自动创建新的二进制日志文件 。二进制日志文件可能会比[`max_binlog_size`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Freplication-options-binary-log.html%23sysvar_max_binlog_size)使用大型事务时更大， 因为事务是以一个部分写入文件，而不是在文件之间分割。

为了跟踪已使用的二进制日志文件， [**mysqld**](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fmysqld.html)还创建了一个二进制日志索引文件，其中包含所有使用的二进制日志文件的名称。默认情况下，它具有与二进制日志文件相同的基本名称，并带有扩展名`'.index'`。在[**mysqld**](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fmysqld.html)运行时，您不应手动编辑此文件。

术语`二进制日志文件`通常表示包含数据库事件的单个编号文件。

术语 `二进制日志`  表示含编号的二进制日志文件集加上索引文件。

`SUPER` 权限的用户可以使用`SET sql_log_bin=0`语句禁用其当前环境下自己的语句的二进制日志记录

#### 配置 Replication

##### 配置步骤：

1. 在主服务器上，您必须启用二进制日志记录并配置唯一的服务器ID。需要重启服务器。

编辑主服务器的配置文件 `my.cnf`，添加如下内容



```bash
[mysqld]
log-bin=/var/log/mysql/mysql-bin
server-id=1
```

创建日志目录并赋予权限



```bash
shell> mkdir /var/log/mysql
shell> chown mysql.mysql /var/log/mysql
```

重启服务



```bash
shell> systemctl restart mysqld
```

**注意：**

如果省略server-id（或将其显式设置为默认值0），则主服务器拒绝来自从服务器的任何连接。

为了在使用带事务的InnoDB进行复制设置时尽可能提高持久性和一致性，
 您应该在master my.cnf文件中使用以下配置项：



```undefined
innodb_flush_log_at_trx_commit = 1
sync_binlog = 1
```

确保在主服务器上 `skip_networking` 选项处于 `OFF` 关闭状态, 这是默认值。
 如果是启用的，则从站无法与主站通信，并且复制失败。



```ruby
mysql> show variables like '%skip_networking%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| skip_networking | OFF   |
+-----------------+-------+
1 row in set (0.00 sec)
```

1. 应该创建一个专门用于复制数据的用户

每个从服务器需要使用MySQL 主服务器上的用户名和密码连接到主站。

例如，计划使用用户 `repl`  可以从任何主机上连接到 `master` 上进行复制操作, 并且用户 `repl` 仅可以使用复制的权限。

在 `主服务器` 上执行如下操作



```mysql
mysql> CREATE USER 'repl'@'%' 
mysql> GRANT REPLICATION SLAVE ON *.*  TO  'repl'@'%'  identified by 
 'QFedu123!';
mysql> 
```

1. 在`从服务器`上使用刚才的用户进行测试连接



```bash
shell> mysql -urepl -p'QFedu123!' -hmysql-master1
```

下面的操作根据如下情况继续

##### 主服务器中有数据

- 如果在启动复制之前有现有数据需要与从属设备同步，请保持客户端正常运行，以便锁定保持不变。这可以防止进行任何进一步的更改，以便复制到从站的数据与主站同步。

1. 在主服务器中导出现有的数据

如果主数据库包含现有数据，则必须将此数据复制到每个从站。有多种方法可以实现:

- 使用[**mysqldump**](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fmysqldump.html)工具创建要复制的所有数据库的转储。这是推荐的方法，尤其是在使用时 [`InnoDB`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Finnodb-storage-engine.html)。



```mysql
shell> mysqldump  -u用户名  -p密码    --all-databases  --master-data=1 > dbdump.db
```

> 这里的用户是主服务器的用户

如果不使用 `--master-data` 参数，则需要手动锁定单独会话中的所有表。

1. 从主服务器中使用 `scp` 或 `rsync` 等工具，把备份出来的数据传输到从服务器中。

在主服务中执行如下命令



```tsx
scp  dbdump.db root@mysql-slave1:/root/
```

> 这里的 `mysql-slave1` 需要能被主服务器解析出 IP 地址，或者说可以在主服务器中 `ping` 通。

1. 配置从服务器，并重启
    在`从服务器` 上编辑其配置文件 `my.cnf` 并添加如下内容：



```csharp
// my.cnf 文件
[mysqld]
server-id=2
```

1. 导入数据到从服务器，并配置连接到主服务器的相关信息

登录到从服务器上，执行如下操作



```mysql
/*导入数据*/
mysql> source   /root/fulldb.dump
```

在从服务器配置连接到主服务器的相关信息



```bash
mysql> CHANGE MASTER TO
MASTER_HOST='mysql-master1',  -- 主服务器的主机名(也可以是 IP) 
MASTER_USER='repl',                  -- 连接到主服务器的用户
MASTER_PASSWORD='123';        == 到主服务器的密码
```

1. 启动从服务器的复制线程



```mysql
mysql> start slave;
Query OK, 0 rows affected (0.09 sec)
```

检查是否成功

在从服务上执行如下操作，加长从服务器端 IO线程和 SQL 线程是否是 `OK`



```dart
mysql> show slave status\G
```

输出结果中应该看到 I/O 线程和 SQL 线程都是  `YES`, 就表示成功。

执行此过程后，在主服务上操作的修改数据的操作都会在从服务器中执行一遍，这样就保证了数据的一致性。

![img](https:////upload-images.jianshu.io/upload_images/11414906-49fe7c6b2b4f864d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png



![img](https:////upload-images.jianshu.io/upload_images/11414906-6bd28584326146e1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1036/format/webp)

image.png

##### 将新的服务器加入，变为从服务器

和上面的步骤一样，但是新加入的服务器的`server-id` 的值不能和现有都服务器 `server-id` 的值一样。

> 假如在新加入从服务器之前，主服务器执行了删除库的操作。
>  并且，删除的库刚好是在第一次 `mysqldump` 备份时的数据中。
>  就会出现问题，在从服务器上提示报错没有这个数据库；

##### 主服务器中无数据

**主服务器中设置**

1. `my.cnf`配置文件



```bash
[mysqld]
log-bin=/var/log/mysql/mysql-bin
server-id=1
```

> 设置 `log-bin` 时必须同时设置 `server-id`

创建日志目录并赋予权限



```bash
shell> mkdir /var/log/mysql
shell> chown mysql.mysql /var/log/mysql
```

重启服务

**从服务器设置**

1. `my.cnf`配置文件



```bash
[mysqld]
server-id=3
```

重启服务

1. 查看主服务器的二进制日志的名称

通过使用命令行客户端连接到主服务器来启动主服务器上的会话，并通过执行以下[`FLUSH TABLES WITH READ LOCK`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fflush.html%23flush-tables-with-read-lock)语句来刷新所有表和阻止写语句：



```mysql
mysql> FLUSH TABLES WITH READ LOCK;
```



```mysql
mysql> show master status \G
****************** 1. row ****************
             File: mysql-bin.000001
         Position: 0
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set:
1 row in set (0.00 sec)
```

1. 在从服务器的 mysql 中执行如下语句



```mysql
mysql> CHANGE MASTER TO
MASTER_HOST='mysql-master1',
MASTER_USER='repl',
MASTER_PASSWORD='123',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=0;

mysql> start slave;
```

##### 查看

在master上执行show binlog events命令，可以看到第一个binlog文件的内容。



```mysql
mysql> show binlog events\G
*************************** 1. row ***************************
   Log_name: mysql-bin.000001
        Pos: 4
 Event_type: Format_desc
  Server_id: 1
End_log_pos: 107
       Info: Server ver: 5.5.28-0ubuntu0.12.10.2-log, Binlog ver: 4
*************************** 2. row ***************************
   Log_name: mysql-bin.000001
        Pos: 107
 Event_type: Query
  Server_id: 1
End_log_pos: 181
       Info: create user rep
*************************** 3. row ***************************
   Log_name: mysql-bin.000001
        Pos: 181
 Event_type: Query
  Server_id: 1
End_log_pos: 316
       Info: grant replication slave on *.* to rep identified by '123456'
3 rows in set (0.00 sec)
```

- Log_name 是二进制日志文件的名称，一个事件不能横跨两个文件
- Pos 这是该事件在文件中的开始位置
- Event_type 事件的类型，事件类型是给slave传递信息的基本方法，每个新的binlog都以Format_desc类型开始，以Rotate类型结束
- Server_id 创建该事件的服务器id
- End_log_pos 该事件的结束位置，也是下一个事件的开始位置，因此事件范围为Pos~End_log_pos  -  1
- Info 事件信息的可读文本，不同的事件有不同的信息

#### 在从站上暂停复制

您可以使用[`STOP SLAVE`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fstop-slave.html)和 [`START SLAVE`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fstart-slave.html)语句停止并启动从站上的复制 。

要停止从主服务器处理二进制日志，请使用 [`STOP SLAVE`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fstop-slave.html)：



```undefined
mysql> STOP SLAVE;
```

当复制停止时，从I / O线程停止从主二进制日志读取事件并将它们写入中继日志，并且SQL线程停止从中继日志读取事件并执行它们。您可以通过指定线程类型单独暂停I / O或SQL线程：



```undefined
mysql> STOP SLAVE IO_THREAD;
mysql> STOP SLAVE SQL_THREAD;
```

要再次开始执行，请使用以下[`START SLAVE`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fstart-slave.html)语句：



```undefined
mysql> START SLAVE;
```

要启动特定线程，请指定线程类型：



```undefined
mysql> START SLAVE IO_THREAD;
mysql> START SLAVE SQL_THREAD;
```

#### 复制原理实现细节

MySQL复制功能使用三个线程实现，一个在主服务器上，两个在从服务器上：

- **Binlog转储线程** 主设备创建一个线程，以便在从设备连接时将二进制日志内容发送到从设备。可以[`SHOW PROCESSLIST`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fshow-processlist.html)在主服务器的输出中将此线程标识为`Binlog Dump`线程。

  二进制日志转储线程获取主机二进制日志上的锁，用于读取要发送到从机的每个事件。一旦读取了事件，即使在事件发送到从站之前，锁也会被释放。

- **从属 I/O线程** 在从属服务器上发出 `START SLAVE` 语句时，从属服务器会创建一个 I/O 线程，该线程连接到主服务器并要求主服务器发送其在二进制日志中的更新记录。

  从属 I/O线程读取主`Binlog Dump`线程发送的更新 （请参阅上一项）并将它们复制到包含从属中继日志的本地文件。

  此线程的状态显示为 `Slave_IO_running`输出 [`SHOW SLAVE STATUS`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fshow-slave-status.html)或 [`Slave_running`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fserver-status-variables.html%23statvar_Slave_running)输出中的状态[`SHOW STATUS`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fshow-status.html)。

- **从属SQL线程** 从属设备创建一个SQL线程来读取由从属 I/O 线程写入的中继日志，并执行其中包含的事件。

- 当从属服务器从放的事件，追干上主服务器的事件后，从属服务器的 I/O 线程将会处于休眠状态，直到主服务器的事件有更新时，被主服务器发送的信号唤醒。

在前面的描述中，每个主/从连接有三个线程。具有多个从站的主站为每个当前连接的从站创建一个二进制日志转储线程，每个从站都有自己的I / O和SQL线程。

从站使用两个线程将读取更新与主站分开并将它们执行到独立任务中。因此，如果语句执行缓慢，则不会减慢读取语句的任务。例如，如果从服务器尚未运行一段时间，则当从服务器启动时，其I / O线程可以快速从主服务器获取所有二进制日志内容，即使SQL线程远远落后。如果从服务器在SQL线程执行了所有获取的语句之前停止，则I / O线程至少已获取所有内容，以便语句的安全副本本地存储在从属的中继日志中，准备在下次执行时执行奴隶开始。

该[`SHOW PROCESSLIST`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fshow-processlist.html)语句提供的信息可以告诉您主服务器和从服务器上有关复制的信息。有关主状态的信息，请参见[第8.14.4节“复制主线程状态”](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fmaster-thread-states.html)。有关从站状态，请参见[第8.14.5节“复制从站I / O线程状态”](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fslave-io-thread-states.html)和 [第8.14.6节“复制从站SQL线程状态”](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fslave-sql-thread-states.html)。

以下示例说明了三个线程如何显示在输出中[`SHOW PROCESSLIST`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fshow-processlist.html)。

在主服务器上，输出[`SHOW PROCESSLIST`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fshow-processlist.html)如下所示：



```cpp
mysql> SHOW PROCESSLIST\G
*************************** 1\. row ***************************
     Id: 2
   User: root
   Host: localhost:32931
     db: NULL
Command: Binlog Dump
   Time: 94
  State: Has sent all binlog to slave; waiting for binlog to
         be updated
   Info: NULL
```

这里，线程2是`Binlog Dump`为连接的从属服务的复制线程。该 `State`信息表明所有未完成的更新已发送到从站，并且主站正在等待更多更新发生。如果`Binlog Dump`在主服务器上看不到任何 线程，则表示复制未运行; 也就是说，目前没有连接任何从站。

在从属服务器上，输出[`SHOW PROCESSLIST`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fshow-processlist.html)如下所示：



```cpp
mysql> SHOW PROCESSLIST\G
*************************** 1\. row ***************************
     Id: 10
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 11
  State: Waiting for master to send event
   Info: NULL
*************************** 2\. row ***************************
     Id: 11
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 11
  State: Has read all relay log; waiting for the slave I/O
         thread to update it
   Info: NULL
```

该`State`信息指示线程10是与主服务器通信的I / O线程，并且线程11是处理存储在中继日志中的更新的SQL线程。在 [`SHOW PROCESSLIST`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fshow-processlist.html)运行时，两个线程都处于空闲状态，等待进一步更新。

`Time`列中 的值可以显示从站与主站进行比较的时间。请参见 [第A.13节“MySQL 5.7 FAQ：复制”](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Ffaqs-replication.html)。如果主站侧有足够的时间在`Binlog Dump`线程上没有活动，则主站确定从站不再连接。对于任何其他客户端连接，这样做的超时取决于的值 `net_write_timeout`和 `net_retry_count`; 有关这些的更多信息，请参见[第5.1.7节“服务器系统变量”](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fserver-system-variables.html)。

该[`SHOW SLAVE STATUS`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fshow-slave-status.html)语句提供有关从属服务器上的复制处理的其他信息。请参见 [第16.1.7.1节“检查复制状态”](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Freplication-administration-status.html)。

#### 关于复制的格式

[阅读这篇官方文档](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Freplication-formats.html)

#### 基于事务的 Replication

就是利用 GTID 来实现的复制

GTID（全局事务标示符）最初由google实现，在MySQL 5.6中引入.GTID在事务提交时生成，由UUID和事务ID组成.uuid会在第一次启动MySQL时生成，保存在数据目录下的auto .CNF文件里，事务ID则从1开始自增使用GTID的好处主要有两点：

1. 不再需要指定传统复制中的master_log_files和master_log_pos，使主从复制更简单可靠
2. 可以实现基于库的多线程复制，减小主从复制的延迟

#### 实验环境要求： 5.7.6 以上版本

##### 主库配置



```bash
[mysqld]
log-bin=/var/log/mysql/mysql-bin
server-id=1
gtid_mode=ON
enforce_gtid_consistency=1   # 强制执行GTID一致性。
```

重启服务

其他和之前的一样

- 创建专属用户并授权
- 假如有数据导出数据



```mysql
mysql> CREATE USER 'repl'@'%' IDENTIFIED BY '123';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
mysql> 
```

##### 从库配置

测试用户有效性



```bash
shell> mysql -urepl -p'123' -hmysql-master1
```



```bash
[mysqld]
server-id=2
gtid_mode=ON
enforce_gtid_consistency=1

# 可选项, 把连接到 master 的信息存到数据库中的表中
master-info-repository=TABLE
relay-log-info-repository=TABLE
```

重启服务

假如有数据，先导入数据



```mysql
mysql> source dump.db
```

Mysql 终端执行连接信息



```mysql
mysql> CHANGE MASTER TO
MASTER_HOST='172.16.153.10',
MASTER_USER='repl',
MASTER_PASSWORD='123',
MASTER_AUTO_POSITION=1;

> start slave;
```

检查 slave 状态



```dart
mysql> show slave status\G
```

设置 从服务器只读状态

> 查看当前只读的状态
>  SHOW VARIABLES LIKE '%read_only%';
>
> 设置普通用户只读
>  SET GLOBAL read_only=1;
>
> 设置超级用户只读
>  SET GLOBAL super_read_only=1;



```ruby
mysql> SHOW VARIABLES LIKE '%read_only%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_read_only      | OFF   |
| read_only             | OFF   |
| super_read_only       | OFF   |
| transaction_read_only | OFF   |
| tx_read_only          | OFF   |
+-----------------------+-------+
5 rows in set (0.01 sec)

mysql> SET GLOBAL read_only=1;
Query OK, 0 rows affected (0.00 sec)

mysql> ;
ERROR:
No query specified

mysql> SHOW VARIABLES LIKE '%read_only%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_read_only      | OFF   |
| read_only             | ON    |
| super_read_only       | OFF   |
| transaction_read_only | OFF   |
| tx_read_only          | OFF   |
+-----------------------+-------+
5 rows in set (0.00 sec)

mysql> SET GLOBAL super_read_only=1;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE '%read_only%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_read_only      | OFF   |
| read_only             | ON    |
| super_read_only       | ON    |
| transaction_read_only | OFF   |
| tx_read_only          | OFF   |
+-----------------------+-------+
5 rows in set (0.00 sec)

mysql> create database db1;
ERROR 1290 (HY000): The MySQL server is running with the --super-read-only option so it cannot execute this statement
```

#### 开启 GTID 后的导出导入数据的注意点

> Warning: A partial dump from a server that has GTIDs will by default include the GTIDs of all transactions, even those that changed suppressed parts of the database. If you don't want to restore GTIDs, pass --set-gtid-purged=OFF. To make a complete dump, pass --all-databases --triggers --routines --events

意思是： 当前数据库实例中开启了 GTID 功能, 在开启有 GTID 功能的数据库实例中, 导出其中任何一个库, 如果没有显示地指定--set-gtid-purged参数, 都会提示这一行信息. 意思是默认情况下, 导出的库中含有 GTID 信息, 如果不想导出包含有 GTID 信息的数据库, 需要显示地添加--set-gtid-purged=OFF参数.



```bash
mysqldump -uroot  -p  --set-gtid-purged=OFF   --all-databases > alldb.db
```

导入数据是就可以相往常一样导入了。

### 配置多线程复制

多线程复制在 `5.6` 中被引入，并且在 `5.7` 中得到了进一步的完善。

`5.7` 中是基于逻辑时钟的方式进行的多线程复制。

#### 配置过程：

1. 先在从库上查看默认的多线程复制类型



```ruby
mysql> show variables like "slave_parallel_type";
+---------------------+----------+
| Variable_name       | Value    |
+---------------------+----------+
| slave_parallel_type | DATABASE |
+---------------------+----------+
1 row in set (0.01 sec)

mysql>
```

1. 接着在从库上停止目前正在运行复制链路

停止之前可以查看目前的线程数



```dart
show processlist;
```



```undefined
mysql> stop slave
```

1. 配置并发线程的方式



```csharp
mysql> set global slave_parallel_type = "logical_clock";
Query OK, 0 rows affected (0.00 sec)
```

1. 配置并发数量



```dart
mysql> set global slave_parallel_workers = 4;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like "slave_parallel_workers";
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| slave_parallel_workers | 4     |
+------------------------+-------+
1 row in set (0.00 sec)
```

1. 启动从服务器的复制链路



```undefined
mysql> start slave
```

#### 关于复制的架构（扩展）

1. #### 主主复制

![img](https:////upload-images.jianshu.io/upload_images/11414906-6f6288bc65418ae4.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/212/format/webp)

1460000008942626.jpeg



上图中，Master-Master复制的两台服务器，既是master，又是另一台服务器的slave。这样，任何一方所做的变更，都会通过复制应用到另外一方的数据库中。在这种复制架构中，各自上运行的不是同一db，比如左边的是db1,右边的是db2，db1的从在右边反之db2的从在左边，两者互为主从，再辅助一些监控的服务还可以实现一定程度上的高可以用。

1. #### 主动—被动模式的Master-Master(Master-Master in Active-Passive Mode)

![img](https:////upload-images.jianshu.io/upload_images/11414906-b87e84eba995adf5.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/213/format/webp)

1460000008942627.jpeg

上图中，这是由master-master结构变化而来的，它避免了M-M的缺点，实际上，这是一种具有容错和高可用性的系统。它的不同点在于其中只有一个节点在提供读写服务，另外一个节点时刻准备着，当主节点一旦故障马上接替服务。比如通过corosync+pacemaker+drbd+MySQL就可以提供这样一组高可用服务，主备模式下再跟着slave服务器，也可以实现读写分离。

1. #### 带从服务器的Master-Master结构(Master-Master with Slaves)

![img](https:////upload-images.jianshu.io/upload_images/11414906-752e1f66f13fde41.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/291/format/webp)

1460000008942628.jpeg



这种结构的优点就是提供了冗余。在地理上分布的复制结构，它不存在单一节点故障问题，而且还可以将读密集型的请求放到slave上。

##### 半同步机制（扩展）

MySQL-5.5 及以上支持半同步复制
 早前的MySQL复制只能是基于异步来实现，从MySQL-5.5开始，支持半自动复制。在以前的异步（asynchronous）复制中，主库在执行完一些事务后，是不会管备库的进度的。如果备库处于落后，而更不幸的是主库此时又出现Crash（例如宕机），这时备库中的数据就是不完整的。简而言之，在主库发生故障的时候，我们无法使用备库来继续提供数据一致的服务了。Semisynchronous Replication(半同步复制)则一定程度上保证提交的事务已经传给了至少一个备库。Semi synchronous中，仅仅保证事务的已经传递到备库上，但是并不确保已经在备库上执行完成了。

此外，还有一种情况会导致主备数据不一致。在某个session中，主库上提交一个事务后，会等待事务传递给至少一个备库，如果在这个等待过程中主库Crash，那么也可能备库和主库不一致，这是很致命的。如果主备网络故障或者备库挂了，主库在事务提交后等待10秒（rpl_semi_sync_master_timeout的默认值）后，就会继续。这时，主库就会变回原来的异步状态。

MySQL在加载并开启Semi-sync插件后，每一个事务需等待备库接收日志后才返回给客户端。如果做的是小事务，两台主机的延迟又较小，则Semi-sync可以实现在性能很小损失的情况下的零数据丢失。

![img](https:////upload-images.jianshu.io/upload_images/11414906-93fea93cc7067e62.png?imageMogr2/auto-orient/strip|imageView2/2/w/1137/format/webp)

千锋云计算

##### 开启



```ruby
mysql> install plugin rpl_semi_sync_master soname 'semisync_master.so';
Query OK, 0 rows affected (0.08 sec)

mysql> show global variables like '%semi%';                            
+-------------------------------------------+------------+
| Variable_name                             | Value      |
+-------------------------------------------+------------+
| rpl_semi_sync_master_enabled              | OFF        |
| rpl_semi_sync_master_timeout              | 10000      |
| rpl_semi_sync_master_trace_level          | 32         |
| rpl_semi_sync_master_wait_for_slave_count | 1          |
| rpl_semi_sync_master_wait_no_slave        | ON         |
| rpl_semi_sync_master_wait_point           | AFTER_SYNC |
+-------------------------------------------+------------+
6 rows in set (0.00 sec)

mysql>
```

#### 关于主从复制的更多参数

官网： [https://dev.mysql.com/doc/refman/5.7/en/change-master-to.html](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fchange-master-to.html)

##### 加密复制

官网：[https://dev.mysql.com/doc/refman/5.7/en/replication-solutions-encrypted-connections.html](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Freplication-solutions-encrypted-connections.html)

主服务器

创建 CA 证书和私钥 公钥



```sh
shell> mysql_ssl_rsa_setup
```

My.cnf 文件配置项

以下的情况是用 `yum` 安装 mysql 的情况



```sh
[mysqld]
ssl-ca=/var/lib/mysql/ca.pem
ssl-cert=/var/lib/mysql/server-cert.pem
ssl-key=/var/lib/mysql/server-key.pem
```

选项如下：

- [`--ssl-ca`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fencrypted-connection-options.html%23option_general_ssl-ca)：证书颁发机构（CA）证书文件的路径名。（`--ssl-capath`类似但指定CA证书文件目录的路径名。）
  - [`--ssl-cert`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fencrypted-connection-options.html%23option_general_ssl-cert)：服务器公钥证书文件的路径名。可以将其发送到客户端，并根据其拥有的CA证书进行身份验证。
  - [`--ssl-key`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Fencrypted-connection-options.html%23option_general_ssl-key)：服务器私钥文件的路径名。

从服务器配置

首先要保证从服务器的 sql 线程和 io 线程处于关闭状态



```mysql
mysql> stop slave;
mysql> stop slave sql_thread;
```



```mysql
mysql> CHANGE MASTER TO
    -> MASTER_HOST='master_hostname',
    -> MASTER_USER='repl',
    -> MASTER_PASSWORD='password',
    -> MASTER_SSL=1,
    -> MASTER_SSL_CA = 'ca_file_name',
    -> MASTER_SSL_CAPATH = 'ca_directory_name',
    -> MASTER_SSL_CERT = 'cert_file_name',
    -> MASTER_SSL_KEY = 'key_file_name';
mysql> START SLAVE;
```

**关于复制用户**

全新创建



```mysql
mysql> CREATE USER 'repl'@'%.example.com' IDENTIFIED BY 'password'
    -> REQUIRE SSL;
mysql> GRANT REPLICATION SLAVE ON *.*
    -> TO 'repl'@'%.example.com';
```

给原来的用户添加 `REQUIRE SSL`



```mysql
mysql> ALTER USER 'repl'@'%.example.com' REQUIRE SSL;
```

#### 二进制的日志的自动删除

**Mysql 终端中设置**

不用重启服务

下面的命令是只保留 10 天内的日志，就是10天前的全部删除



```mysql
mysql> set global expire_logs_days = 10;
```

当二进制日志的大小达到[`max_binlog_size`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Freplication-options-binary-log.html%23sysvar_max_binlog_size)系统变量的值时，将刷新二进制日志 。

[`max_binlog_size`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Freplication-options-binary-log.html%23sysvar_max_binlog_size)

| 属性           | 值                    |
| -------------- | --------------------- |
| **命令行格式** | `--max-binlog-size=#` |
| **系统变量**   | `max_binlog_size`     |
| **范围**       | 全局                  |
| **动态**       | 是                    |
| **类型**       | 整数                  |
| **默认值**     | `1073741824`          |
| **最低价值**   | `4096`                |
| **最大价值**   | `1073741824`          |

如果对二进制日志的写入导致当前日志文件大小超过此变量的值，则服务器将轮转二进制日志（关闭当前文件并打开下一个文件）。最小值为4096字节。最大值和默认值为1GB。

事务在一个块中写入二进制日志，因此它永远不会在几个二进制日志之间拆分。因此，如果您有大事务，您可能会看到大于的二进制日志文件[`max_binlog_size`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Freplication-options-binary-log.html%23sysvar_max_binlog_size)。

如果[`max_relay_log_size`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Freplication-options-slave.html%23sysvar_max_relay_log_size)为0，则该值也 [`max_binlog_size`](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Freplication-options-binary-log.html%23sysvar_max_binlog_size)适用于中继日志。

**配置文件中设置**

此方法需要重启服务



```mysql
[mysqld]
expire_logs_days=10
```

**在 MySQL 终端中手动删除**



```mysql
--清除MySQL-bin.010日志
mysql> PURGE MASTER LOGS TO 'MySQL-bin.010';

--清除2008-06-22 13:00:00前binlog日志
mysql> PURGE MASTER LOGS BEFORE '2008-06-22 13:00:00';   

--清除3天前binlog日志BEFORE，变量的date自变量可以为'YYYY-MM-DD hh:mm:ss'格式。
mysql> PURGE MASTER LOGS BEFORE DATE_SUB( NOW(), INTERVAL 3 DAY);  
```