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

![image-20201221104504502](/images/2020122101.png)

这个类型转换的错误在运行期才报出来，显然是不符合我们的预期，应该在编译器就应该解决掉。

设置List容器的类型，就可以解决上面的问题

![image-20201221104807273](/images/2020122102.png)



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

![image-20201221111507121](/images/2020122103.png)



**注意：**

- 泛型类在创建对象的时候，没有指定类型，将按照Object类型来操作

- 泛型参数不支持基本类型，只能是类类型

  ![image-20201221113906058](/images/2020122104.png)

- 泛型相关的信息不会进入到运行时阶段

  ![image-20201221114453417](/images/2020122105.png)

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

  ![image-20210102213439918](/images/2020122107.png)

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



## 4.泛型方法

> 泛型方法，是在调用方法的时候指明泛型的具体类型。
>
> 泛型方法既能用在泛型类、泛型接口里，也能用在普通类或者接口里

**语法**

```java
修饰符 <T，E, ...> 返回值类型 方法名(形参列表) { 
	方法体... 
}
```

- public与返回值中间`<T>`非常重要，可以理解为声明此方法为泛型方法。
- 只有声明了`<T>`的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法。
- `<T>`表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T。
- 与泛型类的定义一样，此处`T`可以随便写为任意标识，常见的如`T、E、K、V`等形式的参数常用于表示泛型。

```java
public class GenericFruit {
    class Fruit{
        @Override
        public String toString() {
            return "fruit";
        }
    }

    class Apple extends Fruit{
        @Override
        public String toString() {
            return "apple";
        }
    }

    class Person{
        @Override
        public String toString() {
            return "Person";
        }
    }

    class GenerateTest<T>{
        public void show_1(T t){
            System.out.println(t.toString());
        }

        //在泛型类中声明了一个泛型方法，使用泛型E，这种泛型E可以为任意类型。可以类型与T相同，也可以不同。
        //由于泛型方法在声明的时候会声明泛型<E>，因此即使在泛型类中并未声明泛型，编译器也能够正确识别泛型方法中识别的泛型。
        public <E> void show_3(E t){
            System.out.println(t.toString());
        }

        //在泛型类中声明了一个泛型方法，使用泛型T，注意这个T是一种全新的类型，可以与泛型类中声明的T不是同一种类型。
        public <T> void show_2(T t){
            System.out.println(t.toString());
        }
    }

    public static void main(String[] args) {
        Apple apple = new Apple();
        Person person = new Person();

        GenerateTest<Fruit> generateTest = new GenerateTest<Fruit>();
        //apple是Fruit的子类，所以这里可以
        generateTest.show_1(apple);
        //编译器会报错，因为泛型类型实参指定的是Fruit，而传入的实参类是Person
        //generateTest.show_1(person);

        //使用这两个方法都可以成功
        generateTest.show_2(apple);
        generateTest.show_2(person);

        //使用这两个方法也都可以成功
        generateTest.show_3(apple);
        generateTest.show_3(person);
    }
}
```



**泛型方法与可变参数:**

```java
public <E> void print(E... e){
	for (E e1 : e) {
		System.out.println(e);
	}
}
```

泛型方法和可变参数的例子：

```java
public <T> void printMsg( T... args){
    for(T t : args){
        Log.d("泛型测试","t is " + t);
    }
}
```

```
printMsg("111",222,"aaaa","2323.4",55.55);
```



**静态方法与泛型**

静态方法有一种情况需要注意一下，那就是在类中的静态方法使用泛型：**静态方法无法访问类上定义的泛型；如果静态方法操作的引用数据类型不确定的时候，必须要将泛型定义在方法上。**

即：**如果静态方法要使用泛型的话，必须将静态方法也定义成泛型方法** 。

```java
public class StaticGenerator<T> {
    ....
    ....
    /**
     * 如果在类中定义使用泛型的静态方法，需要添加额外的泛型声明（将这个方法定义成泛型方法）
     * 即使静态方法要使用泛型类中已经声明过的泛型也不可以。
     * 如：public static void show(T t){..},此时编译器会提示错误信息：
          "StaticGenerator cannot be refrenced from static context"
     */
    public static <T> void show(T t){

    }
}
```



**泛型方法总结**

泛型方法能使方法独立于类而产生变化，以下是一个基本的指导原则：

> 无论何时，如果你能做到，你就该尽量使用泛型方法。也就是说，如果使用泛型方法将整个类泛型化，那么就应该使用泛型方法。另外对于一个static的方法而已，无法访问泛型类型的参数。所以如果static方法要使用泛型能力，就必须使其成为泛型方法。



## 5.泛型通配符

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

## 6.类型擦除

**概念:**

泛型是Java 1.5版本才引进的概念，在这之前是没有泛型的，但是泛型代码能够很好地和之前版本的代码兼容。那是因为，泛型信息只存在于代码编译阶段，在进入JVM之前，与泛型相关的信息会被擦除掉，我们称之为–类型擦除。

**分类：**

- 无限制类型擦除
  ![image-20210103145023407](/images/2020122108.png)
- 有限制类型擦除
  ![image-20210103145059877](/images/2020122109.png)擦除方法中类型定义的参数
  ![在这里插入图片描述](/images/2020122110.png)
- 桥接方法
  ![在这里插入图片描述](/images/2020122111.png)

## 7.泛型和数组

**泛型数组的创建**

- 可以声明带泛型的数组引用，但是不能直接创建带泛型的数组对象

```java
ArrayList<String>[] listArr = new ArrayList<5>(); //会报错
```

会报错

```java
ArrayList[] list = new ArrayList[5];
ArrayList<String>[] listArr = list;
或者
ArrayList<String>[] listArr = new ArrayList[5];
```

不会报错

- 可以通过java.lang.reflect.Array的newInstance(Class,int)创建T[]数组

```java
public class Fruit<T> {
    private T[] array;

    public Fruit(Class<T> clz, int length){
        //通过Array.newInstance创建泛型数组
        array = (T[])Array.newInstance(clz, length);
    }
}
```

## 8.泛型和反射

- 反射常用的泛型类
  Class
  Constructor

```java
public class Person {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
/**
 * 泛型与反射
 */
public class Test11 {
	public static void main(String[] args) throws Exception {
	     Class<Person> personClass = Person.class;
	     Constructor<Person> constructor = personClass.getConstructor();
	     Person person = constructor.newInstance();
	 }
}
```