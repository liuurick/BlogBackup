---
title: SpringBoot整合Undertow
date: 2020-12-30 22:15:33
tags: Undertow
categories: [运维知识,应用服务器]
---

[TOC]

<!--more-->

# 1.maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

# 2.配置Undertow

>Undertow基本配置:https://www.pianshen.com/article/7390262599/

这里以application.properties示例：

```
# 是否打开 undertow 日志，默认为 false
server.undertow.accesslog.enabled=false
# 设置访问日志所在目录
server.undertow.accesslog.dir=logs
# 指定工作者线程的 I/0 线程数，默认为 2 或者 CPU 的个数
server.undertow.io-threads=
# 指定工作者线程个数，默认为 I/O 线程个数的 8 倍
server.undertow.worker-threads=
# 设置 HTTP POST 内容的最大长度，默认不做限制
server.undertow.max-http-post-size=0
```





# 3.使用 Undertow 监听多个端口示例

```java
@Bean
public UndertowEmbeddedServletContainerFactory embeddedServletContainerFactory() {
    UndertowEmbeddedServletContainerFactory factory = new UndertowEmbeddedServletContainerFactory();
    factory.addBuilderCustomizers(new UndertowBuilderCustomizer() {

        @Override
        public void customize(Builder builder) {
            builder.addHttpListener(8080, "0.0.0.0");
        }

    });
    return factory;
}
```

