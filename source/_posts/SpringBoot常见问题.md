---
title: SpringBoot常见问题
date: 2021-06-02 11:17:25
tags: SpringBoot
---

[TOC]

<!--more-->

# LocalDateTime日期类使用问题

> Caused by: java.sql.SQLFeatureNotSupportedException

```
Caused by: java.sql.SQLFeatureNotSupportedException
    at com.alibaba.druid.pool.DruidPooledResultSet.getObject(DruidPooledResultSet.java:1771)
    at org.apache.ibatis.type.LocalDateTimeTypeHandler.getNullableResult(LocalDateTimeTypeHandler.java:38)
    at org.apache.ibatis.type.LocalDateTimeTypeHandler.getNullableResult(LocalDateTimeTypeHandler.java:28)
    at org.apache.ibatis.type.BaseTypeHandler.getResult(BaseTypeHandler.java:81)
org.springframework.dao.InvalidDataAccessApiUsageException: Error attempting to get column 'create_time' from result set.  Cause: java.sql.SQLFeatureNotSupportedException
; null; nested exception is java.sql.SQLFeatureNotSupportedException

    at org.springframework.jdbc.support.SQLExceptionSubclassTranslator.doTranslate(SQLExceptionSubclassTranslator.java:96)
    at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:72)
    at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:81)
    at org.mybatis.spring.MyBatisExceptionTranslator.translateExceptionIfPossible(MyBatisExceptionTranslator.java:73)
    at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:446)
    at com.sun.proxy.$Proxy134.selectOne(Unknown Source)
    at org.mybatis.spring.SqlSessionTemplate.selectOne(SqlSessionTemplate.java:166)
    at com.baomidou.mybatisplus.core.override.MybatisMapperMethod.execute(MybatisMapperMethod.java:89)
    at com.baomidou.mybatisplus.core.override.MybatisMapperProxy.invoke(MybatisMapperProxy.java:62)
    at com.sun.proxy.$Proxy139.getSysLogById(Unknown Source)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
```

- 问题原因是JDK8的LocalDate、LocalTime、LocalDateTime日期类型，druid数据源尚不支持
- [druid issues](https://github.com/alibaba/druid/issues/3230)

## 目前解决办法

1. 更换数据源
   1. 在mybatis-plus生成代码中配置，将日期类型生成为`DateType.ONLY_DATE`，数据库中的日期类型生成为Date

```
// 全局配置
GlobalConfig gc = new GlobalConfig();
gc.setDateType(DateType.ONLY_DATE); // 设置日期类型为Date
```

[mybatis-plus dateType配置](https://mybatis.plus/config/generator-config.html#datetype)

