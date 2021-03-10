---
title: git常见错误
date: 2019-07-15 17:02:07
tags: git常见错误
categories: [开发工具,代码管理工具,Git]
---

记录git常见错误

<!--more-->

一般是没有初始化git本地版本管理仓库，所以无法执行git命令 。

操作之前执行以下命令行: `git init` 
然后执行一下`git status`查看状态信息



1.git Could not read from remote repository

```
git remote add origin git@github.com:liuurick/BlogBackup.git
git remote -v
git bash输出
```



2.error: failed to push some refs to xxx

> ! [rejected]        master -> master (fetch first)
> error: failed to push some refs to 'git@github.com:liuurick/BlogBackup.git'
> hint: Updates were rejected because the remote contains work that you do
> hint: not have locally. This is usually caused by another repository pushing
> hint: to the same ref. You may want to first integrate the remote changes
> hint: (e.g., 'git pull ...') before pushing again.
> hint: See the 'Note about fast-forwards' in 'git push --help' for details.


查了几种解决方式都不太管用，最后发现是由于github中的README.md文件不在本地代码目录中

检查了一下果然如此！
这时候可以通过 `git pull --rebase origin master` 把README.md文件克隆到本地库。

git pull --rebase origin master
最后提交：`git push origin master`

**下次遇到这样的问题，记得检查.gitignore和README.md文件**



