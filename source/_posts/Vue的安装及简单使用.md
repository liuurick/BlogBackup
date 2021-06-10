---
title: Vue的安装及简单使用
date: 2021-04-25 12:00:35
tags: Vue
categories: [应用框架,前端,组件化框架]
---

[TOC]

<!--more-->

 # 1 node 环境的安装

![img](https://note.youdao.com/yws/res/3/7FFB1B52E6EB475F9D423F472BFABBDD)

简单的说 Node.js 就是运行在服务端的 JavaScript。

Node.js 是一个基于Chrome JavaScript 运行时建立的一个平台。

Node.js是一个事件驱动I/O服务端JavaScript环境，基于Google的V8引擎，V8引擎执行Javascript的速度非常快，性能非常好。

> 在安装Vue前要先安装node，下载网址如下：http://nodejs.cn/download/

下载后，直接安装，一路“下一步”即可。

安装完成后，可以使用node --version查看安装后的版本

**NPM**是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：

- 允许用户从NPM服务器下载别人编写的第三方包到本地使用。
- 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
- 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。

由于新版的nodejs已经集成了npm，所以之前npm也一并安装好了。同样可以通过输入 **"npm -v"** 来测试是否成功安装。命令如下，出现版本提示表示安装成功:

# 2 Vue环境安装与使用

Vue有三种使用方式，前面的两种适合**学习**，理解vue思想及各个成员的用法，第3种适合进行快速项目开发。

- 简单的页面引入方式
- 使用`npm`安装使用
- 使用`vue cli`脚手架

## 2.1 简单的页面引入方式

分为两种

- 直接下载到本地使用

- 进行CDN引用



## 2.2 使用npm安装使用

```bash
npm install vue
```



## 2.3 使用vue cli脚手架

```bash
npm intall vue-cli -g     // 3.0以前的版本

npm install @vue/cli -g    // 3.0以后 的版本

有时候，npm的速度可能会非常的慢，这时可以考虑安装淘宝镜像中的cnpm,命令如下：
cnpm的安装
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

安装查看版本

```bash
cnpm -v
```

使用cnpm安装vue-cli

```bash
cnpm install @vue/cli -g    // 3.0以后 的版本
```



# 3 在控制台中使用Vue脚手架创建项目

按官网说明，在项目的父文件夹中以`管理员身份`启动cmd，使用如下的命令来创建项目：

```bash
vue create hello
```

[vue : 无法加载文件 C:\Users\XXX\AppData\Roaming\npm\vue.ps1，因为在此系统上禁止运行脚本](https://www.cnblogs.com/YangxCNWeb/p/11773175.html)

**问题: **使用命令行安装完成vue/cli后，使用vue ui无法创建demo

![image-20210526121256936](/images/2021052503.png)vue : 无法加载文件 C:\Users\admin\AppData\Roaming\npm\vue.ps1，因为在此系统上禁止运行脚本。有关详细信息，请参阅 [https:/](https://www.cnblogs.com/)go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。

所在位置 行:1 字符: 1

- vue ui
- \~~~

- - CategoryInfo : SecurityError: (:) []，PSSecurityException
  - FullyQualifiedErrorId : UnauthorizedAccess

**解决办法:**

**运行powershell（管理员身份）**

输入 `set-ExecutionPolicy RemoteSigned`，然后输入`A`



## 3.1 使用Vue UI图形化界面创建项目

在项目的父目录中，以`管理员身份`打开控制台，使用如下的命令：

```
vue ui
```

即可以打开图形化界面来可视化的创建项目



## 3.2 使用Vue创建Element项目

使用命令行方法创建一个项目myelement，可以按需要进行选择，使用的命令如下：

```bash
vue create mylement
```



创建完成后的界面如下：

使用命令：

```bash
cd mylement

npm run serve
```

会启动服务器，提供了两个地址，一个是localhost，一个是ip地址

