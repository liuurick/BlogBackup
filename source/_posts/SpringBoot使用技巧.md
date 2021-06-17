---
title: SpringBoot使用技巧
date: 2021-04-23 15:56:20
tags: SpringBoot
categories: [后端,Spring家族,SpringBoot]
---

[TOC]

<!--more-->

# 测试模板

在Application类中写好一个模板，其他测试类继承即可

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class XXXApplicationTests {

}
```



# 启动日志美化

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {

    private static final Logger LOG = LoggerFactory.getLogger(EurekaApplication.class);

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(EurekaApplication.class);
        Environment env = app.run(args).getEnvironment();
        LOG.info("启动成功！！");
        LOG.info("Eureka地址: \thttp://127.0.0.1:{}", env.getProperty("server.port"));
    }
}
```

