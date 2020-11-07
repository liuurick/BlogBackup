---
title: SpringBoot实现邮件发送功能
date: 2020-08-13 14:34:43
tags: 邮件
categories: SpringBoot
---
### 

<!--more-->

### 基本知识

#### POP3协议是什么？

POP3是Post Office Protocol 3的简称，即邮局协议的第3个版本,它规定怎样将个人计算机连接到Internet的邮件服务器和下载电子邮件的电子协议。它是因特网电子邮件的第一个离线协议标准,POP3允许用户从服务器上把邮件存储到本地主机（即自己的计算机）上,同时删除保存在邮件服务器上的邮件，而POP3服务器则是遵循POP3协议的接收邮件服务器，用来接收电子邮件的。


#### SMTP协议是什么？

SMTP 的全称是Simple Mail Transfer Protocol，即简单邮件传输协议。它是一组用于从源地址到目的地址传输邮件的规范，通过它来控制邮件的中转方式。SMTP 协议属于 TCP/IP 协议簇，它帮助每台计算机在发送或中转信件时找到下一个目的地。SMTP 服务器就是遵循 SMTP 协议的发送邮件服务器。SMTP 认证，简单地说就是要求必须在提供了账户名和密码之后才可以登录 SMTP 服务器，这就使得那些垃圾邮件的散播者无可乘之机。增加 SMTP 认证的目的是为了使用户避免受到垃圾邮件的侵扰。

#### IMAP协议是什么？

IMAP全称是Internet Mail Access Protocol，即交互式邮件存取协议，它是跟POP3类似邮件访问标准协议之一。不同的是，开启了IMAP后，您在电子邮件客户端收取的邮件仍然保留在服务器上，同时在客户端上的操作都会反馈到服务器上，如：删除邮件，标记已读等，服务器上的邮件也会做相应的动作。所以无论从浏览器登录邮箱或者客户端软件登录邮箱，看到的邮件以及状态都是一致的。

#### IMAP和POP3协议有什么不同呢？
POP3协议允许电子邮件客户端下载服务器上的邮件，但是在客户端的操作（如移动邮件、标记已读等），不会反馈到服务器上，比如通过客户端收取了邮箱中的3封邮件并移动到其他文件夹，邮箱服务器上的这些邮件是没有同时被移动的 。

而IMAP提供webmail 与电子邮件客户端之间的双向通信，客户端的操作都会反馈到服务器上，对邮件进行的操作，服务器上的邮件也会做相应的动作。

同时，IMAP像POP3那样提供了方便的邮件下载服务，让用户能进行离线阅读。IMAP提供的摘要浏览功能可以让你在阅读完所有的邮件到达时间、主题、发件人、大小等信息后才作出是否下载的决定。此外，IMAP 更好地支持了从多个不同设备中随时访问新邮件。
![](/images/2020081401.png)
总之，IMAP 整体上为用户带来更为便捷和可靠的体验。POP3 更易丢失邮件或多次下载相同的邮件，但 IMAP 通过邮件客户端与webmail 之间的双向同步功能很好地避免了这些问题。



### 功能实现

对于邮箱功能，首先需要开启`POP3/SMTP服务`获取邮箱授权码。

在邮箱主页->设置->账户中可以看到

![](/images/2020081402.png)

开启之后会获取一个授权码，用来第三方验证，具体的可以看官方教程。[官方教程](https://service.mail.qq.com/cgi-bin/help?subtype=1&&no=1001256&&id=28 )



#### 小试牛刀

创建一个springboot项目，引入下方依赖。

```
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-mail</artifactId>
	</dependency>
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-web</artifactId>
	</dependency>
```

 然后修改application.properties 配置信息，这里我使用的是application.yml文件。
```
spring:
  mail:
    host: smtp.qq.com
    port: 587
    username: 2826803629@qq.com
    password: xxxxwwnddfh
    default-encoding: UTF-8
    properties:
      mail:
        smtp:
          socketFactoryClass: javax.net.ssl.SSLsocketFactory
        debug: true
```

配置完成之后进行测试：

```java
	@Autowired
    JavaMailSender javaMailSender;
    @Test
    void contextLoads() {
        SimpleMailMessage msg = new SimpleMailMessage();
        //邮件主题
        msg.setSubject("测试邮件");
        //邮件内容
        msg.setText("hhhhhhh");
        //邮件发送者
        msg.setFrom("xxx@qq.com");
        //邮件接受者
        msg.setTo("xxx@qq.com");
        //发送邮件
        javaMailSender.send(msg);
    }
```





### 常见错误

有些同学可能会遇到下面的错误，这是授权码的问题

```bash
javax.mail.AuthenticationFailedException: 535 Login Fail. Please enter your authorization code to login. More information in http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256
```

重新开启POP3/SMTP服务即可。