---
title: Win常用快捷键与操作
date: 2021-04-04 21:03:50
tags: [Windows,快捷键]
categories: [开发工具,快捷键]
---

常用Windows快捷键命令与操作

<!--more-->

# 常用Windows快捷键命令

1.创建虚拟桌面

> Win：Windows键+CTRL+D
>
> MAC：CONTROL+UP

2.顺序选中

> Shift + 鼠标左键		

3.随意选中 / 取消选择

>Ctrl + 鼠标左键

4.快速到行首行尾

> Home & End

5.快速选中至行首或行尾

> Shift + Home & Shift + End

6.快速到文档首或尾

> Ctrl + Home & Ctrl + End

7.选中光标前面 / 后面 的所有的内容

> Ctrl + Shilf + Home & Ctrl + Shilf + End

8.在选项卡上向前移动

> ALT + Tab　　

9.在选项卡上向后移动

> ALT + Shift + Tab　　



# 常用操作

## 1. cmd查杀端口

方法一：

> 1.win + R 打开命令行窗口
>
> 2.netstat -ano 显示端口
>
> 3.taskkill /pid 端口号 -f

方法二：

> 1.netstat -aon|findstr "8081"
>
> 2.taskkill /pid 端口号 -f