---
title: 浅析Java平台无关性的实现
date: 2020-10-22 21:45:23
tags: [Java,JVM]
categories: Java
---

### 

<!--more-->	

​	平台无关性就是一种语言在计算机上的运行不受平台的约束，一次编译，到处执行（Write Once ,Run Anywhere）。

​	简而言之，Java是通过`javac`将java代码编译成字节码文件，这也是java跨平台的基础，接着虚拟机将class文件翻译成机器可识别的机器码。这也就实现了java的跨平台性。



### 通过代码来理解

```java
public class bytecode {
    public static void main(String[] args) {
        int i = 1,j=1;
        i++;
        ++j;
        System.out.println("i="+i+"-----j="+j);
    }
}
```



反编译之后：

可以看到JVM帮我们做了很多事情

```java
public class bytecode {
    public bytecode() {
    }

    public static void main(String[] var0) {
        byte var1 = 1;
        byte var2 = 1;
        int var3 = var1 + 1;
        int var4 = var2 + 1;
        System.out.println("i=" + var3 + "-----j=" + var4);
    }
}
```

