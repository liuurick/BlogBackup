---
title: .gitignore的使用
date: 2021-01-20 11:52:40
tags: gitignore
categories: [开发工具,代码管理工具,Git]
---

很长一段时间忽视了.gitignore的作用，果真是自己太菜了。。。

[TOC]

<!--more-->

> 官方文档：https://github.com/github/gitignore

# 官方模板

.gitignore文件

```text
# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
```



# 个人常用模板

.gitignore文件

```text
# Compiled class file #
*.class

# Eclipse #
.project
.classpath
.settings/

# Intellij #
*.ipr
*.iml
*.iws
.idea/

# Maven #
target/

# Gradle #
build
.gradle

# Log file #
*.log
log/

# out #
**/out/

# Mac #
.DS_Store

# others #
*.jar
*.war
*.zip
*.tar
*.tar.gz
*.pid
*.orig
temp/
```



# 可能遇到的问题

## .gitignore规则不生效

.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。

解决方法就是先把本地缓存删除（改变成未track状态），然后再提交:

```bash
$ git rm -r --cached .
$ git add .
$ git commit -m 'update .gitignore'
```

如果你确实想添加该文件，可以用-f强制添加到Git：

```bash
$ git add -f xxx.class
```

或者你发现，可能是.gitignore写得有问题，需要找出来到底哪个规则写错了，

可以用`git check-ignore`命令检查：

```bash
$ git check-ignore -v xxx.class
.gitignore:3:*.class    xxx.class
```