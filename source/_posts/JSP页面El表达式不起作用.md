---
title: JSP页面El表达式不起作用
date: 2020-11-22 09:21:39
tags: [JSP,session]
categories: JSP
---

首先查看web.xml文件 查看jsp servlet版本是多少？

<!--more-->

头文件中

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
```

此时的jsp servlet的版本为2.3，**Servlet2.3或者更早的JSP页面默认是不使用EL表达式的**，即不解析EL表达式，所以要不就是使用更高版本的jsp servlet。这样比较麻烦，这里Java提供了一各属性来解决这个问题，使2.3或更早版本能使用El表达式，该属性为`<%@page isELIgnored="false" %>`默认是true，表示暂时不解析，此时我们在jsp页面头文件中设置为false，再次运行，此时jsp页面就会解析El表达式