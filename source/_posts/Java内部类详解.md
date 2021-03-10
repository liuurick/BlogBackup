---
title: Java内部类详解
date: 2021-12-27 11:09:06
tags: 内部类
categories: [编程基础,Java语言,JavaSE基础]
---

[TOC]

<!--more-->

> https://www.runoob.com/w3cnote/java-inner-class-intro.html
>
> https://www.iteye.com/topic/494230#

# 内部类概述

> 内部类：就是在一个类中定义一个类。举例：在一个类A的内部定义一个类B，类B就被称为内部类

**定义格式：**

```java
public class 类名{
	修饰符 class 类名{
	}
}
```



# 内部类分类

- 静态内部类：类似于静态变量

- 实例内部类：类似于实例变量

- 局部内部类：类似于局部变量



**注意：**

- 使用内部类编写的代码，可读性很差。能不用尽量不用。

- 匿名内部类是局部内部类的一种

```java
public class Test01 {

    /**
     * 该类在类的内部，所以称为内部类
     * 由于前面有static，称为静态内部类
     */
    static class Inner1{

    }

    /**
     * 该类在类的内部，所以称为内部类
     * 没有static称为实例内部类
     */
    class Inner2{

    }

    public void doSome(){
        //局部变量
        int num = 10;

        /**
         * 该类在类的内部，所以称为内部类
         * 局部内部类
         */
        class Inner3{

        }
    }

}
```



## 静态内部类

类似于静态变量



## 实例内部类

成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括private成员和静态成员）。

不过要注意的是，当成员内部类拥有和外部类同名的成员变量或者方法时，会发生隐藏现象，即默认情况下访问的是成员内部类的成员。如果要访问外部类的同名成员，需要以下面的形式进行访问：

```
外部类.this.成员变量
外部类.this.成员方法
```



虽然成员内部类可以无条件地访问外部类的成员，而外部类想访问成员内部类的成员却不是这么随心所欲了。在外部类中如果要访问成员内部类的成员，必须先创建一个成员内部类的对象，再通过指向这个对象的引用来访问：

```
class Circle {
    private double radius = 0;
 
    public Circle(double radius) {
        this.radius = radius;
        getDrawInstance().drawSahpe();   //必须先创建成员内部类的对象，再进行访问
    }
     
    private Draw getDrawInstance() {
        return new Draw();
    }
     
    class Draw {     //内部类
        public void drawSahpe() {
            System.out.println(radius);  //外部类的private成员
        }
    }
}
```

成员内部类是依附外部类而存在的，也就是说，如果要创建成员内部类的对象，前提是必须存在一个外部类的对象。创建成员内部类对象的一般方式如下：

```
public class Test {
    public static void main(String[] args)  {
        //第一种方式：
        Outter outter = new Outter();
        Outter.Inner inner = outter.new Inner();  //必须通过Outter对象来创建
         
        //第二种方式：
        Outter.Inner inner1 = outter.getInnerInstance();
    }
}
 
class Outter {
    private Inner inner = null;
    public Outter() {
         
    }
     
    public Inner getInnerInstance() {
        if(inner == null)
            inner = new Inner();
        return inner;
    }
      
    class Inner {
        public Inner() {
             
        }
    }
}
```

内部类可以拥有 private 访问权限、protected 访问权限、public 访问权限及包访问权限。比如上面的例子，如果成员内部类 Inner 用 private 修饰，则只能在外部类的内部访问，如果用 public 修饰，则任何地方都能访问；如果用 protected 修饰，则只能在同一个包下或者继承外部类的情况下访问；如果是默认访问权限，则只能在同一个包下访问。这一点和外部类有一点不一样，外部类只能被 public 和包访问两种权限修饰。我个人是这么理解的，由于成员内部类看起来像是外部类的一个成员，所以可以像类的成员一样拥有多种权限修饰。

类似于实例变量

```
public class Outer {
    public static void main(String[] args) {
        Outer o = new Outer();
        Outer.MyMath myMath = o.new MyMath();
        myMath.mysum(new Compute() {
            @Override
            public int sum(int x, int y) {
                return x+y;
            }
        }, 10, 20);
    }

    class MyMath{
        public void mysum(Compute c,int x ,int y){
            int value = c.sum(x,y);
            System.out.println(x + "+"+ y + "="+value);
        }
    }

    interface comput{
        public int sum(int x,int y);
    }

    class computImpl implements comput{

        @Override
        public int sum(int x, int y) {
            return x+y;
        }
    }
}
```



## 局部内部类

类似于局部变量