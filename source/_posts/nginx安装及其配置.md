---
title: nginx安装及其配置
date: 2020-07-31 08:18:05
tags: nginx
categories: nginx
---

nginx安装及其配置

<!--more-->

# 1 nginx 介绍

## 1 什么是nginx

Nginx是一款高性能的http 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。

由俄罗斯的程序设计师Igor Sysoev所开发，官方测试nginx能够支支撑5万并发链接，

并且cpu、内存等资源消耗却非常低，运行非常稳定。

## 2 应用场景

1、http服务器。Nginx是一个http服务可以独立提供http服务。可以做网页静态服务器。

2、虚拟主机。可以实现在一台服务器虚拟出多个网站。例如个人网站使用的虚拟主机。

3、反向代理，负载均衡。当网站的访问量达到一定程度后，单台服务器不能满足用户的请求时，

需要用多台服务器集群可以使用nginx做反向代理。并且多台服务器可以平均分担负载，

不会因为某台服务器负载高宕机而某台服务器闲置的情况。

 

# 2 nginx安装

## 1 下载

官方网址：http://nginx.org/en/download.html

官网提供三种版本：

Nginx官网提供了三个类型的版本
Mainline version：Mainline 是 Nginx 目前主力在做的版本，可以说是开发版
Stable version：最新稳定版，生产环境上建议使用的版本
Legacy versions：遗留的老版本的稳定版

![img](/images/2020073101.png)

我们这里下载的是Stable version下面的

![img](/images/2020073102.png)

使用的版本是1.14.0.tar.gz.

## 2 安装要求的环境

下面的环境需要视自己的系统情况而定，没有的环境安装以下就好。

**1.需要安装gcc环境**

```
# yum install gcc-c++
```

**2.第三方的开发包**

**1 PERE**

PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。

nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库。

**注：pcre-devel是使用pcre开发的一个二次开发库。nginx****也需要此库**。

```
# yum install -y pcre pcre-devel
```

**2 zlib**

zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库。

```
# yum install -y zlib zlib-devel
```

**3 openssl**

OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，

并提供丰富的应用程序供测试或其它目的使用。

nginx不仅支持http协议，还支持https（即在ssl协议上传输http），所以需要在linux安装openssl库。

```
# yum -y install pcre  pcre-devel zlib  zlib-devel openssl openssl-devel
```

## 3 nginx安装过程

**1 把nginx源码包上传到linux系统上**

![img](/images/2020073103.png)

**2 解压到/usr/local下面**

```
# tar -xvf nginx-1.14.0.tar.gz -C /usr/local
```

**3 使用cofigure命令创建一个makeFile文件**

**执行下面的命令的时候，一定要进入到nginx-1.14.0目录里面去。**

![img](/images/2020073104.png)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi \--with-http_stub_status_module \--with-http_ssl_module \--with-file-aio \--with-http_realip_module
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果没有makeFile文件，编译的时候会报错

![img](/images/2020073105.png)

\ 表示命令还没有输入完，换行的意思。

```
--prefix=/usr/local/nginx  表示软件安装到/usr/local/nginx下面。
这个make install 的时候就不用在指定安装路径。
执行完成后查看目录里面已经多了一个Makefile文件
```

![img](/images/2020073106.png)

**注意：启动nginx之前，上边将临时文件目录指定为/var/temp/nginx，**

```
需要在/var下创建temp及nginx目
```

**4 创建目录/var/temp/nginx/**

```
# mkdir /var/temp/nginx -p
```

-p 表示级联创建的意思

**5 进入nginx-1.14.0里面执行make命令进行编译**

 ![img](/images/2020073107.png)

**6 进入nginx-1.14.0里面执行make install 命令进行安装**

 这里不需要再次执行安装路径，创建makefile文件的时候已经指定了。

![img](/images/2020073108.png)

**7 进入安装位置/usr/local/nginx查看目录结构**

![img](/images/2020073109.png)

其中html是里面首页html文件。conf里面是配置文件。sbin里面只执行文件。

# 3 启动nginx

进入sbin目录，执行命令./nginx

```
[root@admin sbin]# ./nginx
```

# 4 查看nginx是否启动

```
[root@admin sbin]# ps -aux | grep nginx
```

![img](/images/2020073110.png)

**ps命令**用于报告当前系统的进程状态。

-a：显示所有终端机下执行的程序，除了阶段作业领导者之外。

a：显示现行终端机下的所有程序，包括其他用户的程序。

u：以用户为主的格式来显示程序状况。

x：显示所有程序，不以终端机来区分。

 

# 5 关闭nginx

```
[root@admin sbin]#  ./nginx -s stop
```

或者

```
[root@admin sbin]# ./nginx -s quit
```

 

# 6 重启nginx

先关闭，然后启动

 

# 7 刷新配置文件

```
[root@admin sbin]# ./nginx -s reload
```

 

# 8 关闭防火墙，开启远程访问

首先需要关闭防火墙：默认端口是80

**方法一：永久开放80端口**

```
/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
/etc/rc.d/init.d/iptables save
```

**方法二：临时关闭系统防火墙**

```
# service iptables stop  
```

**方法三：永久关闭修改配置开机不启动防火墙**

```
# chkconfig iptables off 
```

**特殊：针对阿里云**

需要添加安全组规则

![img](/images/2020073111.png)

# 9 访问nginx

![img](/images/2020073112.png)

# 10 配置虚拟主机

就是在一台服务器启动多个网站。

如何区分不同的网站：主要有以下两种方式

方式一：端口不同

方式二：域名不同

# 11 通过端口区分不同的主机

nginx配置文件的位置：/usr/local/nginx/conf/nginx.conf

原始配置文件的内容如下：

![img](/images/2020073113.png)

我们可以通过配置多个server,从而配置多个虚拟机

![img](/images/2020073114.png)

下面测试以下：复制原来的html目录，改名为html-81

![img](/images/2020073115.png)

修改以下里面的index.html文件，方便区分

```
[root@admin nginx]# vim html-81/index.html
```

![img](/images/2020073116.png)

修改完成之后刷新以下配置文件

```
[root@admin sbin]# ./nginx -s reload
```

然后分别访问192.168.204.131:80 和192.168.204.131:81

![img](/images/2020073117.png)

![img](/images/2020073118.png)

# 12 多个域名区分虚拟主机

## 1 什么是域名

域名就是网站：www.baidu.com就是域名

DNS域名解析服务器，把域名解析为ip地址。保存的就是域名和ip地址的映射关系。

一级域名：baidu.com

二级域名：www.baidu.com

三级域名：image.baidu.com

一个域名对应与一个ip地址，一个ip地址可以被多个域名绑定。

只需要买一个一级域名，后面的二级，三级域名你自己可以随便定义。

 

本地测试我们可以通过修改hosts配置文件来完成：

hosts文件的位置：C:\Windows\System32\drivers\etc

可以自己手动配置域名和ip的映射关系，如果hosts文件中配置了域名和ip的对应关系，不需要走DNS域名解析服务器。

因为拿到一个域名，首先是到hosts文件里面查找，没有才有去DNS域名解析器查找。

## 2 nginx配置

![img](/images/2020073119.png)

## 3 测试

1 修改本地hosts配置文件

![img](https://images2018.cnblogs.com/blog/1320077/201805/1320077-20180528233243735-537226043.png)

2 复制html目录，分别改名为html-taobao和html-baidu

![img](https://images2018.cnblogs.com/blog/1320077/201805/1320077-20180528233425639-1929848567.png)

3 分别修改html-baidu和html-taobao里面的index.html文件，方便区分

![img](https://images2018.cnblogs.com/blog/1320077/201805/1320077-20180528233638275-1668361848.png)

![img](https://images2018.cnblogs.com/blog/1320077/201805/1320077-20180528233727028-1216322786.png)

4 刷新配置文件

```
[root@admin sbin]# ./nginx -s reload
```

5 然后使用浏览器分别访问：www.taobao.com 和 www.baidu.com

 

# 13 正向代理

![img](https://images2018.cnblogs.com/blog/1320077/201805/1320077-20180529000121513-942126145.png)

# 14 反向代理

![img](https://images2018.cnblogs.com/blog/1320077/201805/1320077-20180529000622924-995860976.png)

反向代理服务器决定那台服务器提供服务

# 15 nginx实现反向代理

两个域名指向同一台nginx服务器，用户访问不同的域名显示不同的网页内容。

两个域名是www.baidu.com和www.taobao.com

nginx代理服务器使用虚拟机192.168.204.131

![img](https://images2018.cnblogs.com/blog/1320077/201805/1320077-20180529001151869-999884385.png)

第一步：安装两个tomcat，分别运行在8080和8081端口。

第二步：启动两个tomcat。

第三步：反向代理服务器的配置

 ![img](https://images2018.cnblogs.com/blog/1320077/201805/1320077-20180529001600606-1806158722.png)

第四步：nginx重新加载配置文件

第五步：配置域名

在hosts文件中添加域名和ip的映射关系

192.168.204.131 www.baidu.com

192.168.204.131 www.taobao.com

 

# 16 负载均衡

如果一个服务由多个服务器提供，需要把负载分配到不同的服务器处理，需要负载均衡。

![img](https://images2018.cnblogs.com/blog/1320077/201805/1320077-20180529002643025-1569805563.png)

可以根据服务器的实际情况调整服务器权重。权重越高分配的请求越多，权重越低，请求越少。默认是都是1

![img](https://images2018.cnblogs.com/blog/1320077/201805/1320077-20180529002741304-1542056839.png)

# 17 设置nginx开机自启动（centos6.5）

每次启动nginx服务都需要到安装目录下的/sbin下面，感觉挺麻烦的。

下面介绍一下如何在Linux(CentOS)系统上，设置nginx开机自启动。

## 1 用脚本管理nginx服务

**第一步：在/etc/init.d/目录下创建nginx文件，命令如下：**

```
# touch /etc/init.d/nginx
```

**第二步：在创建的nginx文件中加入下面的内容**

首先执行命令：

```
# vim /etc/init.d/nginx
```

然后加下面的内容复制到nginx配置文件中

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
nginx="/usr/sbin/nginx"
prog=$(basename $nginx)
NGINX_CONF_FILE="/etc/nginx/nginx.conf"
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
lockfile=/var/lock/subsys/nginx
make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -z "`grep $user /etc/passwd`" ]; then
       useradd -M -s /bin/nologin $user
   fi
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
force_reload() {
    restart
}
configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}
rh_status() {
    status $prog
}
rh_status_q() {
    rh_status >/dev/null 2>&1
}
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面的脚本文件并不是自己写的，是nginx官方提供的。

地址：http://wiki.nginx.org/RedHatNginxInitScript

注意：如果是自定义安装的nginx,修改根据实际情况修改安装路和配置文件。

```
nginx="/usr/sbin/nginx" 修改成你的nginx执行程序的路径。比如我的是nginx="/usr/local/nginx/sbin/nginx"
NGINX_CONF_FILE="/etc/nginx/nginx.conf" 修改成你的配置文件的路径
例如：NGINX_CONF_FILE="/usr/local/nginx/nginx.conf
```

修改完成后保存脚本文件，wq 保存并退出

**第三步：设置nginx文件的权限**

```
# chmod a+x /etc/init.d/nginx
```

解释：a+x==>all user can execute 所有用户可执行）的意思

**第四步：管理脚本**

到这里，我们就可以使用nginx脚本对服务进行管理了

```
# /etc/init.d/nginx start      启动服务
# /etc/init.d/nginx stop       停止服务  # /etc/init.d/nginx restart    重启服务
# /etc/init.d/nginx status     查看服务的状态# /etc/init.d/nginx reload     刷新配置文件
```

## 2 使用chkconfig管理

上面的方法完成了用脚本管理nginx服务的功能，但是还是不太方便，比如要设置nginx开机启动等。

这个时候我们可以使用chkconfig来进行管理。

**第一步：将nginx服务加入chkconfig管理列表**

```
# chkconfig --add /etc/init.d/nginx
```

**第二步：使用service管理服务**

```
# service nginx start    启动服务
# service nginx stop     停止服务# service nginx restart  重启服务# service nginx status   查询服务的状态# service nginx relaod   刷新配置文
```

**第三步：设置终端模式开机启动**

```
# chkconfig nginx on
```

 

# 17 设置nginx开机自启动（centos7.4）

 **第一步：进入到/lib/systemd/system/目录**

```
[root@iz2z init.d]# cd /lib/systemd/system/
```

**第二步：创建nginx.service文件，并编辑**

```
# vim nginx.service
```

内如如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[Unit]
Description=nginx service
After=network.target 
   
[Service] 
Type=forking 
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true 
   
[Install] 
WantedBy=multi-user.target
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[Unit]: 服务的说明

Description:描述服务
After:描述服务类别
[Service]服务运行参数的设置
Type=forking是后台运行的形式
ExecStart为服务的具体运行命令
ExecReload为重启命令
ExecStop为停止命令
PrivateTmp=True表示给服务分配独立的临时空间
注意：[Service]的启动、重启、停止命令全部要求使用绝对路径
[Install]运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3

保存退出。

**第三步：加入开机自启动**

```
# systemctl enable nginx
```

如果不想开机自启动了，可以使用下面的命令取消开机自启动

```
# systemctl disable nginx
```

**第四步：服务的启动/停止/刷新配置文件/查看状态**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# systemctl start nginx.service　         启动nginx服务
# systemctl stop nginx.service　          停止服务
# systemctl restart nginx.service　       重新启动服务
# systemctl list-units --type=service     查看所有已启动的服务
# systemctl status nginx.service          查看服务当前状态
# systemctl enable nginx.service          设置开机自启动
# systemctl disable nginx.service         停止开机自启动
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

##  **一个常见的错误**

### Warning: nginx.service changed on disk. Run 'systemctl daemon-reload' to reload units.

 直接按照提示执行命令systemctl daemon-reload 即可。

```
# systemctl daemon-reload
```

 

# 18 重启系统，再次启动nginx报错

## 1 故障现场

之前在虚拟机centos6.5上面设置自启动之后，重新启动系统可以正常启动，也不会出错。

centos6.5的自启动设置见16部分知识点。

但是在centos7.4(阿里云上面），参照第17部分配置好了自启动。重启系统发现nginx并没有自启动

使用命名systemctl status nginx查看了一下状态，内容如下：

![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180603235731837-1149240490.png)

然后我直接进入/usr/local/nginx/sbin目录下面，执行./nginx，出现了下面的错误提示：

![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180603235854971-2001750680.png)

从这两个提示信息，可以大概看出告诉我们的就是找不到/var/run/nginx/目录下面的nginx.pid文件。

## 2 故障解决

**第一步：进入 cd /usr/local/nginx/conf/ 目录，编辑配置文件nginx.conf ；**

在配置文件中找到：#pid    logs/nginx.pid;

![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180604000215412-935975488.png)

将其修改为：去掉注释，修改成自己的路径

![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180604000522605-1193262105.png)

修改完成保存退出

**第二步：创建目录/var/run/nginx/**

```
# mkdir /var/run/nginx -p
```

**第三步：启动nginx服务**

```
# /usr/local/nginx/sbin/nginx
```

可以查看一下是否成功启动了

![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180604001021288-75578496.png)

## 3 故障重现

**[emerg] open() "/var/run/nginx/nginx.pid" failed (2: No such file or directory)处理**

测试发现，只要执行reboot命令重启，var/run/nginx，nginx这个文件夹都会被删除，

搞得每一次都要去建立nginx这个文件夹，简直麻烦到了极点，实在受不了。下面

继续来解决这个问题。

**第一步：进入 cd /usr/local/nginx/conf/ 目录，编辑配置文件nginx.conf ；**

![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180604004515961-636067355.png)

**第二步：在/usr/local/nginx目录下建立logs文件夹**

```
# mkdir /usr/local/nginx/logs
```

**![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180604005704151-2060063432.png)**

**第三步：把/var/run/nginx/目录下的nginx.pid这个文件拷贝到第二步创建的logs文件夹里面。**

```
# cp nginx.pid /usr/local/nginx/logs/
```

 ![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180604005724309-1632561610.png)

**第四步：把logs这个文件夹在conf下也拷贝一份**

```
# cp -r logs conf
```

![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180604005846851-1054721549.png)

**第五步：修改权限/usr/local/nginx/logs/目录下面的nginx.pid文件的权限。**

```
[root@iz2logs]# chmod 755 nginx.pid
```

![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180604010132048-1203053027.png)

**第六步：重启reboot**

```
# reboot
```

**第六步：启动nginx**

```
# /usr/local/nginx/sbin/nginx
```

![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180604010643083-893375042.png)

这次是终于成功解决了，一边安装一边解决问题，到这里nginx总是算是可以自启动了，并且也不会重启后找不到nginx.pid文件。真的太不容易了。

**解决的原理：就是让它去另外一个地方找nginx.pid文件，**

**因为/var/run/nginx/nginx.pid这个文件总是重启就删除了**。

## 简单解决方案

上面的过程有点繁琐了，实际可以直接按照下面的这个简单方法解决

修改nginx.conf文件如下：

![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180604012457087-1758425381.png)

在/usr/local/nginx/目录下创建一个logs目录。

然后启动就可以了，并且重启也不会被删除。

这样下面的日志文件的配置也可以简化为去掉# error_log logs/error.log info; 前面的“#”就可以了

error_log logs/error.log info;

# 19 配置日志文件的位置

**第一步：进入 cd /usr/local/nginx/conf/ 目录，编辑配置文件nginx.conf ；**

![img](https://images2018.cnblogs.com/blog/1320077/201806/1320077-20180604001806779-1643312454.png)

**第二步：保证肯定有这个路径，可以直接创建一下这个配置的目录**

```
# mkdir -p /var/log/nginx/
```

**第三步：刷新配置文件**

```
# /usr/local/nginx/sbin/nginx -s reload
```