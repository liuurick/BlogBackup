---
title: Spring源码的下载与编译
date: 2020-10-19 19:13:57
tags: Spring
categories: Spring源码
---
Spring学了很久了，但是一直没有深入到源码，希望在接下来的源码学习中有所收获。。。
<!--more-->

### 一、环境安装
安装Idea，JDK，maven并完成相应的配置
![image-20201019191945647](/images/2020101901.png)

**注意：**当前官方Spring最新版本为5.3.0，文档要求JDK版本需要在 `JDK 8 update 262 or later`。

因为JDK官网只能下载到1.8_261，所以我使用的Spring-5.2.9

![image-20201019192232952](/images/2020101902.png)



### 二、下载和编译

#### 2.1 下载并解压

![image-20201019192816197](/images/2020101903.png)

#### 2.2 修改build.gradle

在build.gradle添加阿里云的中央仓库地址，提高依赖文件的下载速度

```bash
repositories {
			maven { url 'https://maven.aliyun.com/repository/public'}
			maven { url 'https://maven.aliyun.com/repository/public'}
			mavenCentral()
			maven { url "https://repo.spring.io/libs-spring-framework-build" }
		}
```

![image-20201019195803623](/images/2020101904.png)

**查看阿里云Maven中央仓库：**https://maven.aliyun.com/mvn/guide



#### 2.3 安装官方文档进行配置

![image-20201019200051103](/images/2020101906.png)



1.Precompile `spring-oxm` with `./gradlew :spring-oxm:compileTestJava`

![image-20201019201845436](/images/2020101907.png)

2.Import into IntelliJ (File -> New -> Project from Existing Sources -> Navigate to directory -> Select build.gradle)

导入项目

![image-20201019211425497](/images/2020101908.png)

3.When prompted exclude the `spring-aspects` module (or after the import via File-> Project Structure -> Modules)

因为`spring-aspects`有自己的编译器(AJC)，AJC会影响JVM的加载，所以需要排除出去

这里右键项目--》Load/Unload Modules--》选择spring-aspects--》unload--》重新加载即可

![image-20201019212036285](/images/2020101909.png)

4.Code away

到这里就完成Spring源码的下载与编译

### 三、通过一个小demo来测试一下

