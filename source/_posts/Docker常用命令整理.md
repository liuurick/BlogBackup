---
title: Docker常用命令整理
date: 2019-10-01 17:34:37
tags: Docker
categories: Docker
---

## 

<!--more-->

## 常用命令

```text
docker pull ${CONTAINER NAME}                    #拉取镜像
docker images                                    #查看本地所有镜像
docker ps                                        #查看所有正在运行的容器，加-q返回id
docker ps -a                                     #查看所有容器，加-q返回id
docker rmi ${IMAGE NAME/ID}                      #删除镜像
docker rm ${CONTAINER NAME/ID}                   #删除容器
docker save ${IMAGE NAME} > ${FILE NAME}.tar     #将镜像保存成文件
docker load < ${FILE NAME}.tar                   #从文件加载镜像
docker start ${CONTAINER NAME/ID}                #运行一个以前运行过的容器
docker stop ${CONTAINER NAME/ID}                 #停止一个正在运行的容器
docker logs ${CONTAINER NAME/ID}                 #显示运行容器的日志
docker run...                                    #运行一个容器
    --name ${container name}                          #设置容器名称
    -p ${host port}:${container port}                 #映射主机和容器内的端口
    -e ${env name}=${env value}                       #添加环境变量
    -d                                                #后台运行
    -v ${host folder path}:${container folder path}   #将主机目录挂在到容器内
```

## 高级命令

```text
# Advance use 
docker ps -f "status=exited"                                   #显示所有退出的容器
docker ps -a -q                                                #显示所有容器id
docker ps -f "status=exited" -q                                #显示所有退出容器的id
docker restart $(docker ps -q)                                 #重启所有正在运行的容器
docker stop $(docker ps -a -q)                                 #停止所有容器
docker rm $(docker ps -a -q)                                   #删除所有容器
docker rm $(docker ps -f "status=exited" -q)                   #删除所有退出的容器
docker rm $(docker stop $(docker ps -a -q))                    #停止并删除所有容器
docker start $(docker ps -a -q)                                #启动所有容器
docker rmi $(docker images -a -q)                              #删除所有镜像
docker exec -it ${CONTAINER NAME/ID} /bin/bash                 #进入容器内
docker exec -it ${CONTAINER NAME/ID} ping ${CONTAINER NAME/ID} #一个容器ping另外一个容器
docker top ${CONTAINER NAME/ID}                                #显示一个容器的top信息
docker stats                                                   #显示容器统计信息(正在运行)
    docker stats -a                                            #显示所有容器的统计信息(包括没有运行的)
    docker stats -a --no-stream                                #显示所有容器的统计信息(包括没有运行的) ，只显示一次
    docker stats --no-stream | sort -k8 -h                     #统计容器信息并以使用流量作为倒序
docker system 
      docker system df           #显示硬盘占用
      docker system events       #显示容器的实时事件
      docker system info         #显示系统信息
      docker system prune        #清理文件
```



![image-20201101173643142](/images/2020110101.png)