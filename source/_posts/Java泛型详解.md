---
title: Java泛型详解
date: 2020-12-27 18:32:16
tags: 泛型
categories: [编程基础,Java语言,JavaSE基础]
---

从今天开始复习Java基础知识，泛型是JavaSE中一个很重要的知识点，这里写一篇文档，强化自己的记忆。

[TOC]

<!--more-->

> 泛型就这么简单:https://segmentfault.com/a/1190000014120746
>
> java 泛型详解:https://blog.csdn.net/s10461/article/details/53941091

# 1 泛型概念

>**Java泛型设计原则：只要在编译时期没有出现警告，那么运行时期就不会出现ClassCastException异常**.
>
>泛型：**把类型明确的工作推迟到创建对象或调用方法的时候才去明确的特殊的类型**

**让数据类型变得参数化**，也就是所操作的数据类型被指定为一个参数

- 定义泛型时，对应的数据类型是不确定的
- 泛型方法被调用时，会指定具体类型
- 核心目标：解决容器类型在编译时安全检查的问题

泛型优点：

- 类型安全
- 消除了强制类型的转换



demo:

```java
public class GenericDemo {
    public static void main(String[] args) {
        List ls = new ArrayList();
        ls.add("hello");
        ls.add(1);

        for (int i = 0; i < ls.size(); i++) {
            String item = (String) ls.get(i);
            System.out.println(item);
        }
    }
}
```

报错如下：

![image-20201221104504502](C:\Users\admin\Desktop\blog\source\images\2020122101.png)

这个类型转换的错误在运行期才报出来，显然是不符合我们的预期，应该在编译器就应该解决掉。

设置List容器的类型，就可以解决上面的问题

![image-20201221104807273](C:\Users\admin\Desktop\blog\source\images\2020122102.png)



# 2 泛型基础

## 1.泛型类

- 泛型类的定义语法：

```java
class 类名称<泛型标识 : 可以随便写任意标识号 , 标识指定的泛型的类型>{
	修饰符 泛型标识 /* (成员变量类型) */
	修饰符 构造函数(泛型标识 参数)
	......
	}
}
```

- 常用的泛型标识：`T、E、K、V`

  - E - Element (在集合中使用，因为集合中存放的是元素)
  - T - Type（表示Java 类，包括基本的类和我们自定义的类）
  - K - Key（表示键，比如Map中的key）
  - V - Value（表示值）
  - N - Number（表示数值类型）
  - ？ - （表示不确定的java类型）
  - S、U、V - 2nd、3rd、4th types

- 使用语法：

  `类名<具体的数据类型> 对象名 = new 类名<具体的数据类型>();`

- Java7以后，后面的<>中的具体的数据类型可以省略不写

  `类名<具体的数据类型> 对象名 = new 类名<>();`

```java
@Data
public class GenericClassExample<T> {
    //member这个成员变量的类型为T，T的类型由外部指定
    private T member;
    
    //泛型构造方法形参member的类型也为T，T的类型由外部指定
    public GenericClassExample(T member) {
        this.member = member;
    }
    
    // member的get set方法
    
    public T handleSomething(T target){
        return target;
    }
    
    //泛型类中是可以定义正常的方法
    public String sayhello(String name){
        return "hello "+ name;
    }
}
```

![image-20201221111507121](C:\Users\admin\Desktop\blog\source\images\2020122103.png)



**注意：**

- 泛型类在创建对象的时候，没有指定类型，将按照Object类型来操作

- 泛型参数不支持基本类型，只能是类类型

  ![image-20201221113906058](C:\Users\admin\Desktop\blog\source\images\2020122104.png)

- 泛型相关的信息不会进入到运行时阶段

  ![image-20201221114453417](C:\Users\admin\Desktop\blog\source\images\2020122105.png)

- 同一泛型类，根据不同数据类型创建对象，本质是同一类型

  ```java
  /**
   * 泛型类的定义
   * @param <T> 泛型标识-类型形参
   *           T 创建对象的时候指定具体的数据类型
   */
  public class Generic<T> {
      /**
       * T，是由外部
       */
      private T key;
  
      public Generic(T key) {
          this.key = key;
      }
  
      public T getKey() {
          return key;
      }
  
      public void setKey(T key) {
          this.key = key;
      }
  
      @Override
      public String toString() {
          return "GenericDemo{" +
                  "key=" + key +
                  '}';
      }
  }
  ```

  ```
  public class main {
      public static void main(String[] args) {
          Generic<String> stringGeneric = new Generic<>("hello");
          Generic<Integer> integerGeneric = new Generic(100);
  
          System.out.println(stringGeneric.getClass());
          System.out.println(integerGeneric.getClass());
  
          System.out.println(stringGeneric.getClass() == integerGeneric.getClass());
      }
  }
  ```

  ![image-20210102213439918](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210102213439918.png)

## 2.泛型类派生子类

- 子类也是泛型类，子类和父类的泛型类型要一致

  `class ChildGeneric<T> extends Generic<T>`

  ```java
  //父类
  public class Parent<E> {
      private E value;
      public E getValue() {
          return value;
      }
      public void setValue(E value) {
          this.value = value;
      }
  }
  
  /**
   * 泛型类派生子类，子类也是泛型类，那么子类的泛型标识要和父类一致。
   * @param <T>
   */
  public class ChildFirst<T> extends Parent<T> {
      @Override
      public T getValue() {
          return super.getValue();
      }
  }
  
  ```

  

- 子类不是泛型类，父类要明确泛型的数据类型

  `class ChildGeneric extends Generic<String> `

  ```java
  /**
   * 泛型类派生子类，如果子类不是泛型类，那么父类要明确数据类型
   */
  public class ChildSecond extends Parent<Integer> {
      @Override
      public Integer getValue() {
          return super.getValue();
      }
      @Override
      public void setValue(Integer value) {
          super.setValue(value);
      }
  }
  ```

  

> 能否在泛型里面使用具备继承关系的类?
>
> 不能使用
>
> - 使用通配符？，但是会使得泛型的类型检查失去意义
> - 给泛型加入上边界 `? extends E`
> - 给泛型加入下边界 `? super E`

```java
public class GenericDemo {
    public static void handleMember(GenericClassExample<?> integerGenericClassExample){
        Integer result = 111 + (Integer)integerGenericClassExample.getMember();
        System.out.println(result);
    }

    public static void main(String[] args) {
        GenericClassExample<String> stringGE = new GenericClassExample<String>("hello");
        GenericClassExample<Number> integerGE = new GenericClassExample<Number>(123);
        System.out.println(stringGE.getMember().getClass());
        System.out.println(integerGE.getMember().getClass());
        System.out.println(stringGE.sayhello("liubin"));

        handleMember(integerGE);
    }
}
```





## 3.泛型接口

> 泛型接口与泛型类的用法基本相同，常用于数据类型的生产工厂接口中

**定义方法：**

```java
interface 接口名称 <泛型标识，泛型标识，…> {
	泛型标识 方法名(); 
	.....
}
```

**泛型接口的使用**：

- 实现类也是泛型类，实现类和接口的泛型类型要一致

```java
/**
 * 泛型接口
 * @param <T>
 */
public interface Generator<T> {
    T getKey();
}
/**
 * 泛型接口的实现类，是一个泛型类，
 * 那么要保证实现接口的泛型类泛型标识包含泛型接口的泛型标识
 * @param <T>
 * @param <E>
 */
public class Pair<T,E> implements Generator<T> {

    private T key;
    private E value;

    public Pair(T key, E value) {
        this.key = key;
        this.value = value;
    }

    @Override
    public T getKey() {
        return key;
    }

    public E getValue() {
        return value;
    }
}
```

- 实现类不是泛型类，接口要明确数据类型

```java
/**
 * 实现泛型接口的类，不是泛型类，需要明确实现泛型接口的数据类型。
 */
public class Apple implements Generator<String> {
    @Override
    public String getKey() {
        return "hello generic";
    }
}
```

## 4.泛型通配符

我们知道`Ingeter`是`Number`的一个子类，同时在特性章节中我们也验证过`Generic<Ingeter>`与`Generic<Number>`实际上是相同的一种基本类型。那么问题来了，在使用`Generic<Number>`作为形参的方法中，能否使用`Generic<Ingeter>`的实例传入呢？在逻辑上类似于`Generic<Number>`和`Generic<Ingeter>`是否可以看成具有父子关系的泛型类型呢？

为了弄清楚这个问题，我们使用`Generic<T>`这个泛型类继续看下面的例子：

```java
public void showKeyValue1(Generic<Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}123
Generic<Integer> gInteger = new Generic<Integer>(123);
Generic<Number> gNumber = new Generic<Number>(456);

showKeyValue(gNumber);

// showKeyValue这个方法编译器会为我们报错：Generic<java.lang.Integer> 
// cannot be applied to Generic<java.lang.Number>
// showKeyValue(gInteger);12345678
```

通过提示信息我们可以看到`Generic<Integer>`不能被看作为``Generic<Number>`的子类。由此可以看出:**同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的**。

回到上面的例子，如何解决上面的问题？总不能为了定义一个新的方法来处理`Generic<Integer>`类型的类，这显然与java中的多台理念相违背。因此我们需要一个在逻辑上可以表示**同时**是`Generic<Integer>`和`Generic<Number>`父类的引用类型。由此类型通配符应运而生。

我们可以将上面的方法改一下：

```java
public void showKeyValue1(Generic<?> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}123
```

类型通配符一般是使用？代替具体的类型实参，注意了，**此处’？’是类型实参，而不是类型形参** 。重要说三遍！**此处’？’是类型实参，而不是类型形参** ！ **此处’？’是类型实参，而不是类型形参** ！再直白点的意思就是，此处的？和Number、String、Integer一样都是一种实际的类型，可以把？看成所有类型的父类。是一种真实的类型。

可以解决当具体类型不确定的时候，这个通配符就是 **?** ；**当操作类型时，不需要使用类型的具体功能时，只使用Object类中的功能。那么可以用 ? 通配符来表未知类型。**

## 5.泛型方法

> 泛型方法既能用在泛型类、泛型接口里，也能用在普通类或者接口里

