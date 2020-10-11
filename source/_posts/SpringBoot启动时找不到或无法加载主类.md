---
title: SpringBoot启动时找不到或无法加载主类
date: 2020-10-06 22:48:11
tags: SpringBoot
categories: SpringBoot错误
---
在启动SpringBoot项目时，控制台页面突然报出下列错误：
>找不到或无法加载主类 com.xxx.xxxx.xxxxApplication

一开始以为是项目打包的问题，于是启动`mvn clean install`。。。无果
只得硬着头皮一步步排查，最后是在workspace.xml文件中找到问题所在,也就是SPRING_BOOT_MAIN_CLASS的value并没有设置正确
```
 <component name="RunManager" selected="Spring Boot.DianpingApplication">
    <configuration name="DianpingApplication" type="SpringBootApplicationConfigurationType" factoryName="Spring Boot">
      <module name="dianping" />
      <option name="SPRING_BOOT_MAIN_CLASS" value="com.liuurick.dianping.DianpingApplication" />
      <method v="2">
        <option name="Make" enabled="true" />
      </method>
    </configuration>
  </component>

```