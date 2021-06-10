---
title: 消灭if-else的几种方案
date: 2021-05-30 09:49:16
tags:
---

[TOC]

<!--more-->

# 方案 1：提前 return，去除不必要的 else

如果 if-else 代码块包含 return 语句，可以考虑通过提前 return，把多余 else 干掉，使代码更加优雅。

优化前：

```
if(condition){ 
 	//doSomething 
 }else{ 
 	return ; 
} 
```

优化后：

```
if（!condition）{ 
 return ; 
} 
//doSomething 
```

# 方案 2：使用条件三目运算符

使用条件三目运算符可以简化某些 if-else，使代码更加简洁，更具有可读性。

优化前：

```
int price ; 
if(condition){ 
 price = 80; 
}else{ 
 price = 100; 
} 
```

优化后：

```
int price = condition ? 80 : 100; 
```

# 方案 3：使用枚举

在某些时候，使用枚举也可以优化 if-else 逻辑分支，按个人理解，它也可以看作一种表驱动方法。

优化前：

```
String OrderStatusDes; 
if(orderStatus==0){ 
 OrderStatusDes ="订单未支付"; 
}else if(OrderStatus==1){ 
 OrderStatusDes ="订单已支付"; 
}else if(OrderStatus==2){ 
 OrderStatusDes ="已发货"; 
} 
```

优化后：（先定义一个枚举）

```
public enum OrderStatusEnum { 
 UN_PAID(0,"订单未支付"),PAIDED(1,"订单已支付"),SENDED(2,"已发货"),; 
 private int index; 
 private String desc; 
 public int getIndex() { 
  return index; 
 } 
 public String getDesc() { 
  return desc; 
 } 
 OrderStatusEnum(int index, String desc){ 
  this.index = index; 
  this.desc =desc; 
 } 
 OrderStatusEnum of(int orderStatus) { 
  for (OrderStatusEnum temp : OrderStatusEnum.values()) { 
   if (temp.getIndex() == orderStatus) { 
    return temp; 
   } 
  } 
  return null; 
 } 
} 
```

有了枚举之后，以上 if-else 逻辑分支，可以优化为一行代码：

```
String OrderStatusDes = OrderStatusEnum.0f(orderStatus).getDesc(); 
```



# 方案 4：合并条件表达式

如果有一系列条件返回一样的结果，可以将它们合并为一个条件表达式，让逻辑更加清晰。

优化前：

```
double getVipDiscount() { 
  if(age<18){ 
   return 0.8; 
  } 
  if("深圳".equals(city)){ 
   return 0.8; 
  } 
  if(isStudent){ 
   return 0.8; 
  } 
  //do somethig 
 } 
```

优化后：

```
double getVipDiscount(){ 
  if(age<18|| "深圳".equals(city)||isStudent){ 
   return 0.8; 
  } 
  //doSomthing 
 } 
```

# 方案 5：使用 Optional

有时候 if-else 比较多，是因为非空判断导致的，这时候你可以使用 java8 的 Optional 进行优化。

优化前：

```
String str = "jay@huaxiao"; 
if (str != null) { 
 System.out.println(str); 
} else { 
 System.out.println("Null"); 
} 
```

优化后：

```
Optional<String> strOptional = Optional.of("jay@huaxiao"); 
strOptional.ifPresentOrElse(System.out::println, () -> System.out.println("Null")); 
```



# 方案 6：表驱动法

表驱动法，又称之为表驱动、表驱动方法。表驱动方法是一种使你可以在表中查找信息，而不必用很多的逻辑语句（if 或 case）来把它们找出来的方法。

以下的 demo，把 map 抽象成表，在 map 中查找信息，而省去不必要的逻辑语句。

优化前：

```
if (param.equals(value1)) { 
 doAction1(someParams); 
} else if (param.equals(value2)) { 
 doAction2(someParams); 
} else if (param.equals(value3)) { 
 doAction3(someParams); 
} 
// ... 
```

优化后：

```
Map<?, Function<?> action> actionMappings = new HashMap<>(); // 这里泛型 ? 是为方便演示，实际可替换为你需要的类型 
// 初始化 
actionMappings.put(value1, (someParams) -> { doAction1(someParams)}); 
actionMappings.put(value2, (someParams) -> { doAction2(someParams)}); 
actionMappings.put(value3, (someParams) -> { doAction3(someParams)}); 
// 省略多余逻辑语句 
actionMappings.get(param).apply(someParams); 
```

# 方案 7：优化逻辑结构，让正常流程走主干

优化前：

```
public double getAdjustedCapital(){ 
 if(_capital <= 0.0 ){ 
  return 0.0; 
 } 
 if(_intRate > 0 && _duration >0){ 
  return (_income / _duration) *ADJ_FACTOR; 
 } 
 return 0.0; 
} 
```

优化后：

```
public double getAdjustedCapital(){ 
 if(_capital <= 0.0 ){ 
  return 0.0; 
 } 
 if(_intRate <= 0 || _duration <= 0){ 
  return 0.0; 
 } 
 return (_income / _duration) *ADJ_FACTOR; 
 } 
```



将条件反转使异常情况先退出，让正常流程维持在主干流程，可以让代码结构更加清晰。

# 方案 8：策略模式+工厂方法消除 if else

假设需求为，根据不同勋章类型，处理相对应的勋章服务，优化前有以下代码：



首先，我们把每个条件逻辑代码块，抽象成一个公共的接口，可以得到以下代码：

我们根据每个逻辑条件，定义相对应的策略实现类，可得以下代码：

```
//守护勋章策略实现类 
public class GuardMedalServiceImpl implements IMedalService { 
 @Override 
 public void showMedal() { 
  System.out.println("展示守护勋章"); 
 } 
} 
//嘉宾勋章策略实现类 
public class GuestMedalServiceImpl implements IMedalService { 
 @Override 
 public void showMedal() { 
  System.out.println("嘉宾勋章"); 
 } 
} 
//VIP勋章策略实现类 
public class VipMedalServiceImpl implements IMedalService { 
 @Override 
 public void showMedal() { 
  System.out.println("会员勋章"); 
 } 
} 
```



接下来，我们再定义策略工厂类，用来管理这些勋章实现策略类，如下：

```
//勋章服务工产类 
public class MedalServicesFactory { 
 private static final Map<String, IMedalService> map = new HashMap<>(); 
 static { 
  map.put("guard", new GuardMedalServiceImpl()); 
  map.put("vip", new VipMedalServiceImpl()); 
  map.put("guest", new GuestMedalServiceImpl()); 
 } 
 public static IMedalService getMedalService(String medalType) { 
  return map.get(medalType); 
 } 
} 
```



使用了策略+工厂模式之后，代码变得简洁多了，如下：

```
public class Test { 
 public static void main(String[] args) { 
  String medalType = "guest"; 
  IMedalService medalService = MedalServicesFactory.getMedalService(medalType); 
  medalService.showMedal(); 
 } 
} 
```

# 方案9：规则引擎Drools

> https://tech.meituan.com/2017/06/09/maze-framework.html