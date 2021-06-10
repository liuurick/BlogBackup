---
title: Java注解详解
date: 2020-12-27 18:32:49
tags: 注解
categories: [编程基础,Java语言,JavaSE基础]
---

[TOC]

<!--more-->

> https://www.zhihu.com/question/24401191/answer/1724982163

# 概述

**格式**

```java
public @interface 注解名称{
    属性列表;
}
```

格式有点奇怪，我们稍后再研究。

**分类**

大致分为三类：自定义注解、JDK内置注解、还有第三方框架提供的注解。

- 自定义注解就是我们自己写的注解，比如@UserLog
- JDK内置注解，比如@Override检验方法重写，@Deprecated标识方法过期等
- 第三方框架定义的注解比如SpringMVC的@Controller等

**使用位置**

实际开发中，注解常常出现在类、方法、成员变量、形参位置。当然还有其他位置，这里不提及。

**作用**

如果说注释是写给人看的，那么注解就是写给程序看的。**它更像一个标签，**贴在一个类、一个方法或者字段上。它的目的是**为当前读取该注解的程序提供判断依据及少量附加信息。**比如程序只要读到加了@Test的方法，就知道该方法是待测试方法，又比如@Before注解，程序看到这个注解，就知道该方法要放在@Test方法之前执行。有时我们还可以通过注解属性，为将来读取这个注解的程序提供必要的附加信息，比如@RequestMapping("/user/info")提供了Controller某个接口的URL路径。

**级别**

注解和类、接口、枚举是同一级别的。

## 注解的本质

@interface和interface从名字上看非常相似，我猜注解的本质是一个接口（当然，这是瞎猜）。

为了验证这个猜测，我们做个实验。先按上面的格式写一个注解（暂时不附加属性）：

![img](https://pic4.zhimg.com/50/v2-29d4d9135b5a7d9ef502b72b16c0ad3f_hd.jpg?source=1940ef5c)![img](https://pic4.zhimg.com/80/v2-29d4d9135b5a7d9ef502b72b16c0ad3f_720w.jpg?source=1940ef5c)

编译后得到字节码文件：

![img](https://pic4.zhimg.com/50/v2-9d83edc7e199e02561a554323e226743_hd.jpg?source=1940ef5c)![img](https://pic4.zhimg.com/80/v2-9d83edc7e199e02561a554323e226743_720w.jpg?source=1940ef5c)

通过XJad工具反编译MyAnnotation.class：

![img](https://pic2.zhimg.com/50/v2-670bbfacda7ddffba009f9c6596d7029_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-670bbfacda7ddffba009f9c6596d7029_720w.jpg?source=1940ef5c)

我们发现，@interface变成了interface，而且自动继承了Annotation：

![img](https://pic1.zhimg.com/50/v2-04276048f7c252af482c859ff8eb920c_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-04276048f7c252af482c859ff8eb920c_720w.jpg?source=1940ef5c)

既然确实是个接口，那么我们自然可以在里面写方法

![img](https://pic1.zhimg.com/50/v2-d81578e334afd9f58b11422d460eef5a_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-d81578e334afd9f58b11422d460eef5a_720w.jpg?source=1940ef5c)

得到class文件后反编译

![img](https://pic1.zhimg.com/50/v2-31f25ef01099745ce616e95f8848e4cc_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-31f25ef01099745ce616e95f8848e4cc_720w.jpg?source=1940ef5c)

由于接口默认方法的修饰符就是public abstract，所以可以省略，直接写成：

```java
/**
 * @author qiyu
 */
public @interface MyAnnotation {
    String getValue();
}
```

虽说注解的本质是接口，但是仍然有很多怪异的地方，比如使用注解时，我们竟然可以给getValue()赋值：

```java
/**
 * @author qiyu
 */
@MyAnnotation(getValue = "annotation on class")
public class Demo {

    @MyAnnotation(getValue = "annotation on field")
    public String name;

    @MyAnnotation(getValue = "annotation on method")
    public void hello() {}

}
```

你见过给方法赋值的操作吗？（别闹了，你脑中想到的是给方法传参）。

虽然这里的getValue可能不是指getValue()，底层**或许**是getValue()返回的一个同名变量。但不管怎么说，还是太怪异了。所以在注解里，类似于String getValue()这种，被称为“属性”，而给属性赋值显然听起来好接受多了。

另外，我们还可以为属性指定默认值：

![img](https://pic3.zhimg.com/50/v2-456c8c4a196938283b93f1cf7093f6c4_hd.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-456c8c4a196938283b93f1cf7093f6c4_720w.jpg?source=1940ef5c)

![img](https://pic2.zhimg.com/50/v2-1eded742a59efaae6ec3381e6dcc5164_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-1eded742a59efaae6ec3381e6dcc5164_720w.jpg?source=1940ef5c)

当没有赋值时，属性将使用默认值，比如上面的defaultMethod()，它的getValue就是“no description"。

基于以上差异，以后还是把注解单独归为一类，不要当成接口使用。

## 反射注解信息

上文已经说过，注解就像一个标签，是贴在程序代码上供另一个程序读取的。所以三者关系是：

![img](https://pic4.zhimg.com/50/v2-ba15fc0944dd5b1cfe4f8d2d841faeed_hd.jpg?source=1940ef5c)![img](https://pic4.zhimg.com/80/v2-ba15fc0944dd5b1cfe4f8d2d841faeed_720w.jpg?source=1940ef5c)

要牢记，只要用到注解，必然有**三角关系：**

- **定义**注解
- **使用**注解
- **读取**注解

仅仅完成前两步，是没什么卵用的。就好比你写了一本武林秘籍却没人去学，那么这门武功还不如一把菜刀。

所以，接下来需要我们编写一个程序读取注解。读取注解的思路是：

![img](https://pic4.zhimg.com/50/v2-7d7b8b60ab8861c4c401f00257f76e0f_hd.jpg?source=1940ef5c)![img](https://pic4.zhimg.com/80/v2-7d7b8b60ab8861c4c401f00257f76e0f_720w.jpg?source=1940ef5c)

反射获取注解信息：

```java
/**
 * @author qiyu
 */
public class AnnotationTest {
    public static void main(String[] args) throws Exception {
        // 获取类上的注解
        Class<Demo> clazz = Demo.class;
        MyAnnotation annotationOnClass = clazz.getAnnotation(MyAnnotation.class);
        System.out.println(annotationOnClass.getValue());

        // 获取成员变量上的注解
        Field name = clazz.getField("name");
        MyAnnotation annotationOnField = name.getAnnotation(MyAnnotation.class);
        System.out.println(annotationOnField.getValue());

        // 获取hello方法上的注解
        Method hello = clazz.getMethod("hello", (Class<?>[]) null);
        MyAnnotation annotationOnMethod = hello.getAnnotation(MyAnnotation.class);
        System.out.println(annotationOnMethod.getValue());

        // 获取defaultMethod方法上的注解
        Method defaultMethod = clazz.getMethod("defaultMethod", (Class<?>[]) null);
        MyAnnotation annotationOnDefaultMethod = defaultMethod.getAnnotation(MyAnnotation.class);
        System.out.println(annotationOnDefaultMethod.getValue());

    }
}
```

Class、Method、Field对象都有个getAnnotation()方法，可以获取**各自位置上**的注解信息，但IDEA好像提示我们错误：

> Annotation 'MyAnnotation.class' is not retained for reflective。

直译的话就是：注解MyAnnotation并没有为反射保留。

![img](https://pic4.zhimg.com/50/v2-43f84b51cc7a84c40051f5bf159a03be_hd.jpg?source=1940ef5c)![img](https://pic4.zhimg.com/80/v2-43f84b51cc7a84c40051f5bf159a03be_720w.jpg?source=1940ef5c)

我不管，我要开搞了，哪轮得到你一个编译器在这瞎比比，直接run一下：

![img](https://pic2.zhimg.com/50/v2-9812a1e2284f56ed108f1d813ed22103_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-9812a1e2284f56ed108f1d813ed22103_720w.jpg?source=1940ef5c)

不听老人言，吃亏在眼前。

这是因为注解其实有所谓“保留策略”的说法。大家学习JSP时，应该学过<!-- -->和<%-- -->的区别：前者可以在浏览器检查网页源代码时看到，而另一个在服务器端输出时就被抹去了。同样的，注解通过保留策略，控制自己可以保留到哪个阶段。保留策略也是通过注解实现，它属于元注解，也叫元数据。

# 基本Annotation

`@Override`：限定重写父类方法，强制一个子类必须覆盖父类的的方法。

```java
@Target(value=METHOD)
@Retention(value=SOURCE)
public @interface Override
```



`@Deprecated`：标记已过时，用于表示某个程序元素（类，方法等）已过时，当其他程序使用已过时的类，方法时，编译器将会给出警告。

```java
@Documented
@Retention(value=RUNTIME)
@Target(value={CONSTRUCTOR,ElementType,LOCAL_VARIABLE,METHOD,PACKAGE,PARAMETER,TYPE})
public @interface Deprecated
```



`@SuppressWarnings`：抑制编译器警告，指示被该Annotation修饰的程序元素（以及该程序元素中的所有子元素）取消显示指定的编译器警告。

```java
@Target(value={TYPE,ElementType,METHOD,PARAMETER,CONSTRUCTOR,LOCAL_VARIABLE})
@Retention(value=SOURCE)
public @interface SuppressWarnings
```



`@SafeVarargs`：Java7的“堆污染”警告

堆污染：当把一个不带泛型的对象赋给一个带泛型的的变量时，就会引发堆污染。

```java
@Documented
@Retention(value=RUNTIME)
@Target(value={CONSTRUCTOR,METHOD})
public @interface SafeVarargs
```



`@FunctionalInterface`：函数式接口

```java
@Documented
@Retention(value=RUNTIME)
@Target(value=TYPE)
public @interface FunctionalInterface
```



# JDK的元Annotation

**所谓元注解，就是加在注解上的注解。**作为普通程序员，常用的就是：

- @Documented：用于制作文档，不是很重要，忽略便是
- @Target：加在注解上，限定该注解的使用位置。不写的话，好像默认各个位置都是可以的。
- @Retention（注解的保留策略）
- @Inherted（指定该注解将具有继承性）

所以如果需要限定注解的使用位置，可以**在自定义的注解上使用@Target（普通注解上使用元注解）**。我们本次默认即可，不特别限定。

![img](https://pic4.zhimg.com/50/v2-af3dfd2339cac04738b1d325d5b6b37d_hd.jpg?source=1940ef5c)![img](https://pic4.zhimg.com/80/v2-af3dfd2339cac04738b1d325d5b6b37d_720w.jpg?source=1940ef5c)

- @Retention（注解的保留策略）

![img](https://pic1.zhimg.com/50/v2-38c0293d6574504a1711d3714236adee_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-38c0293d6574504a1711d3714236adee_720w.jpg?source=1940ef5c)

注解的保留策略有三种：SOURCE/ClASS/RUNTIME

![img](https://pic2.zhimg.com/50/v2-14ec3964feb69f2e8ed2dc4a2e90d6a2_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-14ec3964feb69f2e8ed2dc4a2e90d6a2_720w.jpg?source=1940ef5c)

“保留策略”这个元注解的意义在哪呢？

一般来说，普通开发者使用注解的时机都是运行时，比如反射读取注解（也有类似Lombok这类编译期注解）。既然反射是运行时调用，那就要求注解的信息必须保留到虚拟机将.class文件加载到内存为止。如果你需要反射读取注解，却把保留策略设置为RetentionPolicy.SOURCE、RetentionPolicy.CLASS，那就读取不到了。

现在，我们已经完成了使用注解的三部曲：

定义注解

```java
/**
 * @author qiyu
 */
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String getValue() default "no description";
}
```

使用注解：

```java
/**
 * @author qiyu
 */
@MyAnnotation(getValue = "annotation on class")
public class Demo {

    @MyAnnotation(getValue = "annotation on field")
    public String name;

    @MyAnnotation(getValue = "annotation on method")
    public void hello() {}

    @MyAnnotation() // 故意不指定getValue
    public void defaultMethod() {}
}
```

重新运行程序，成功读取注解信息：

![img](https://pic1.zhimg.com/50/v2-627c701b41afad0504846e458ba849a0_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-627c701b41afad0504846e458ba849a0_720w.jpg?source=1940ef5c)

注意，defaultMethod()反射得到的注解信息是：no description，就是MyAnnotion中getValue的默认值。

但是，注解的读取并不只有反射一种途径。比如@Override，它由编译器读取（你写完代码ctrl+s时就编译了），而编译器只是检查语法错误，此时程序尚未运行。

![img](https://pic2.zhimg.com/50/v2-4cdb1cae3404986f86201c475d639a2c_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-4cdb1cae3404986f86201c475d639a2c_720w.jpg?source=1940ef5c)

保留策略为SOURCE，仅仅是源码阶段，编译成.class文件后就消失

![img](https://pic2.zhimg.com/50/v2-925faf75a3e6938b0e5c6200c08c21d0_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-925faf75a3e6938b0e5c6200c08c21d0_720w.jpg?source=1940ef5c)



## 属性的数据类型及特别的属性：value和数组

**属性的数据类型**

- 八种基本数据类型
- String
- 枚举
- Class
- 注解类型
- 以上类型的一维数组

```java
/**
 * @author qiyu
 */
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
   // 8种基本数据类型
    int intValue();
    long longValue();
    // ...其他类型省略

    // String
    String name();
    // 枚举
    CityEnum cityName();
    // Class类型
    Class<?> clazz();
    // 注解类型
    MyAnnotation2 annotation2();

    // 以上几种类型的数组类型
    int[] intValueArray();
    String[] names();
    // ...其他类型省略
}

@interface MyAnnotation2 {
}

enum CityEnum {
    BEIJING,
    HANGZHOU,
    SHANGHAI;
}
```

![img](https://pic2.zhimg.com/50/v2-80858cf1fd06fa7710650cd22920190d_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-80858cf1fd06fa7710650cd22920190d_720w.jpg?source=1940ef5c)

以Demo类上注解为例，演示给注解属性赋值多种类型：

```java
/**
 * @author qiyu
 */
@MyAnnotation(
        // 8种基本类型
        intValue = 1,
        longValue = 0L,

        // String
        name = "annotation on class",
        // 枚举
        cityName = CityEnum.BEIJING,
        // Class
        clazz = Demo.class,
        // 注解
        annotation2 = @MyAnnotation2,
        // 一维数组
        intValueArray = {1, 2},
        names = {"Are", "you", "OK?"}
)
public class Demo {
    // 省略...
}
```

![img](https://pic3.zhimg.com/50/v2-d3dd1bf78c616b1e02d3a6e71e5a3fb0_hd.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-d3dd1bf78c616b1e02d3a6e71e5a3fb0_720w.jpg?source=1940ef5c)

**value属性**

如果注解的属性只有一个，且叫value，那么使用该注解时，可以不用指定属性名，因为默认就是给value赋值：

![img](https://pic4.zhimg.com/50/v2-391ee80c3d5662e9eb2389c0d6b1a6ab_hd.jpg?source=1940ef5c)![img](https://pic4.zhimg.com/80/v2-391ee80c3d5662e9eb2389c0d6b1a6ab_720w.jpg?source=1940ef5c)

![img](https://pic1.zhimg.com/50/v2-3b72d5d30923c3544043405bc7865681_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-3b72d5d30923c3544043405bc7865681_720w.jpg?source=1940ef5c)

![img](https://pic2.zhimg.com/50/v2-c6b1084157082dcad07737fe9fcc978b_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-c6b1084157082dcad07737fe9fcc978b_720w.jpg?source=1940ef5c)

但是注解的属性如果有多个，无论是否叫value，都必须写明属性的对应关系：

![img](https://pic4.zhimg.com/50/v2-b4e2de16d6ea0c988436bc925e585385_hd.jpg?source=1940ef5c)![img](https://pic4.zhimg.com/80/v2-b4e2de16d6ea0c988436bc925e585385_720w.jpg?source=1940ef5c)

![img](https://pic1.zhimg.com/50/v2-0374e35f05ac834aa8f60b547a70d61a_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-0374e35f05ac834aa8f60b547a70d61a_720w.jpg?source=1940ef5c)

按IDEA的提示修正：

![img](https://pic2.zhimg.com/50/v2-edeb92ba769146886ca7a121e29c33a2_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-edeb92ba769146886ca7a121e29c33a2_720w.jpg?source=1940ef5c)

**数组属性**

如果数组的元素只有一个，可以省略花括号{}：

![img](https://pic2.zhimg.com/50/v2-f0e217faebdd695b55868312b4495d1f_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-f0e217faebdd695b55868312b4495d1f_720w.jpg?source=1940ef5c)

## 用常量类为注解属性赋值

如果你希望为注解的属性提供统一的几个可选值，可以使用常量类：

![img](https://pic1.zhimg.com/50/v2-bac6e555e0faae64518a8985664863e1_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-bac6e555e0faae64518a8985664863e1_720w.jpg?source=1940ef5c)

![img](https://pic1.zhimg.com/50/v2-af15e422b2cef1a5ecc410cd80451efc_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-af15e422b2cef1a5ecc410cd80451efc_720w.jpg?source=1940ef5c)

或者：

![img](https://pic1.zhimg.com/50/v2-2d8c97fd580722069d99411614c37eb4_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-2d8c97fd580722069d99411614c37eb4_720w.jpg?source=1940ef5c)

![img](https://pic1.zhimg.com/50/v2-d20bcb91fe2d44a6932feb1238d65f7c_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-d20bcb91fe2d44a6932feb1238d65f7c_720w.jpg?source=1940ef5c)

本质其实还是String，只不过是通过常量表现。上面只是举个例子，大家可以根据实际业务自由发挥。

## 山寨版Junit

还是三步曲：

![img](https://pic1.zhimg.com/50/v2-116b805ba12495e7d2758acd99dd6c4c_hd.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-116b805ba12495e7d2758acd99dd6c4c_720w.jpg?source=1940ef5c)

MyBefore注解（定义注解）

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyBefore {
}
```

MyTest注解（定义注解）

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyTest {
}
```

MyAfter注解（定义注解）

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyAfter {
}
```

EmployeeDAOTest（使用注解）

```java
/**
 * 和我们平时使用Junit测试时一样
 *
 * @author qiyu
 */
public class EmployeeDAOTest {
    @MyBefore
    public void init() {
        System.out.println("初始化...");
    }

    @MyAfter
    public void destroy() {
        System.out.println("销毁...");
    }

    @MyTest
    public void testSave() {
        System.out.println("save...");
    }

    @MyTest
    public void testDelete() {
        System.out.println("delete...");
    }
}
```

MyJunitFrameWork（读取注解）

```java
/**
 * 这个就是注解三部曲中最重要的：读取注解并操作
 * 相当于我们使用Junit时看不见的那部分（在隐秘的角落里帮我们执行标注了@Test的方法）
 *
 * @author qiyu
 */
public class MyJunitFrameWork {
    public static void main(String[] args) throws Exception {
        // 1.先找到测试类的字节码：EmployeeDAOTest
        Class clazz = EmployeeDAOTest.class;
        Object obj = clazz.newInstance();

        // 2.获取EmployeeDAOTest类中所有的公共方法
        Method[] methods = clazz.getMethods();

        // 3.迭代出每一个Method对象，判断哪些方法上使用了@MyBefore/@MyAfter/@MyTest注解
        List<Method> myBeforeList = new ArrayList<>();
        List<Method> myAfterList = new ArrayList<>();
        List<Method> myTestList = new ArrayList<>();
        for (Method method : methods) {
            if (method.isAnnotationPresent(MyBefore.class)) {
                //存储使用了@MyBefore注解的方法对象
                myBeforeList.add(method);
            } else if (method.isAnnotationPresent(MyTest.class)) {
                //存储使用了@MyTest注解的方法对象
                myTestList.add(method);
            } else if (method.isAnnotationPresent(MyAfter.class)) {
                //存储使用了@MyAfter注解的方法对象
                myAfterList.add(method);
            }
        }

        // 执行方法测试
        for (Method testMethod : myTestList) {
            // 先执行@MyBefore的方法
            for (Method beforeMethod : myBeforeList) {
                beforeMethod.invoke(obj);
            }
            // 测试方法
            testMethod.invoke(obj);
            // 最后执行@MyAfter的方法
            for (Method afterMethod : myAfterList) {
                afterMethod.invoke(obj);
            }
        }
    }
}
```

效果展示：

![img](https://pic4.zhimg.com/50/v2-93bf7c67a5e021d5de64d7d6147033bd_hd.jpg?source=1940ef5c)![img](https://pic4.zhimg.com/80/v2-93bf7c67a5e021d5de64d7d6147033bd_720w.jpg?source=1940ef5c)



## 小结

- 注解就像标签，是程序判断执行的依据。比如，程序读到@Test就知道这个方法是待测试方法，而@Before的方法要在测试方法之前执行
- 注解需要三要素：定义、使用、读取并执行
- 注解分为自定义注解、JDK内置注解和第三方注解（框架）。自定义注解一般要我们自己定义、使用、并写程序读取，而JDK内置注解和第三方注解我们只要使用，定义和读取都交给它们
- 大多数情况下，三角关系中我们只负责使用注解，无需定义和执行，框架会将**注解类**和**读取注解的程序**隐藏起来，除非阅读源码，否则根本看不到。**平时见不到定义和读取的过程，光顾着使用注解，久而久之很多人就忘了注解如何起作用了！**

![img](https://pic2.zhimg.com/50/v2-6b30334cc200e79829acfdac84d51ab7_hd.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-6b30334cc200e79829acfdac84d51ab7_720w.jpg?source=1940ef5c)