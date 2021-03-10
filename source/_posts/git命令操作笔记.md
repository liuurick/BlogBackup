---
title: git命令操作笔记
date: 2019-12-25 12:39:19
tags: git常见命令
categories: [开发工具,代码管理工具,Git]
---

[TOC]

<!--more-->

>git官网教程：https://git-scm.com/book/zh/v2
>
>codesheep：https://mp.weixin.qq.com/s/DQVVYOWdOPuRsy3m0fg6Xg
>
>图解Git：https://marklodato.github.io/visual-git-guide/index-zh-cn.html
>
>Pro Git：https://bingohuang.gitbooks.io/progit2/content/

![image](/images/2018062701.png)



创建一个目录用于存放仓库，或在有内容的目录下操作。

然后切换到此目录下初始化：`git --bare init`

```shell
git status 查看仓库状态
branches 分支目录
config 定义项目特有的配置选项
description 仅供git web使用
HEAD 当前分支
hooks 包含git钩子文件
info 包含一个全局排除文件
objects 存放所有数据内容，有info、pack两个子文件夹
refs 存放只想数据（分支）的提交对象的指针
index 保存暂存区信息，在执行git 初始化的时候这个文件还不存在,git add 后生成
工作目录 暂存区 本地仓库 远程仓库
git add git commit git push
```



```shell
git add FILE 添加file到暂存区
git add . 添加目录中所有改动过的文件到暂存区
git rm --cached FILE 将暂存区的FILE 撤回到工作区
git rm -f FILE 同时删除暂存区、工作区的FILE（即直接删除暂存区的文件）
git commit -m "add newfile a" 添加到本地仓库（相当于做了一次快照，可根据引号中内容恢复）
【真正意义上的通过版本控制系统管理文件：工作目录必须有代码文件，通过git add file添加到暂存区，通过git commit -m "输入的备注"添加到本地仓库】
修改文件名两种方式：
1.mv a a.txt 即先删除a 然后生成了a.txt，所以改名不用这个 （删除本地文件）
git rm --cached a
git add a.txt 然后git status 即可看到这两条命令即是rename （删除暂存区文件）
git commit -m "modified a a.txt" a改名a.txt并提交
2.git mv old new 直接更改文件名，改完直接git commit提交即可
```



```shell
git diff 比对工作目录与缓存区有什么不同
git diff --cached 比对暂存区与本地仓库有什么不同

ls检查下当前目录下是否有仓库信息，
git remote add origin git@10.0.0.227:web/control.git 创建远程仓库origin
git remote 查看当前远程仓库的名称
git remote remove origin 删除远程仓库origin
```

# 日志命令

```shell
git log 查看历史提交信息
git log --online 查看历史提交信息的哈希值
git log --online --decorate 历史提交信息并查看当前指针位置
git log -p 展示具体变化内容
git log -1 展示一条提交信息的内容
git log -1 -p 展示详细具体的最后一条变更的信息内容
```

# 初始化命令

```shell
git reset --hard xx 恢复到从前的位置
git reflog 查看所有历史提交信息，包括回复到指定位置之前的
git reset --hard xxx 前后都能回滚 来回滚
```

# 分支管理

```shell
git branch 查看分支
git branch fenzhi 创建分支fenzhi
git checkout fenzhi 切换分支fenzhi
git checkout -b testing 创建并切换到分支testing
删除分支要先切换到master然后删除创建的testing ,删除也会给自动创建一个快照，可恢复
git branch -d testing 删除分支testing
git merge testing 合并分支
冲突时，直接编辑冲突的文件，例如：vi aaa 然后去掉大于号、小于号、等于号，然后选择保留的代码，可都保留。
```

# 版本管理

```shell
git tag -a v1.0 -m "hehe" 当前状态打标签为V1.0
git tag -a V1.0 haxizhi -m "hehe" 把某个哈希状态的状态打标签
git show v1.0 查看某个标签的信息
git reset --hard v2.0 回滚数据到V2.0
git tag -d v2.0 删除V2.0标签的数据
```



