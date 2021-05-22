---
title: Java IO流详解
date: 2020-12-27 22:01:11
tags: IO流
categories: [编程基础,Java语言,JavaSE基础]
---

[TOC]

<!--more-->

> Java之美[从菜鸟到高手演变]之Java中的IO:https://blog.csdn.net/zhangerqing/article/details/8466532
>
> Java IO流学习总结一：输入输出流:https://blog.csdn.net/zhaoyanjun6/article/details/54292148
>
> 史上最骚最全最详细的IO流教程，没有之一！:https://www.cnblogs.com/yichunguo/p/11775270.html
>
> Java IO:http://www.cyc2018.xyz/Java/Java%20IO.html#%E4%B8%80%E3%80%81%E6%A6%82%E8%A7%88

# 1 概述

Java IO流大概可以分为以下几类：

- 磁盘操作：File类
- 字节流操作：InputStream，OutputStream
- 字符流操作：Reader，Writer
- 对象操作：Serializable
- 网络操作：socket
- 新的输入/输出：NIO



# 2 磁盘操作

File类可以用于获取目录和文件的信息，但他不表示文件的内容。

```java
public static void fileMethod(String path) {
        File file = new File(path);
        if (file.exists()) {
            File[] files = file.listFiles();
            if (null != files) {
                for (File file2 : files) {
                    if (file2.isDirectory()) {
                        System.out.println("文件夹:" + file2.getAbsolutePath());
                        fileMethod(file2.getAbsolutePath());
                    } else {
                        System.out.println("文件:" + file2.getAbsolutePath());
                    }
                }
            }
        } else {
            System.out.println("文件不存在!");
        }
    }
```

从 Java7 开始，可以使用 Paths 和 Files 代替 File。文件的操作更加简便快捷

> Java文件处理：Paths和Files：https://www.jianshu.com/p/3b3c5dbcfae7



# 3 字节流操作

实现文件复制



# 4 字符流操作

# 5 对象操作

> 对象的序列化:https://liuyangjun.blog.csdn.net/article/details/80422143





# 6 网络操作





# 7 新的输入/输出（NIO）

> https://www.zhihu.com/question/29005375

NIO主要有**三个核心部分组成**：

- **buffer缓冲区**
- **Channel管道**
- **Selector选择器**