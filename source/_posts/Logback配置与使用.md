---
title: Logback配置与使用
date: 2018-10-31 18:56:50
tags: Logback
---

[TOC]

# Logback介绍

Logback 分为三个模块：

- Core模块是其他两个模块的基础。
- Classic模块扩展了core模块。Classic模块相当于log4j的显著改进版，Logback-classic 直接实现了 SLF4J API。
- Access访问模块与Servlet容器集成提供通过Http来访问日志的功能。

要引入logback，由于Logback-classic依赖slf4j-api.jar和logback-core.jar，所以要把slf4j-api.jar、logback-core.jar、logback-classic.jar，添加到要引入Logbac日志管理的项目的classpath中。



# 配置模板

> https://www.cnblogs.com/warking/p/5710303.html

在项目中使用的模板

pom依赖：

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
</dependency>
```

配置介绍

- Logger、appender及layout

　　Logger作为日志的记录器，把它关联到应用的对应的context上后，主要用于存放日志对象，也可以定义日志类型、级别。
　　Appender主要用于指定日志输出的目的地，目的地可以是控制台、文件、远程套接字服务器、 MySQL、PostreSQL、 Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等。 
　　Layout 负责把事件转换成字符串，格式化的日志信息的输出。

- logger context

　　各个logger 都被关联到一个 LoggerContext，LoggerContext负责制造logger，也负责以树结构排列各logger。其他所有logger也通过org.slf4j.LoggerFactory 类的静态方法getLogger取得。 getLogger方法以 logger名称为参数。用同一名字调用LoggerFactory.getLogger 方法所得到的永远都是同一个logger对象的引用。

- 有效级别及级别的继承

　　Logger 可以被分配级别。级别包括：TRACE、DEBUG、INFO、WARN 和 ERROR，定义于ch.qos.logback.classic.Level类。如果 logger没有被分配级别，那么它将从有被分配级别的最近的祖先那里继承级别。root logger 默认级别是 DEBUG。

- 打印方法与基本的选择规则

　　打印方法决定记录请求的级别。例如，如果 L 是一个 logger 实例，那么，语句 L.info("..")是一条级别为 INFO的记录语句。记录请求的级别在高于或等于其 logger 的有效级别时被称为被启用，否则，称为被禁用。记录请求级别为 p，其 logger的有效级别为 q，只有则当 p>=q时，该请求才会被执行。
该规则是 logback 的核心。

级别排序为： `TRACE < DEBUG < INFO < WARN < ERROR`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
	<!-- 定义参数常量 -->
	<!-- TRACE<DEBUG<INFO<WARN<ERROR -->
	<!-- logger.trace("msg") logger.debug... -->
	<property name="log.level" value="debug" />
	<property name="log.maxHistory" value="30" />
	<property name="log.filePath" value="${catalina.base}/logs/webapps" />
	<property name="log.pattern"
		value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n" />
	<!-- 控制台设置 -->
	<appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>${log.pattern}</pattern>
		</encoder>
	</appender>
	<!-- DEBUG -->
	<appender name="debugAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<!-- 文件路径 -->
		<file>${log.filePath}/debug.log</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<!-- 文件名称 -->
			<fileNamePattern>${log.filePath}/debug/debug.%d{yyyy-MM-dd}.log.gz
			</fileNamePattern>
			<!-- 文件最大保存历史数量 -->
			<maxHistory>${log.maxHistory}</maxHistory>
		</rollingPolicy>
		<encoder>
			<pattern>${log.pattern}</pattern>
		</encoder>
		<filter class="ch.qos.logback.classic.filter.LevelFilter">
			<level>DEBUG</level>
			<onMatch>ACCEPT</onMatch>
			<onMismatch>DENY</onMismatch>
		</filter>
	</appender>
	<!-- INFO -->
	<appender name="infoAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<!-- 文件路径 -->
		<file>${log.filePath}/info.log</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<!-- 文件名称 -->
			<fileNamePattern>${log.filePath}/info/info.%d{yyyy-MM-dd}.log.gz
			</fileNamePattern>
			<!-- 文件最大保存历史数量 -->
			<maxHistory>${log.maxHistory}</maxHistory>
		</rollingPolicy>
		<encoder>
			<pattern>${log.pattern}</pattern>
		</encoder>
		<filter class="ch.qos.logback.classic.filter.LevelFilter">
			<level>INFO</level>
			<onMatch>ACCEPT</onMatch>
			<onMismatch>DENY</onMismatch>
		</filter>
	</appender>
	<!-- ERROR -->
	<appender name="errorAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<!-- 文件路径 -->
		<file>${log.filePath}/erorr.log</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<!-- 文件名称 -->
			<fileNamePattern>${log.filePath}/error/error.%d{yyyy-MM-dd}.log.gz
			</fileNamePattern>
			<!-- 文件最大保存历史数量 -->
			<maxHistory>${log.maxHistory}</maxHistory>
		</rollingPolicy>
		<encoder>
			<pattern>${log.pattern}</pattern>
		</encoder>
		<filter class="ch.qos.logback.classic.filter.LevelFilter">
			<level>ERROR</level>
			<onMatch>ACCEPT</onMatch>
			<onMismatch>DENY</onMismatch>
		</filter>
	</appender>
	<logger name="com.liuurick.o2o" level="${log.level}" additivity="true">
		<appender-ref ref="debugAppender"/>
		<appender-ref ref="infoAppender"/>
		<appender-ref ref="errorAppender"/>
	</logger>
	<root level="info">
		<appender-ref ref="consoleAppender"/>
	</root>
</configuration>
```

