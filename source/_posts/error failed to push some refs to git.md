---
title: 'error: failed to push some refs to git@github.com:liuurick/BlogBackup.git'
date: 2019-07-30 12:05:11
categories: git常见错误
tags: git
---
### 

<!--more-->

git push的时候报错：

> ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'git@github.com:liuurick/BlogBackup.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.


查了几种解决方式都不太管用，最后发现是由于github中的README.md文件不在本地代码目录中

检查了一下果然如此！
这时候可以通过 `git pull --rebase origin master` 把README.md文件克隆到本地库。

git pull --rebase origin master
最后提交：`git push origin master`