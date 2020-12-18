---
title: Centos7上安装docker
date: 2019-12-17 21:27:49
tags: Docker
categories: Docker
---

Docker分为社区版CE和企业版EE。CE版完全足够个人使用，选择stable版本即可。

<!--more-->

##  一、安装docker

1.Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

通过 **uname -r** 命令查看你当前的内核版本

```
 # uname -r
```

![image-20201217213427007](/images/2019121701.png)

2.使用 `root` 权限登录 Centos。确保 yum 包更新到最新。

```
# sudo yum update
```

3.卸载旧版本(如果安装过旧版本的话)

```
# sudo yum remove docker  docker-common docker-selinux docker-engine
```

4.安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```
# sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

5.设置yum源

```
# sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

![image-20201217213649924](/images/2019121702.png)

6.可以查看所有仓库中所有docker版本，并选择特定版本安装

```
# yum list docker-ce --showduplicates | sort -r
```

7.安装docker

```
# sudo yum install docker-ce  #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版
# sudo yum install <FQPN>  # 例如：sudo yum install docker-ce-17.12.0.ce
```

8.启动docker

```
# sudo systemctl start docker
```

9.设置开机自启动

```
# sudo systemctl enable docker
```

10.验证安装是否成功(有client和service两部分表示docker安装启动都成功了)

```
# docker version
```

 

## 二、配置镜像加速

> 阿里云界面：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

Docker客户端版本需要大于 1.10.0 

可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://26udi78a.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```





## 三、问题

1.因为之前已经安装过旧版本的docker，在安装的时候报错如下：

```
Transaction check error:
  file /usr/bin/docker from install of docker-ce-17.12.0.ce-1.el7.centos.x86_64 conflicts with file from package docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64
  file /usr/bin/docker-containerd from install of docker-ce-17.12.0.ce-1.el7.centos.x86_64 conflicts with file from package docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64
  file /usr/bin/docker-containerd-shim from install of docker-ce-17.12.0.ce-1.el7.centos.x86_64 conflicts with file from package docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64
  file /usr/bin/dockerd from install of docker-ce-17.12.0.ce-1.el7.centos.x86_64 conflicts with file from package docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64
```

2.卸载旧版本的包

```
# sudo yum erase docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64
```

3.再次安装docker

```
# sudo yum install docker-ce
```



2.yum安装的时候遇到公钥尚未安装的问题解决

```
在 yum install xxxx 命令之后添加 --nogpgcheck 进行跳过公钥检查安装
```

或者

导入密钥重新安装Docker

```java
rpm --import http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
```