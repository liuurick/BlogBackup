---
title: MinIO对象存储
date: 2021-05-09 10:58:18
tags: MinIO
categories: MinIO
---

MinIO使用

<!--more-->

> 官网：http://docs.minio.org.cn/docs/

# MinIO Docker快速入门

1.拉取镜像

```bash
docker pull minio/minio 
```

2.运行

```bash
docker run -d -p 9000:9000 \
--name minio \
-v /mnt/data:/usr/local/minio \
-v /mnt/config:/root/.minio \
-e "MINIO_ACCESS_KEY=minio" \
-e "MINIO_SECRET_KEY=minio" \
minio/minio server /usr/local/minio
```



# SpringBoot整合MinIO

> minIO-demo:https://github.com/liuurick/springboot-learning-example/tree/master/minio-demo