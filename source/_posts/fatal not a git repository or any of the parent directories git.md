---
title: 'fatal: not a git repository (or any of the parent directories): .git'
date: 2019-07-28 11:43:50
categories: git常见错误
tags: git
---
一般是没有初始化git本地版本管理仓库，所以无法执行git命令 

### 解决方法：
操作之前执行以下命令行: `git init` 
然后执行一下`git status`查看状态信息