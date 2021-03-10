---
title: Java定时任务调度工具
date: 2021-02-02 09:49:17
tags: [Timer,Quartz]
---

[TOC]

- Timer
- Quartz

<!--more-->

# 1 概述

> 什么是定时任务调度：基于**给定的时间点**，**给定的时间间隔**或者**给定的执行次数**自动执行的任务
>



常用的调度工具

- Timer：JDK提供

- Quartz：第三方提供



# 2 Timer

## 2.1 定义

> 有且仅有一个后台线程对多个业务线程进行定时定频率的调度

## 2.2 主要构件

Timer定时调用TimerTask

- Timer：可以笼统的理解为后台执行的线程

- TimerTask：可以理解为业务线程

## 2.3 Timer工具类

![image-20210202101609478](/images/2021020201.png)



### demo

MyTimerTask:

```java
public class MyTimerTask extends TimerTask {
    private String name;

    public MyTimerTask(String inputName) {
        name = inputName;
    }

    @Override
    public void run() {
        //打印当前name的内容
        System.out.println("Current exec name is" + name);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

MyTimer:

```java
public class MyTimer {
    public static void main(String[] args) {
        //1.创建Timer实例
        Timer timer = new Timer();
        //2.创建TimerTask实例
        MyTimerTask myTimerTask = new MyTimerTask("NO.1");
        //3.通过timer定时定频率调用myTimerTask的业务逻辑
        //第一次执行是在当前时间的2s之后，之后每隔1s执行一次
        timer.schedule(myTimerTask,2000L,1000L);
    }
}
```



## 2.4 Timer的定时调度函数

- schedule的四种用法
- scheduleAtFixedRate的两种用法



### 2.4.1 schedule的四种用法

1.schedule(task,time)

```
参数：
	task：所要安排的任务
	time：执行任务的时间
作用：在时间等于或超过time的时间执行且仅执行一次task
```



2.schedule(task,time,period)

```
参数：
	task：所要安排的任务
	time：首次执行任务的时间
	period：执行一次task的时间间隔，单位是毫秒
作用：时间等于或超过time时首次执行task，之后每隔period毫秒重复执行一次task
```



3.schedule(task,delay)

```
参数：
	task：所要安排的任务
	delay: 执行任务前的延迟时间，单位是毫秒
作用：等待delay毫秒后执行且执行一次task
```



4.scheduleAtFixedRate(task,delay,period)

```
参数：
	task：所要安排的任务
	delay：执行任务前的延迟时间，单位是毫秒
	period：执行一次task的时间间隔，单位是毫秒
作用：时间等于或超过time时首次执行task，之后每隔period毫秒重复执行一次task
```



### 2.4.2 scheduleAtFixedRate的两种用法

1.scheduleAtFixedRate(task,time,period)

```
参数：
	task：所要安排的任务
	time：首次执行任务的时间
	period：执行一次task的时间间隔，单位是毫秒
作用：时间等于或超过time时首次执行task，之后每隔period毫秒重复执行一次task
```



2.scheduleAtFixedRate(task,delay,period)

```
参数：
	task：所要安排的任务
	delay：执行任务前的延迟时间，单位是毫秒
	period：执行一次task的时间间隔，单位是毫秒

作用：等待delay毫秒后首次执行task，之后每隔period毫秒重复执行一次task
```



### 2.4.3 其他重要函数

- TimerTask的canal()，scheduledExecutionTime()


- Timer()的canal（），purge()



1.TimerTask的canal()，scheduledExecutionTime()

canal()：取消当前TimerTask里的任务

scheduledExecutionTime：返回此任务最近实际执行的已安排执行的时间，为long型



2.Timer()的canal（），purge()

canal()：终止此计时器，丢弃当前已安排的任务。 

purge()：从该计时器的任务队列中删除所有取消的任务。 返回值：从队列中移除的任务数



### 2.4.4 schedule与scheduleAtFixedRate的区别

> Java Timer：schedule和scheduleAtFixedRate有何不同：https://www.ewdna.com/2011/12/java-timerschedulescheduleatfixedrate.html

### 2.4.5 Timer函数的综合应用

模拟两个机器人的定时行为

```
 实现两个机器人
 第一个机器人会每隔两秒打印最近一次计划执行时间、执行的内容。
 第二个机器人模拟往桶里灌水，直到桶里的水满为止。
```

灌水机器人：

![image-20210202203154572](/images/2021020202.png)



跳舞机器人：

![image-20210202203211900](/images/2021020203.png)

```java
public class DancingRobot extends TimerTask {

    @Override
    public void run() {
        SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println("Schedule exec time is :" + sf.format(scheduledExecutionTime()));
        System.out.println("Dancing happly");
    }
}
		
		public class WaterRobot extends TimerTask {

    //最大容量为5
    private Integer bucketCapacity = 0;
    private Timer timer;

    public WaterRobot(Timer timer){
        this.timer = timer;
    }

    @Override
    public void run() {
        //灌水直至桶满
        if(bucketCapacity < 5){
            System.out.println("Add 1L water");
            bucketCapacity++ ;
        }else{
            //水满之后停止执行
            cancel();
            System.out.println("The waterRobot has been aborted");
            //等待两秒钟,终止timer里面所有内容
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            timer.cancel();
        }
    }
}

		
		public class Main{
    public static void main(String[] args) {
        Timer timer = new Timer();
        //获取当前时间
        Calendar calendar = Calendar.getInstance();
        SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println("current time is :" + sf.format(calendar.getTime());

        DancingRobot dr = new DancingRobot();
        WaterRobot wr = new WaterRobot(timer);
        timer.schedule(dr,calender.getTime(),2000);
        timer.scheduleAtFixedRate(wr,calender.getTime(),1000);
    }
}
```



## 2.5 Timer的缺陷

- 管理并发任务的缺陷
- 当任务抛出异常时的缺陷

## 2.6 Timer的使用禁区

-  对时效性要求较高的多任务并发作业
-  对复杂任务的调度(抛出异常)不灵活 



# 3 Quartz

## 3.1 初识Quartz

### 3.1.1 Quartz概要

```
OpenSymphony提供的强大的开源任务调度框架
官网：http://www.quartz-scheduler.org
纯Java实现，精细控制排程
```

### 3.1.2 Quartz特点

- 强大的调度功能
- 灵活的应用方式
- 分布式和集群能力



### 3.1.3 Quartz的设计模式

- Builder模式
- Factory模式
- 组件模式
- 链式写法



### 3.1.4 Quartz三个核心概念

- 调度器
- 任务
- 触发器



### 3.1.5 Quartz体系结构

```
JobDetail
trigger
    SimpleTrigger
    CronTrigger
scheduler
    start
    stop
    pause
    resume
```

示意图

![clipboard.png](/images/2021020204.png)

Quartz重要组成

```
Job
JobDetail
JobBuilder
JobStore
Trigger
TriggerBuilder
ThreadPool
Scheduler
Calendar：一个Trigger可以和多个Calendar关联，以排除或包含某些时间点
监听器
    JobListener
    TriggerListener
    SchedulerListener
```

## 3.2 Quartz详解

### 3.2.1 第一个Quartz程序

1.建立Maven项目工程

2.引入Quartz包

3.编写第一个Quartz任务

```
让任务每隔两秒打印一次hellworld
```



1.编写任务类

```
public class HelloJob implements Job {

    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        //打印当前的时间
        Calendar calendar = Calendar.getInstance();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(sdf.format(calendar.getTime()));

        //业务逻辑
        System.out.println("Hello Job");

    }
}
```

2.编写任务调度类

```java
public class HelloScheduler {
    public static void main(String[] args) throws SchedulerException {
        //创建一个JobDetail实例，将该实例与HelloJob Class绑定
        JobDetail jobDetail = JobBuilder.newJob(HelloJob.class)
                .withIdentity("myjob","group1")
                .build();
        //创建一个Trigger实例，定义该job立即执行，并且每隔两秒钟重复执行一次，直到永远
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity("myTrigger","group1")
                .startNow().withSchedule(SimpleScheduleBuilder
                        .simpleSchedule().
                                withIntervalInSeconds(2).repeatForever()).build();
        //创建Scheduler实例
        SchedulerFactory sfact = new StdSchedulerFactory();
        Scheduler scheduler = sfact.getScheduler();
        scheduler.start();

        //打印当前的时间
        java.util.Calendar calendar = Calendar.getInstance();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(sdf.format(calendar.getTime()));

        scheduler.scheduleJob(jobDetail,trigger);

    }
}
```

### 3.2.2 浅谈Job&JobDetail

**Job定义**

实现业务逻辑的任务接口



**浅谈Job**

Job接口非常容易实现，只有一个`execute`方法，类似TimerTask的run方法，在里面编写业务逻辑

```
Job接口源码
public interface Job {
   void execute(JobExecutionContext context) throws JobExecutionException;
}
```



**Job实例在Quartz中的生命周期**

​		每次调度器执行job时，它在调用execute方法前会创建一个新的job实例

​		当调用完成后，关联的job对象实例会被释放，释放的实例会被垃圾回收机制回收。



**浅谈JobDetail**

​		JobDetail为Job实例提供了许多设置属性，以及JobDetailMap成员变量属性，它用来存储特定Job实例的状态信息，调度器需要借助JobDetail对象来添加Job实例。



**JobDetail属性**

```
name：任务名称
group：任务所属组
jobClass：任务实现类
jobDataMap：传参的作用
```

### 3.2.3 浅谈JobExecutionContext&JobDataMap

**JobExecutionContext是什么**

- 当Scheduler调用一个Job，就会将JobExecutionContext传递给Job的execute()方法；
- Job能通过JobExecutionContext对象访问到Quartz运行时候的环境以及Job本身的明细数据。



**JobDataMap是什么**

- 在进行任务调度时JobDataMap存储在JobExecutionContext中，非常方便获取
- JobDataMap可以用来装载任务可序列化的数据对象，当job实例对象被执行时这些参数对象会传递给它
- JobDataMap实现了JDK的Map接口，并且添加了一些非常方便的方法用来存取基本数据类型



**获取JobDataMap的两种方式**

1. 从Map中直接获取
2. Job实现类中添加setter方法对应JobDataMap的键值
   （Quartz框架默认的JobFactory实现类在初始化job实例对象时会自动地调用这些setter方式）



### 3.2.4 浅谈Trigger

Trigger是什么

```
Quartz中的触发器用来告诉调度程序作业什么时候触发
即Trigger对象时用来触发执行Job的
```

Quartz框架中的Trigger示意图

![clipboard.png](/images/2021020205.png)

触发器通用属性

```
JobKey：表示job实例的标识，触发器被触发时，该指定的job实例会执行
StartTime：表示触发器的时间表首次被触发的时间，它的值的类型是Java.util.Date
EndTime：指定触发器的不再被触发的时间，它的值的类型是Java.util.Date
```

### 3.2.5 SimpleTrigger

SimpleTrigger的作用

```
在一个指定时间段内执行一次作业任务
或是在指定的时间间隔内多次执行作业任务
```

需要注意的点

```
重复次数可以为0，正整数或是SimpleTrigger.REPEAT_INDEFINITELY常量值
重复执行间隔必须为0或长整数
一旦被指定了endTime参数，那么它会覆盖重复次数参数的效果
```

### 3.2.6 CronTrigger

CronTrigger的作用

```
基于日历的作业调度器，而不是像SimpleTrigger那样精确指定间隔时间，比SimpleTrigger更常用
```

Cron表达式

```
用于配置CronTrigger实例
是由7个子表达式组成的字符串，描述了时间表的详细信息
格式：[秒][分][小时][日][月][周][年]
```

Cron表达式特殊字符意义对应表

![clipboard.png](/images/2021020206.png)

特殊符号解释

![clipboard.png](/images/2021020207.png)

Cron表达式举例

![clipboard.png](/images/2021020208.png)

Cron表达式小提示

```
L和W可以一组合使用
周字段英文字母不区分大小写即MON与mon相同
利用工具，在线生成
```

### 3.2.7 浅谈Scheduler

Scheduler工厂模式

```
所有的Scheduler实例应该由SchedulerFactory来创建
```

SchedulerFactory类图

![clipboard.png](/images/2021020209.png)

回顾Quartz三个核心概念

```
调度器
任务
触发器
```

示意图

![clipboard.png](/images/2021020210.png)

Scheduler的创建方式

![clipboard.png](/images/2021020211.png)

StdSchedulerFactory

```
使用一组参数（Java.util.Properties）来创建和初始化Quartz调度器
配置参数一般存储在quartz.properties中
调用getScheduler方法就能创建和初始化调度器对象
```

Scheduler的主要函数

```
// 绑定 jobDetail 和 trigger，将它注册进 Scheduler 当中
Date scheduleJob(JobDetail jobDetail, Trigger trigger)
// 启动 Scheduler
void start()
// 暂停 Scheduler
void standby()
// 关闭 Scheduler
void shutdown()
```

### 3.2.8 QuartzProperties文件

quartz.properties组成部分

```
调度器属性
线程池属性
作业存储设置
插件配置
```

调度器属性

![clipboard.png](/images/2021020212.png)

线程池属性

```
threadCount：工作线程数量
threadPriority：工作线程优先级
org.quartz.threadPool.class：配置线程池实现类
```

作业存储设置

```
描述了在调度器实例的生命周期中，Job和Trigger信息是如何被存储的
```

插件配置

```
满足特定需求用到的Quartz插件的配置
```

## 3.3 Spring、Quartz大合体

- 搭建工程

- Quartz和Spring大合体



