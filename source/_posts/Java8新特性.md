---
title: Java8新特性
date: 2020-12-24 11:07:44
tags: 函数式编程
categories: [编程基础,Java语言,JavaSE基础]
---

Java8提供了很多新特性，总结一下

1.lambda表达式

2.函数式（Functional）接口

3.方法引用/构造器引用

4.Stream API

- 并行流：
- 串行流

5.Optional类---最大化减少空指针异常

6.新的时间和日期API

<!--more-->

# 1 lambda表达式

> Lambda表达式详解:https://www.cnblogs.com/haixiang/p/11029639.html
>
> Lambda 表达式有何用处？如何使用？:https://www.zhihu.com/question/20125256/answer/324121308

lambda表达式是JDK8的一个新特性，直接感受是简化代码，个人认为主要目的是取代匿名内部类，从而写出更优雅的 Java 代码，尤其在集合的遍历和其他集合操作中，可以极大地优化代码结构。

## 1.1 为什么使用lambda表达式

Lambda 是一个**匿名函数**，我们可以把 Lambda 表达式理解为是**一段可以传递的代码**（将代码像数据一样进行传递）。可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使Java的语言表达能力得到了提升。

## 1.2 语法

```
lambda操作符 或箭头操作符
左边：lambda形参列表（其实就是接口中的抽象方法的形参列表）
右边：lambda体（其实就是重写的抽象方法的方法体）
```

## 1.3 lambda的使用

主要分为6种情况

- 无参数无返回值
- 一个参数无返回值
- 多个参数无返回值
- 无参数有返回值
- 一个参数有返回值
- 多个参数有返回值

## 1.4 lambda表达式的本质

作为函数式接口的实例

## 1.5 Lambda 表达式常用示例

### lambda 表达式引用方法

有时候我们不是必须要自己重写某个匿名内部类的方法，我们可以可以利用 lambda表达式的接口快速指向一个已经被实现的方法。

**语法：**` 方法归属者::方法名 静态方法的归属者为类名，普通方法归属者为对象`

```java
public class Exe1 {
    public static void main(String[] args) {
        ReturnOneParam lambda1 = a -> doubleNum(a);
        System.out.println(lambda1.method(3));

        //lambda2 引用了已经实现的 doubleNum 方法
        ReturnOneParam lambda2 = Exe1::doubleNum;
        System.out.println(lambda2.method(3));

        Exe1 exe = new Exe1();

        //lambda4 引用了已经实现的 addTwo 方法
        ReturnOneParam lambda4 = exe::addTwo;
        System.out.println(lambda4.method(2));
    }

    /**
     * 要求
     * 1.参数数量和类型要与接口中定义的一致
     * 2.返回值类型要与接口中定义的一致
     */
    public static int doubleNum(int a) {
        return a * 2;
    }

    public int addTwo(int a) {
        return a + 2;
    }
}
```

### 构造方法的引用

一般我们需要声明接口，该接口作为对象的生成器，通过`类名::new` 的方式来实例化对象，然后调用方法返回对象。

```java
interface ItemCreatorBlankConstruct {
    Item getItem();
}
interface ItemCreatorParamContruct {
    Item getItem(int id, String name, double price);
}

public class Exe2 {
    public static void main(String[] args) {
        ItemCreatorBlankConstruct creator = () -> new Item();
        Item item = creator.getItem();

        ItemCreatorBlankConstruct creator2 = Item::new;
        Item item2 = creator2.getItem();

        ItemCreatorParamContruct creator3 = Item::new;
        Item item3 = creator3.getItem(112, "鼠标", 135.99);
    }
}
```

### lambda 表达式创建线程

我们以往都是通过创建 Thread 对象，然后通过匿名内部类重写 run() 方法，一提到匿名内部类我们就应该想到可以使用 lambda 表达式来简化线程的创建过程。

```java
    Thread t = new Thread(() -> {
      for (int i = 0; i < 10; i++) {
        System.out.println(2 + ":" + i);
      }
    });
  	t.start();
```

### 遍历集合

我们可以调用集合的 `public void forEach(Consumer<? super E> action)` 方法，通过 lambda 表达式的方式遍历集合中的元素。以下是 Consumer 接口的方法以及遍历集合的操作。Consumer 接口是 jdk 为我们提供的一个函数式接口。

```java
    @FunctionalInterface
    public interface Consumer<T> {
        void accept(T t);
        //....
    }
      ArrayList<Integer> list = new ArrayList<>();

      Collections.addAll(list, 1,2,3,4,5);

      //lambda表达式 方法引用
      list.forEach(System.out::println);

      list.forEach(element -> {
        if (element % 2 == 0) {
          System.out.println(element);
        }
      });
```

### 删除集合中的某个元素

我们通过`public boolean removeIf(Predicate<? super E> filter)`方法来删除集合中的某个元素，Predicate 也是 jdk 为我们提供的一个函数式接口，可以简化程序的编写。

```java
      ArrayList<Item> items = new ArrayList<>();
      items.add(new Item(11, "小牙刷", 12.05 ));
      items.add(new Item(5, "日本马桶盖", 999.05 ));
      items.add(new Item(7, "格力空调", 888.88 ));
      items.add(new Item(17, "肥皂", 2.00 ));
      items.add(new Item(9, "冰箱", 4200.00 ));

      items.removeIf(ele -> ele.getId() == 7);

      //通过 foreach 遍历，查看是否已经删除
      items.forEach(System.out::println);
```

### 集合内元素的排序

在以前我们若要为集合内的元素排序，就必须调用 sort 方法，传入比较器匿名内部类重写 compare 方法，我们现在可以使用 lambda 表达式来简化代码。

```java
        ArrayList<Item> list = new ArrayList<>();
        list.add(new Item(13, "背心", 7.80));
        list.add(new Item(11, "半袖", 37.80));
        list.add(new Item(14, "风衣", 139.80));
        list.add(new Item(12, "秋裤", 55.33));

        /*
        list.sort(new Comparator<Item>() {
            @Override
            public int compare(Item o1, Item o2) {
                return o1.getId()  - o2.getId();
            }
        });
        */

        list.sort((o1, o2) -> o1.getId() - o2.getId());

        System.out.println(list);
```

## 1.6 Lambda 表达式中的闭包问题

这个问题我们在匿名内部类中也会存在，如果我们把注释放开会报错，告诉我 num 值是 final 不能被改变。这里我们虽然没有标识 num 类型为 final，但是在编译期间虚拟机会帮我们加上 final 修饰关键字。

```java
import java.util.function.Consumer;
public class Main {
    public static void main(String[] args) {

        int num = 10;

        Consumer<String> consumer = ele -> {
            System.out.println(num);
        };

        //num = num + 2;
        consumer.accept("hello");
    }
}
```

# 2  函数式（Functional）接口

函数式（Functional）接口：只包含一个抽象方法

Java内置的四大核心函数式接口：

![image](/images/2020120411.png)

# 3 方法引用/构造器引用

当要传递给Lamdba体的操作，已经有实现的方法了，可以使用方法引用。

方法引用可以看做是lambda表达式深层次的表达。换句话说，方法引用就是lambda表达式，也就是函数式接口的一个实例，通过方法的名字来指向一个方法，可以认为是lambda表达式的一个语法糖。

**要求：**实现接口的抽象方法的参数列表和返回值类型，必须与方法引用的方法的参数列表和返回值类型保持一致。

**格式：**使用操作符“::”将类（或对象）与方法名分隔开来

主要有三种使用情况：

- 对象::实例方法名
- 类::静态方法名
- 类::实例方法名



# 4 Stream API

## 4.1 概述

> Stream API（java.util.stream）把真正的函数式编程风格引入到Java中。这是目前为止对Java类库最好的补充，因为Stream API可以极大提供Java程序员的生成力，让程序员写出高效率，干净，简洁的代码。

Stream 是 Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。 **使用Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。**也可以使用 Stream API 来**并行**执行操作。简言之，Stream API 提供了一种高效且易于使用的处理数据的方式。

## 4.2 为什么使用Stream API

- 实际开发中，项目中多数数据源都来自于MySQL，Oracle等。但现在数据源可以更多了，有MonogoDB，Redis，而这些NoSQL的数据源就需要Java层面去处理。
- Stream和Collection集合的区别：
  - Collection是一种静态的内存数据结构，是主要面向内存，存储在内存中。
  - Stream是有相关计算的，主要是面向CPU，通过CPU实现计算。

## 4.3 什么是Stream

Stream是数据渠道，用于操作数据源（集合，数组等）所生成的元素序列。

**“集合讲的是数据，Stream讲的是计算”**

**注意：**

- Stream 自己不会存储元素。
- Stream 不会改变源对象。每次处理都会返回一个持有结果的新Stream。
- Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。

## 4.4 Stream 的操作三个步骤：

1. **创建 Stream**：通过一个数据源（如：集合、数组），获取一个流
2. **中间操作**：中间操作是个操作链，对数据源的数据进行n次处理，但是在终结操作前，并不会真正执行。
3. **终止操作**：一旦执行终止操作，就执行中间操作链，最终产生结果并结束Stream。
   ![在这里插入图片描述](/images/2020120410.png)

### 4.4.1 创建Stream流

#### 1.创建Stream方式一:通过集合

Java8中的Collection接口被扩展，提供了两个获取流的方法:

​		●default Stream stream() : 返回一个顺序流

​		●default Stream parallelStream() : 返回一个并行流

#### 2.创建Stream方式二:通过数组

Java8中的Arrays的静态方法stream()可以获取数组流:

​		● static Stream stream(T[] array): 返回一个流

重载形式，能够处理对应基本类型的数组:

​		●public static IntStream stream(int[] array)：返回一个整型数据流

​		●public static LongStream stream(long[] array)：返回一个长整型数据流

​		●public static DoubleStream stream(double[] array)：返回一个浮点型数据流

#### 3.创建Stream方式三:通过Stream的of()

可以调用Stream类静态方法`of()`，通过显示值创建一个流。它可以接受任意数量的参数

​		●public static Stream of(T… values) : 返回一个顺序流

#### 4.创建Stream方式四:创建无限流

可以使用静态方法 Stream.iterate() 和Stream.generate(), 创建无限流。

​		●public static Stream iterate(final T seed, final UnaryOperator f):返回一个无限流

​		● public static Stream generate(Supplier s) ：返回一个无限流



### 4.4.2 中间处理数据操作

1. filter(Predicate) 筛选流中某些元素
2. map(Function f) 接收流中元素，并且将其映射成为新元素，例如从student对象中取name属性
3. flatMap(Function f) 将所有流中的元素并到一起连接成一个流
4. peek(Consumer c) 获取流中元素，操作流中元素，与foreach不同的是不会截断流，可继续操作流
5. distinct() 通过流所生成元素的equals和hashCode去重
6. limit(long val) 截断流，取流中前val个元素
7. sorted(Comparator) 产生一个新流，按照比较器规则排序
8. sorted() 产生一个新流，按照自然顺序排序

### 4.4.3 终结操作


