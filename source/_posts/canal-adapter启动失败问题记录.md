---
title: canal-adapter启动失败问题记录
date: 2020-12-14 10:32:44
tags: canal
categories: canal
---

> canal:https://github.com/alibaba/canal

<!--more-->

# 版本信息

canal-1.4 

elasticsearch-7.7.1

jdk1.8

mysql-5.7.30



# 报错信息

```
2020-12-13 09:39:06.006 [main] INFO  o.s.c.annotation.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@37efd131: startup date [Sun Dec 13 09:39:05 GMT+08:00 2020]; root of context hierarchy
2020-12-13 09:39:06.521 [main] INFO  o.s.c.s.PostProcessorRegistrationDelegate$BeanPostProcessorChecker - Bean 'configurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$$EnhancerBySpringCGLIB$$d7bfd1f6] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2020-12-13 09:39:06.852 [main] INFO  c.a.otter.canal.adapter.launcher.CanalAdapterApplication - No active profile set, falling back to default profiles: default
2020-12-13 09:39:06.884 [main] INFO  o.s.b.w.s.c.AnnotationConfigServletWebServerApplicationContext - Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@796d3c9f: startup date [Sun Dec 13 09:39:06 GMT+08:00 2020]; parent: org.springframework.context.annotation.AnnotationConfigApplicationContext@37efd131
2020-12-13 09:39:07.640 [main] INFO  org.springframework.cloud.context.scope.GenericScope - BeanFactory id=ba9c0aec-0105-3f1f-b89e-e85c68567039
2020-12-13 09:39:07.724 [main] INFO  o.s.c.s.PostProcessorRegistrationDelegate$BeanPostProcessorChecker - Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$$EnhancerBySpringCGLIB$$d7bfd1f6] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2020-12-13 09:39:09.358 [main] INFO  o.s.boot.web.embedded.tomcat.TomcatWebServer - Tomcat initialized with port(s): 8081 (http)
2020-12-13 09:39:09.382 [main] INFO  org.apache.coyote.http11.Http11NioProtocol - Initializing ProtocolHandler ["http-nio-8081"]
2020-12-13 09:39:09.402 [main] INFO  org.apache.catalina.core.StandardService - Starting service [Tomcat]
2020-12-13 09:39:09.404 [main] INFO  org.apache.catalina.core.StandardEngine - Starting Servlet Engine: Apache Tomcat/8.5.29
2020-12-13 09:39:09.421 [localhost-startStop-1] INFO  org.apache.catalina.core.AprLifecycleListener - The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [C:\Program Files\Java\jdk-11.0.9\bin;C:\WINDOWS\Sun\Java\bin;C:\WINDOWS\system32;C:\WINDOWS;C:\Program Files\Common Files\Oracle\Java\javapath;".;C:\Program Files\Java\jdk-11.0.9\\bin;E:\tomcat\apache-tomcat-8.5.59\bin;";C:\Program Files\VanDyke Software\Clients\;E:\Python\python3\Scripts\;E:\Python\python3\;E:\Java\maven\apache-maven-3.6.3\bin;E:\IDEA\gradle\gradle-5.4.1\bin;E:\app\admin\product\11.2.0\dbhome_1\bin;C:\Program Files (x86)\Intel\iCLS Client\;C:\Program Files\Intel\iCLS Client\;C:\windows\system32;C:\windows;C:\windows\System32\Wbem;C:\windows\System32\WindowsPowerShell\v1.0\;C:\Program Files (x86)\NVIDIA Corporation\PhysX\Common;C:\Program Files (x86)\MySQL\MySQL Server 5.5\bin;C:\;rogram Files (x86)\Intel\Intel(R) Management Engine Components\DAL;C:\Program Files\Intel\Intel(R) Management Engine Components\DAL;C:\Program Files (x86)\Intel\Intel(R) Management Engine Components\IPT;C:\Program Files\Intel\Intel(R) Management Engine Components\IPT;C:\Program Files (x86)\Microsoft SQL Server\90\Tools\binn\;C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem;C:\WINDOWS\System32\WindowsPowerShell\v1.0\;人网站\Git\bin;:\;人网站\Git\cmd;it;cmd;D:\;C:\WINDOWS\System32\OpenSSH\;"E:\tomcat\apache-tomcat-8.5.59\bin;C:\Program Files\Intel\WiFi\bin\;C:\Program Files\Common Files\Intel\WirelessCommon\";E:\Coding\winhadoop\bin;E:\Git\cmd;E:\node\;C:\Program Files\MySQL\MySQL Server 5.7\bin;C:\Program Files (x86)\CADSeePlus;E:\tomcat\apache-tomcat-8.5.34\bin;C:\Program Files\Java\jdk1.8.0_261\bin;C:\Program Files\Java\jdk-11.0.9\bin;E:\Python\python3\Scripts\;E:\Python\python3\;C:\Program Files (x86)\Microsoft Visual Studio\Common\Tools\WinNT;C:\Program Files (x86)\Microsoft Visual Studio\Common\MSDev98\Bin;C:\Program Files (x86)\Microsoft Visual Studio\Common\Tools;C:\Program Files (x86)\Microsoft Visual Studio\VC98\bin;C:\Qt\Qt5.6.2\5.6\msvc2013\bin;C:\Users\admin\AppData\Local\Microsoft\WindowsApps;E:\个人网站\Git\bin;C:\Program Files (x86)\MySQL\MySQL Server 5.5\bin;C:\Users\admin\AppData\Local\GitHubDesktop\bin;"E:\IDEA\gradle\gradle-5.4.1\bin;";C:\Users\admin\AppData\Local\Microsoft\WindowsApps;E:\IDEA\IntelliJ IDEA 2019.3.4\bin;;C:\Users\admin\AppData\Local\Pandoc\;C:\Users\admin\AppData\Roaming\npm;E:\vscode\Microsoft VS Code\bin;C:\Program Files\MySQL\MySQL Server 5.7\bin;E:\WebStorm\WebStorm 2020.2\bin;;C:\Program Files\Bandizip\;C:\Program Files\JetBrains\PyCharm 2020.1.4\bin;;;.]
2020-12-13 09:39:09.650 [localhost-startStop-1] INFO  o.a.catalina.core.ContainerBase.[Tomcat].[localhost].[/] - Initializing Spring embedded WebApplicationContext
2020-12-13 09:39:09.650 [localhost-startStop-1] INFO  org.springframework.web.context.ContextLoader - Root WebApplicationContext: initialization completed in 2766 ms
2020-12-13 09:39:09.902 [localhost-startStop-1] INFO  o.s.boot.web.servlet.ServletRegistrationBean - Servlet dispatcherServlet mapped to [/]
2020-12-13 09:39:09.910 [localhost-startStop-1] INFO  o.s.boot.web.servlet.FilterRegistrationBean - Mapping filter: 'characterEncodingFilter' to: [/*]
2020-12-13 09:39:09.913 [localhost-startStop-1] INFO  o.s.boot.web.servlet.FilterRegistrationBean - Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2020-12-13 09:39:09.914 [localhost-startStop-1] INFO  o.s.boot.web.servlet.FilterRegistrationBean - Mapping filter: 'httpPutFormContentFilter' to: [/*]
2020-12-13 09:39:09.915 [localhost-startStop-1] INFO  o.s.boot.web.servlet.FilterRegistrationBean - Mapping filter: 'requestContextFilter' to: [/*]
2020-12-13 09:39:10.094 [main] WARN  o.s.b.w.s.c.AnnotationConfigServletWebServerApplicationContext - Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'applicationConfigMonitor': Invocation of init method failed; nested exception is java.lang.RuntimeException: Config dir not found.
2020-12-13 09:39:10.098 [main] INFO  org.apache.catalina.core.StandardService - Stopping service [Tomcat]
2020-12-13 09:39:10.321 [localhost-startStop-1] WARN  org.apache.catalina.loader.WebappClassLoaderBase - The web application [ROOT] appears to have started a thread named [Abandoned connection cleanup thread] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.base@11.0.9/java.lang.Object.wait(Native Method)
 java.base@11.0.9/java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:155)
 app//com.mysql.jdbc.AbandonedConnectionCleanupThread.run(AbandonedConnectionCleanupThread.java:43)
2020-12-13 09:39:10.341 [main] INFO  o.s.b.a.logging.ConditionEvaluationReportLoggingListener - 

Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.
2020-12-13 09:39:10.347 [main] ERROR org.springframework.boot.SpringApplication - Application run failed
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'applicationConfigMonitor': Invocation of init method failed; nested exception is java.lang.RuntimeException: Config dir not found.
	at org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor.postProcessBeforeInitialization(InitDestroyAnnotationBeanPostProcessor.java:138)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInitialization(AbstractAutowireCapableBeanFactory.java:422)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1694)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:579)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:501)
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:317)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:315)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:760)
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:869)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:550)
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:140)
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:759)
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:395)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:327)
	at com.alibaba.otter.canal.adapter.launcher.CanalAdapterApplication.main(CanalAdapterApplication.java:19)
Caused by: java.lang.RuntimeException: Config dir not found.
	at com.alibaba.otter.canal.client.adapter.support.Util.getConfDirPath(Util.java:105)
	at com.alibaba.otter.canal.client.adapter.support.Util.getConfDirPath(Util.java:85)
	at com.alibaba.otter.canal.adapter.launcher.monitor.ApplicationConfigMonitor.init(ApplicationConfigMonitor.java:41)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:566)
	at org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor$LifecycleElement.invoke(InitDestroyAnnotationBeanPostProcessor.java:365)
	at org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor$LifecycleMetadata.invokeInitMethods(InitDestroyAnnotationBeanPostProcessor.java:308)
	at org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor.postProcessBeforeInitialization(InitDestroyAnnotationBeanPostProcessor.java:135)
	... 16 common frames omitted

```

