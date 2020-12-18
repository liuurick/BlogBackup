---
title: SpringBoot项目测试模板
date: 2020-12-15 20:46:19
tags: 测试
categories: SpringBoot
---

在Application类中写好一个模板，其他测试类继承即可

<!--more-->

```
@SpringBootTest
@RunWith(SpringRunner.class)
public class XXXApplicationTests {

}
```