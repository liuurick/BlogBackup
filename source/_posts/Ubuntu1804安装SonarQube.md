---
title: Ubuntu1804安装SonarQube
date: 2021-04-18 22:28:19
tags:
---

[TOC]

<!--more-->

### 密码信息

操作系统：root / root；sonar / sonar；

数据库：root / root；



### Ubuntu更换阿里源

```shell
# 备份原来的源
sudo cp /etc/apt/sources.list /etc/apt/sources_init.list

# 更换源
sudo vi /etc/apt/sources.list

# 将阿里源复制进去
deb http://mirrors.aliyun.com/ubuntu/ xenial main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main

deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main

deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates universe

deb http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security universe

# 更新源
sudo apt-get update

# 更新软件
sudo apt-get upgrade

```





### 安装OpenJDK

```shell
# 更新软件包列表
sudo apt-get udpate

# 安装openjdk-8-jdk
sudo apt-get install openjdk-8-jdk

# 如果安装失败，可以使用--fix-missing修复
sudo apt-get install openjdk-8-jdk --fix-missing

# 查询是否安装成功
java -version
```

[Ubuntu 18.04安装Java JDK8三种方式](<https://blog.csdn.net/zbj18314469395/article/details/86064849>)



### 首次登陆Ubuntu设置root密码

```shell
# 修改密码
sudo passwd
# 输入两次新密码，OK
```



### 安装mysql5.7

```shell
sudo apt-get udpate
# mysql-server
sudo apt-get install mysql-server
# mysql-client
sudo apt install mysql-client
# dev
sudo apt install libmysqlclient-dev
# 查看情况
sudo netstat -tap | grep mysql
```





### 修改mysql root密码

```shell
# 切换root用户
su
# 登陆mysql
mysql
# 打开数据库名字为mysql的数据库
use mysql
# 修改mysql的密码
update user set authentication_string=PASSWORD("root")where user='root';
# 输入
update user set plugin="mysql_native_password";
# 刷新权限
flush privileges;
# 退出mysql命令行
quit;
# 重新打开Ubuntu18.04终端，正常使用其他用户登录mysql
mysql -uroot -p
```

[Ubuntu18.04下安装mysql5.7超详细步骤](<https://blog.csdn.net/weijianyi/article/details/101540270>)



### 创建sonar数据库

```shell
mysql -uroot -p

CREATE DATABASE sonar DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'sonar' IDENTIFIED BY 'sonar';
GRANT ALL ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'sonar'; 
GRANT ALL ON sonar.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar';
FLUSH PRIVILEGES;
```

[在Ubuntu 18.04上搭建SonarQube服务](<https://blog.csdn.net/hekaiyou/article/details/82996279>)



### 上传SonarQube软件包到服务器

```shell
scp sonarqube-6.7.7.zip sonar@192.168.56.130:/home/sonar
```

### 解压zip包

```shell
# 没有unzip，需要安装
sudo apt install unzip
# 解压
unzip sonarqube-6.7.7.zip
```



### 配置环境变量

```shell
# 使用vim编辑
sudo vim /etc/profile
# 添加内容
SONAR_HOME="/home/sonar/sonarqube-6.7.7/"
# 重启环境变量
. /etc/profile
```

[在Ubuntu 18.04上搭建SonarQube服务](<https://blog.csdn.net/hekaiyou/article/details/82996279>)



### 配置数据库

修改/conf/sonar.properties文件内容（都是取消注释，稍微修改就可以了）

```shell
sonar.jdbc.username=root
sonar.jdbc.password=root
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
sonar.web.host=0.0.0.0
sonar.web.context=/sonar
sonar.web.port=9000
```



### 命令启动SonarQube，并检查SonarQube的启动状态

```shell
cd sonarqube-6.7.7/bin/linux-x86-64
# 启动
./sonar.sh start
# 检查状态
./sonar.sh status
```



### 访问

浏览器输入：http://虚拟机IP:9000/sonar 即可访问SonarQube主页。

默认管理账户是：admin / admin



### 安装插件

插件页面：

网址：<https://docs.sonarqube.org/display/PLUG/Plugin+Library>

下载相关插件jar包，放入``/home/sonar/sonarqube-6.7.7/extensions/plugins/``目录下



### maven配置

在MAVEN_HOME/conf/settings.xml中添加配置

```xml
<!-- sonar相关配置 -->
<pluginGroups>
    <pluginGroup>org.sonarsource.scanner.maven</pluginGroup>
</pluginGroups>

<profiles>
    <profile>
        <id>sonar</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <sonar.host.url>http://10.211.55.12:9000/sonar</sonar.host.url>
        </properties>
    </profile>
</profiles>
```



### 使用mvn sonar:sonar命令执行代码分析

Run/Debug Configurations -> Maven -> sonar:sonar 执行代码分析