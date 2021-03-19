---
title: SpringSecurity oauth2权限系统
date: 2021-03-13 12:15:05
tags: SpringSecurity
categories: [后端,分布式/微服务,认证和鉴权]
---

[TOC]

<!--more-->

# 1 Spring Security 概要

## 1.1 Spring Security 概要介绍

​		Spring Security 是基于 Spring 的身份认证（Authentication）和用户授权（Authorization）框架，提供了一 套 Web 应用安全性的完整解决方案。其中核心技术使用了 Servlet 过滤器、IOC 和 AOP 等。 

### 什么是身份认证 

认证（Authentication）：认证解决“我是谁”的问题

​		身份认证指的是用户去访问系统资源时，系统要求验证用户的身份信息，用户身份合法才访问对应资源。 常见的身份认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。 

### 什么是用户授权 

授权（Authorization）：授权解决“我能做什么”的问题

​		当身份认证通过后，去访问系统的资源，系统会判断用户是否拥有访问该资源的权限，只允许访问有权限的 系统资源，没有权限的资源将无法访问，这个过程叫用户授权。 比如 会员管理模块有增删改查功能，有的用户只能进行查询，而有的用户可以进行修改、删除。一般来说， 系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。

## 1.2 Shiro 和 SpringSecurity 的区别

### 1.2.1 Shiro 特点 

1. Shiro 是 Apache 下的项目，相对简单、轻巧，更容易上手使用。 
2. Shiro 权限功能基本都能满足，单点登录都可以实现。且`不用与任何的框架或者容器绑定, 可以独立运行`

### 1.2.2 Spring Security 特点 

1. Spring Security 相对 Shiro 上手更复杂; 
2. Spring Security 功能比 Shiro 更加丰富些; 
3. Spring Security 是 Spring 家族的产品，与 Spring 无缝对接， 社区资源相对比 Shiro 更加丰富； 
4. Spring Security 对 Oauth2 也有支持, Shiro 则需要自己手动实现。而且 Spring Security 的权限细粒度更高

### 1.2.3 如何选择 

1. 如果项目中不是很庞大，没有用到 Spring，那就不要考虑使用 Spring Security ，Shiro 足够满足, 建议使用。 
2. 如果项目使用 Spring 作为基础，配合 Spring Security 做权限更加方便，而 Shiro 需要和 Spring 进行整合开 发。

# 2 Spring Security 开发环境搭建

## 2.1 软件安装

- 安装JDK 
- MySQL 
- Maven开发环境

## 2.2 IntelliJ IDEA的配置与常用快捷键

> [IntelliJ IDEA的配置与常用快捷键](https://liuurick.github.io/2021/01/20/IDEA配置与常用快捷键/)

## 2.3 AdminLTE3 项目框架功能与下载

采用 `AdminLTE` 来完成页面的统一权限管理系统的布局与模板页面。

###  2.3.1 AdminLTE 介绍 

​		AdminLTE是一款建立在 bootstrap 和 jquery 之上的开源前端模板，它提供了一系列响应的、可重复使用的组件， 并内置了多个模板页面；同时自适应多种屏幕分辨率，兼容PC和移动端。 通过AdminLTE，我们可以快速的创建一个响应式的Html5网站。 AdminLTE框架在网页架构与设计上，有很大的辅助作用，尤其是前端架构设计师，用好 AdminLTE 不但美观，而 且可以免去写很大CSS与JS的工作量。 

### 2.3.2 GitHub 下载 

AdminLTE 从 Github 下载

AdminLTE源代码 https://github.com/ColorlibHQ/AdminLTE 

官方指南 ：https://adminlte.io/docs/3.0/



### 2.3.3 AdminLTE3 目录结构与布局-图标Icon

![image](/images/2021031301.png)

```
dist 官方提供的css/img/js...
build 构建项目的源代码
docs 官方文档
pages 官方提供的模板页
plugins 官方提供的第三方前端插件
modules 项目涉及的静态资源，css/img/js...
templates 项目涉及的模板页面
```

### 2.3.4 布局 

布局包括四个主要部分： 

- 整个页面 .wrapper 。包含整个网站的div。 
- 主标题 .main-header 。包含Logo和导航栏。 
- 侧边栏 .sidebar-wrapper 。包含用户面板和侧栏菜单。 
- 内容 .content-wrapper 。包含页面标题和内容。 
- 底部 .main-footer 。包含版权信息

### 2.3.5 图标 icon 

参考：http://fontawesome.dashgame.com/ 

1. 将以下代码粘贴到网页HTML代码的`<head>`部分

   ```
   <link rel="stylesheet" href="https://netdna.bootstrapcdn.com/font-awesome/4.7.0/css/fontawesome.min.css" >
   ```

2.  您可以将Font Awesome图标使用在几乎任何地方，只需要使用CSS前缀 `fa` ，再加上图标名称。 `Font Awesome`是为使用内联元素而设计的。我们通常更喜欢使用 `<li>` *，因为它更简洁。 但实际上使 用 `<span>` 才能更加语义化。 *

```
<i class="fa fa-weixin"></i> 效果
```



# 3 构建 Spring Security 模块化工程

Maven 多模块化构建，课程共 4 个工程

## 3.1 项目目录

| 模块名          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| security-parent | 父模块，pom类型，进行统一的版本管理，聚合管理子模块          |
| security-base   | 基础通用功能管理，如工具类                                   |
| security-core   | 进行安全管理，实现身份认证、验证码认证、手机登录、用户授权等。 |
| security-web    | web业务应用  `thymeleaf dao service controller`              |



Spring Security Maven 模块化工程

- 创建与配置聚合管理 Parent 父工程

- 创建与配置 Base 基础通用工程


- 创建与配置 core 安全管理工程


- 创建与配置web应用工程


- 抽取 AdminLTE3 项目公共代码片段


> 代码：https://github.com/liuurick/security-parent

# 4 Spring Security 身份认证方式和底层源码分析

## 4.1 HttpBasic 身份认证方式 

添加 Spring Security 启动器 在 security-core/pom.xml 中添加 `spring-boot-starter-security` 依赖，如下：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

SpringSecurityConfig 安全控制配置类作为安全控制中心， 用于实现身份认证与授权配置功能 

步骤： 

1.  在 security-core 中创建 com.liuurick.security.config.SpringSecurityConfig 类，继 承 WebSecurityConfigurerAdapter 抽象类 

2. 类上添加注解 @Configuration 标识为配置类、 @EnableWebSecurity 启动 SpringSecurity 过滤器链功能 

3. 重写以下两个方法： 

   **configure(AuthenticationManagerBuilder auth)** 身份认证管理器 ：

   - 认证信息提供方式（用户名、密码、当前用户的资源权限） 
   - 可采用**内存存储方式**，也可能采用**数据库方式**等 。

   **configure(HttpSecurity http)** 资源权限配置（过滤器链） ：

   - 拦截的哪一些资源 
   - 资源所对应的角色权限 
   - 定义认证方式： httpBasic 、 httpForm 
   - 定制登录页面、登录请求地址、错误处理方式 
   - 自定义 spring security 过滤器等

4. 代码如下：

```java
@Configuration
@EnableWebSecurity
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * 认证管理器：
     * 1、认证信息提供方式（用户名、密码、当前用户的资源权限）
     * 2、可采用内存存储方式，也可能采用数据库方式等
     * @param auth
     * @throws Exception
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        super.configure(auth);
    }

    /**
     * 资源权限配置（过滤器链）:
     * 1、被拦截的资源
     * 2、资源所对应的角色权限
     * 3、定义认证方式：httpBasic 、httpForm
     * 4、定制登录页面、登录请求地址、错误处理方式
     * 5、自定义 spring security 过滤器
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic()
                .and()
                .authorizeRequests() // 认证请求
                .anyRequest().authenticated() // 所有进入应用的HTTP请求都要进行认证
        ;
    }
}
```

## 4.2 基于内存存储认证信息和加解密码处理 

### 4.2.1 概述

上面用户名和密码是 Spring Security 为我们提供的，下面基于内存存储存储自定义用户名和密码。



### 4.2.2 实现

1. 在 `configure(AuthenticationManagerBuilder auth)` 方法中指定用户名和密码、权限标识

```java
@Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("liubin")
                .password("123456")
                .authorities("ADMIN");
    }
```

2. 重启 security-web 下的 com.liuurick.WebApplication ，控制台没有自动密码了 
3. 重新访问 http://localhost:8080 ，一样弹出登录窗口，输入用户名和密码
4. 提交登录后还是继续弹出登录页，此时查看 IDEA 控制台，发现报错

![image-20210315211947953](/images/2021031302.png)

### 4.2.3 分析问题 

关注：PasswordEncoder 加密解密统一接口，它有很多的实现类，通过 `ctrl+alt+b` 查看其实现类 

- encode 用于加密明文 
- matches 输入的密码与数据库中的密码对比 
- upgradeEncoding 是否需要编码，一般不需要

![image-20210315212333643](/images/2021031303.png)

- 在 Spring Security 5.0 版本前，加密的 PasswordEncoder 接口默认实现类为 NoOpPasswordEncoder ，这个 是可以不用加密的，直接使用明文密码存储。当前已经标注过时了。 
- 在 Spring Security 5.0 版本后 ，默认实现类改为了 `DelegatingPasswordEncoder `，这个实现类要求我们必须 对加密后存储。

### 4.2.4 解决问题

​		在 SpringSecurityConfig 指定 BCryptPasswordEncoder 加密方式 相同的密码, 在每次加密后的结果都不一样的，因为它每次都会随机生成盐值，会将随机生成的盐加到密码串中每次判断时，通过随机生成的盐反推回加密时的密码串，最终判断是否匹配。

​		加密的最终结果分为两部分，**盐值 + MD5(password+盐值)**, 调用 matches(..) 方法的时候，先从密文中得到 盐值，用该盐值加密明文和最终密文作对比， 这样可以避免有一个密码被破解， 其他相同的密码的帐户都可以破解。因为通过当前机制相同密码生成的密文都不一样。 

​		加密过程（注册）： aaa (盐值) + 123(密码明文) --> 生成密文  --> 最终得到结果盐值密文：aaa.asdlkf 

​		存入数据库校验过程（登录）： aaa (盐值, 数据库中得到) + 123(用户输入密码)> 生成密文 aaa.asdlkf，与数据库对比一 致密码正确。

```java
    @Bean
    public PasswordEncoder passwordEncoder() {
    // 设置默认的加密方式
    return new BCryptPasswordEncoder();
    }
    /**
    * 认证管理器：
    * 1、认证信息提供方式（用户名、密码、当前用户的资源权限）
    * 2、可采用内存存储方式，也可能采用数据库方式等
    * @param auth
    * @throws Exception
    */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    String password = passwordEncoder().encode("123456");
    log.info("加密密码为："+password);

    auth.inMemoryAuthentication()
    .withUser("liubin")
    .password(password)
    .authorities("admin");
    }
```



## 4.3 默认 HttpForm 表单登录方式 

### 4.4.1 概述 

上面弹出登录窗口，体验不太好，希望通过页面的方式展示登录页面，可采用 HttpForm 表单认证来实现这个功 能。 

### 4.4.2 实现流程 

1. 在 SpringSecurityConfig.configure 中通过 `http.formLogin() `方法指定为表单认证方式
2.  重新启动 security-web 
3. 访问 http://localhos:8080 发现不是弹出登录窗口，而是自动请求一个 localhost/login  登录页面。 

**注意**：如果重启项目后，还是一个弹窗，关闭浏览器，再打开浏览器进行访问。还不行就清除浏览器缓存

## 4.4 分析 Spring Security 底层源码认证流程 

![image-20210315215134633](/images/2021031304.png)



# 5 Spring Security 用户名密码身份认证实战

## 5.1 自定义登录页面 

表单认证方式 Spring Security 默认提供了一个 bootstrap 登录页面，如果希望使用自定义的登录页面怎么办？ 

### 5.1.1 指定跳转自定义登录页面的URL 

SpringSecurityConfig.configure(HttpSecurity http) 中使`loginPage("/login/page") `指定前往认证请求

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // http.httpBasic()
        http.formLogin() // 表单认证
        .loginPage("/login/page") // 交给 /login/page 响应认证(登录)页面
        .and()
        .authorizeRequests() // 认证请求
        .anyRequest().authenticated() // 所有进入应用的HTTP请求都要进行认证
    ; 
  }

```

### 5.1.2 实现登录控制器 

在`security-core` 创建 com.liuurick.security.controller.CustomLoginController， 用于实现认证（登录）处理

```java
@Controller
public class CustomLoginController {

    @RequestMapping("/login/page")
    public String loginPage(HttpServletRequest request, HttpServletResponse response){
        return "login";
    }
}
```



### 5.1.3 添加登录页面 

找到 login.html 静态文件添加到 -security-web 模块的 templates 目录下



### 5.1.4 测试 

重启 security-web 下的 com.liuurick.WebApplication 访问 http://localhost:8080 会进入跳转到 http://localhost:8080/login/page ， 并且请求报错 localhost 将您重定向的次数过多

![image-20210316162802532](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210316162802532.png)



### 5.1.5 解决重定向的次数过多 

**分析问题**:

​		因为当前将所有请求都被拦截，拦截后会重写向到认证请求上（默认是 /login ）, 当认证通过后才可以访问，而现 在认证请求改为 /login/page ，它也一样被拦截了，拦截后就重写向回认证请求上（修改的 /login/page 上），这 样反复请求 /login/page 拦截后就报重定向次数过多 。

**解决问题** 

只需要在 SpringSecurityConfig  中放行对应请求资源： 

1. 放行跳转认证请求（前往登录页面请求）

2. 放行静态资源路径，注意重写是： `configure(WebSecurity web)`

```java
	@Override
    public void configure(WebSecurity web) {
        web.ignoring().antMatchers("/dist/**", "/modules/**", "/plugins/**");
    }
```

3. 重启项目, 访问 http://localhost:8080/ ，成功重定向到 http://localhost:8080/login/page 认证页面

## 5.2 登录表单提交处理

### 5.2.1 概述

只要按照 Spring Security 规则进行配置后，Spring Security 会自动帮我们进行身份认证。

### 5.2.2 实现流程 

1. 登录表单默认的 action="/login"  , 通过 loginProcessingUrl("/login/form") 修改为 /login/form 。 
2. 登录表单的用户名参数名默认是 name="username" , 通过 usernameParameter("name") 修改为 name 。 
3. 登录表单的密码参考名默认是 name="password" ，通过 passwordParameter("pwd") 修改为 pwd 。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    // http.httpBasic()
    http.formLogin() // 表单认证
    .loginPage("/login/page") // 交给 /login/page 响应认证(登录)页面
    .loginProcessingUrl("/login/form") // 登录表单提交处理Url, 默认是 /login
    .usernameParameter("name") // 默认用户名的属性名是 username
    .passwordParameter("pwd") // 默认密码的属性名是 password
    .and()
    .authorizeRequests() // 认证请求
    .antMatchers("/login/page").permitAll() // 放行跳转登录请求
    .anyRequest().authenticated() // 所有进入应用的HTTP请求都要进行认证
    ; // 分号`;`不要少了
}
```

4. 检查 security-web 工程中 login.html 页面相关标签属性值是否和上面一致。 注意: html标签上有引入thymeleaf名称空间 

```html
<form th:action="@{/login/form}" th:method="post">
    <div class="form-group has-feedback">
    <input type="text" name="name" class="form-control" required placeholder="用户名">
    <span class="glyphicon glyphicon-envelope form-control-feedback"></span>
    </div>
    <div class="form-group has-feedback">
    <input type="password" name="pwd" class="form-control" required placeholder="密码">
    <span class="glyphicon glyphicon-lock form-control-feedback" ></span>
    </div>
    ...
</form>
```

### 5.2.3 测试 

1. 重启项目 

2. 访问 http://localhost/index 重定向到登录页 

3. 输入错误用户信息，回到登录页面 http://localhost/login/page?error 

4. 输入有效用户信息 , 进入首页。 

5. 如果输入有效用户信息，还是回到登录页，则要禁用 CSRF 攻击。

   -  CSRF（Cross-site request forgery） 跨站请求伪造 

   - 关闭 CSRF 攻击

     ```java
     .and().csrf().disable()
     ```

## 5.3 登录页面回显提示信息 

### 5.3.1 概述 

1. 当提交登录表单数据认证失败后，通过 http://localhost/login/page?error 重定向回登录页，此时地址带有一 个 error 参数，标识认证失败。 并且当用户名或密码错误时，后台会响应提示信息 Bad credentials ，我们要将提示信息在登录页上回显。 
2. 默认情况下，提示信息默认都是英文的，其实是可配置成中文信息。



### 5.3.2 实现页面回显提示信息 

1. http://localhost/login/page?error 有 error 参数就是认证失败 `th:if="${param.error}" `

2. login.html 页面渲染提示信息 `th:text="${session.SPRING_SECURITY_LAST_EXCEPTION?.message}"` 

   >注意 html标签上有引入thymeleaf名称空间 

3. 重启项目，当输入错误用户信息时，页面是否会回显 `Bad credentials`



### 5.3.3 实现中文提示信息

1. 观察 spring-security-core-xxx.jar 下有国际化配置文件 messages_xxx.properties 

2. 默认 ReloadableResourceBundleMessageSource 是加载了 messages.properties 英文配置文件； 
3. 应该手动指定加载 messages_zh_CN.properties 中文配置文件。 
4. 在 security-core 创建 com.security.config.ReloadMessageConfig

```java
@Configuration
public class ReloadMessageConfig {

    @Bean // 加载中文的认证提示信息
    public ReloadableResourceBundleMessageSource messageSource() {
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        //.properties 不要加到后面
        messageSource.setBasename("classpath:org/springframework/security/messages_zh_CN");
        return messageSource;
    }
}
```

### 5.3.4 测试

- 重启项目，当输入错误用户信息时，页面是否会回显 用户名或密码错误

![image-20210319173850288](C:\Users\admin\Desktop\blog\source\images\2021031305.png)

- 用户信息输入正确，会重定向回引发认证的请求中，即首页。

## 5.5 认证相关URL实现可配置 

​		当前在 SpringSecurityConfig 中配置的认证相关URL是写死，这些 URL 根据应用系统的不同，可能需要配置不同的 URL，那我们可以抽取到 application.yml 进行可配置

### 5.5.1 配置 application.yml 

1. 在 security-web\src\main\resources\application.yml 文件中配置如下：

```yml
liubin:
  security:
    authentication:
      loginPage: /login/page # 响应认证(登录)页面的URL
      loginProcessingUrl: /login/form # 登录表单提交处理的url
      usernameParameter: name # 登录表单提交的用户名的属性名
      passwordParameter: pwd  # 登录表单提交的密码的属性名
      staticPaths: # 静态资源 "/dist/**", "/modules/**", "/plugins/**"
        - /dist/**
        - /modules/**
        - /plugins/**
```

### 5.5.2 读取自定义配置数据 

1.在 security-core 中创建 `com.security.properties.SecurityProperties `

- @ConfigurationProperties( prefix = "liubin.security")  绑定 application.yml 配置文件中 以 liubin.security 前缀的数据 
- 类上不要少了 @Component 注解 

> AuthenticationProperties 报错是因为类还未创建出来，继续往下第2步创建即可。 

2.在security-core中创建 com.security.properties.AuthenticationProperties 

```java
@Data
public class AuthenticationProperties {

    private String loginPage = "/login/page";
    private String loginProcessingUrl = "/login/form";
    private String usernameParameter = "name";
    private String passwordParameter = "pwd";
    private String[] staticPaths = {"/dist/**", "/modules/**", "/plugins/**"};

}
```



### 5.5.3 重构 SpringSecurityConfig

1. 将 SecurityProperties 注入到 SpringSecurityConfig

```java
Autowired 
private SecurityProperties securityProperties;
```



2. 将 SpringSecurityConfig#configure 的URL均通过 securityProperties 获取

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //http.httpBasic()
        http.formLogin()
                .loginPage("/login/page").permitAll()
                .loginProcessingUrl("/login/form")
                .usernameParameter("name")
                .passwordParameter("pwd")
                .and()
                .authorizeRequests() // 认证请求
                .antMatchers("/login/page").permitAll()
                .anyRequest().authenticated() // 所有进入应用的HTTP请求都要进行认证
        ;
    }

	@Override
    public void configure(WebSecurity web) {
        web.ignoring().antMatchers(securityProperties.getAuthentication().getStaticPaths());
    }

```



## 5.6 动态认证用户信息 

### 5.6.1 概述 

​		当前身份认证的用户名和密码是启动服务器自动生成的，或者是代码中写死的，存储在内存中。而实际项目中应该从动态的从数据库中获取进行身份认证。

### 5.6.2 实现流程 

1. 重点关注 UserDetailsService 、 UserDetails 接口 
2. 自定义一个 UserDetailsService 接口的实现类 CustomUserDetailsService ，实现该接口中的 loadUserByUsername 方法 ， 通过该方法定义获取用户信息的逻辑。
   - 从数据库获取到的用户信息封装到 UserDetail 接口的实现类中（Spring Security 提供了一个 org.springframework.security.core.userdetails.User 实现类封装用户信息）。 
   - 如果未获取到用户信息，则抛出异常 throws UsernameNotFoundException

```java
public interface UserDetails extends Serializable {
    此用户可访问的资源权限
    Collection<? extends GrantedAuthority> getAuthorities();
    
    密码
    String getPassword();
    
    用户名
    String getUsername();
    
    帐户是否过期(true 未过期，false 已过期)
    boolean isAccountNonExpired();
    
    帐户是否被锁定（true 未锁定，false 已锁定），锁定的用户是可以恢复的
    boolean isAccountNonLocked();
    
    密码是否过期（安全级别比较高的系统，如30天要求更改密码，true 未过期，false 过期）
    boolean isCredentialsNonExpired();
    
    帐户是否可用（一般指定是否删除，系统一般不会真正的删除用户信息，而是假删除，通过一个状态码标志
    用户被删除）删除的用户是可以恢复的
    boolean isEnabled();
}

```

### 5.6.3 编码实现

1. 在 security-web 创建 com.liuurick.security.CustomUserDetailsService ， 

   注意: 应用服务中才知道如何获取用户信息，所以在 security-web 中创建。 @Component("customUserDetailsService") 注解在实现类上不要少了

   ```java
   @Component("customUserDetailsService")
   @Slf4j
   public class CustomUserDetailsService implements UserDetailsService {
   
       @Autowired
       PasswordEncoder passwordEncoder;
   
       @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
           log.info("请求认证的用户名: " + username);
   
           // 1. 通过请求的用户名去数据库中查询用户信息
           if(!"liu".equalsIgnoreCase(username)) {
               throw new UsernameNotFoundException("用户名或密码错误");
           }
           // 假设当前这个用户在数据库当中存储的密码是1234
           String password = passwordEncoder.encode("123456");
           // 2. 查询该用户有哪一些权限
   
           // 3. 封装用户信息和权限信息
           // username 用户名, password 是数据库中这个用户存储的密码,
           // authorities 是权限资源标识, springsecurity会自动的判断用户是否合法,
           return new User(username, password,
                   AuthorityUtils.commaSeparatedStringToAuthorityList("ADMIN"));
       }
   }
   ```

   

2. 重构 com.liuurick.security.config.SpringSecurityConfig

- 注入 CustomUserDetailsService 
- configure(AuthenticationManagerBuilder auth) 方法中指定认证方式

```java
@Autowired
private UserDetailsService customUserDetailsService;

auth.userDetailsService(customUserDetailsService);
```

3. 如果上面 customUserDetailsService 有红色波浪线，idea自身检测的问题这不是bug，不影响项目运行 。 也可让idea不检测，就会去除红线。

4. 运行 WebAppliction，重启项目 
5. 访问 http://localhost:8080/index 被拦截，在输入非 liu 用户名，提示用户名或密码错误



## 5.7 自定义认证成功处理器

### 5.7.1 概述 

重点关注 AuthenticationSuccessHandler 接口 

当前登录成功后，跳转到之前请求的 url , 而现在希望登录成功后，实现其他的业务逻辑。比如累计积分、通过 Ajax 请求响应一个JSON数据，前端接收到响应的数据进行跳转。那可以使用自定义登录成功处理逻辑。

### 5.7.2 编码实现

1. 将MengxueguResult.java 工具类拷贝到 security-base 模块中的 com.liuurick.base.result 包下。用于封装响应JSON数据。 

2. 创建 com.liuurick.security.authentication.CustomAuthenticationSuccessHandler ，实 现 AuthenticationSuccessHandler 接口 

   `注意:实现类上不要少了注解 @Component("customAuthenticationSuccessHandler")`

```java
@Component("customAuthenticationSuccessHandler")
public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request,
        HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        // 认证成功后，响应JSON字符串
        Result result = Result.ok("认证成功");
        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().write(result.toJsonString());
    }
}
```



3. 在 SpringSecurityConfig 中注入 和 引用自定义认证成功处理器 customAuthenticationSuccessHandler

```java
/**
 * 注入自定义的认证成功处理器
*/
@Autowired
private AuthenticationSuccessHandler customAuthenticationSuccessHandler;

@Override
    protected void configure(HttpSecurity http) throws Exception {
        //http.httpBasic()
        http.formLogin()
                .loginPage("/login/page").permitAll()
                .loginProcessingUrl("/login/form")
                .usernameParameter("name")
                .passwordParameter("pwd")
                .successHandler(customAuthenticationSuccessHandler) // 认证成功处理器
                .and()
                .authorizeRequests() // 认证请求
                .antMatchers("/login/page").permitAll()
                .anyRequest().authenticated() // 所有进入应用的HTTP请求都要进行认证
        ;
    }
```

### 5.7.3 测试 

1. 重启项目，访问 http://localhost:8080 跳转到登录页面 
2. 登录成功后，页面响应数据

## 5.8 自定义认证失败处理器

### 5.8.1 概述 

重点关注 AuthenticationFailureHandler 接口

登录错误后记录日志，当次数超过3次后，2小时内不允许登录，那可以使用自定义登录失败后，进行逻辑处理。

### 5.8.2 编码实现

1. 创建 com.liuurick.security.authentication.CustomAuthenticationFailureHandler ，实 现 AuthenticationFailureHandler 接口 

   `注意:实现类上不要少了注解 @Component("customAuthenticationFailureHandler")`

   ```java
   @Component("customAuthenticationFailureHandler")
   public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {
       /**
        * @param exception 认证失败时抛出异常
        */
       @Override
       public void onAuthenticationFailure(HttpServletRequest request,
               HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
           // 认证失败响应JSON字符串，
           Result result = Result.build(HttpStatus.UNAUTHORIZED.value(), exception.getMessage());
           response.setContentType("application/json;charset=UTF-8");
           response.getWriter().write(result.toJsonString());
       }
   }
   ```

   

2. 在 SpringSecurityConfig 中注入 和 引用自定义认证失败处理器 customAuthenticationFailureHandler

   ![image-20210319204052982](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210319204052982.png)

3. 重启项目，访问 http://localhost :8080
4. 输入错误用户名和密码后，页面响应数据



## 5.9 升级异步请求认证处理 

1. 如果是通过ajax发送请求, 应该响应 JSON 通知前端认证成功或失败 
2. 否则直接重定向回来源请求 

要实现这个效果应该在 CustomAuthenticationFailureHandler 和 CustomAuthenticationSuccessHandler 加上一个类型判断，且我们将这些类型都可配置的。

### 5.9.1 创建响应类型枚举类













## 5.10 分析用户名密码认证底层源码







# 6 图形验证码、记住我和手机短信认证项目实战

课时26视频详解图形验证码认证实现流程04:20

课时27视频生成与获取图形验证码15:54

课时28视频自定义图形验证码过滤器18:21

课时29视频RememberMe记住我功能17:20

课时30视频分析记住我功能底层源码实现11:50

课时31视频详解手机短信验证码认证流程12:31

课时32视频创建短信发送服务接口10:28

课时33视频手机登录页与发送短信验证码11:52

课时34视频实现短信验证码校验过滤器05:51

课时35视频实现手机认证过滤器和封装手机认证信息13:51

课时36视频实现手机认证提供者和手机号获取用户信息14:25

课时37视频自定义认证配置组合手机认证组件15:49

课时38视频完善手机短信认证失败处理流程07:48

课时39视频手机验证码登录记住我功能与记住我底层源码分析20:51

课时40视频获取当前用户认证信息09:15

课时41视频重构身份认证代码实现路径可配置09:17

# 7 Session 会话管理与Redis搭建Session集群

课时42视频配置Session会话超时时长与自定义Session失效处理16:45

课时43视频同一用户只允许一台电脑登录情景一17:52

课时44视频同一用户只允许一台电脑登录情景二04:36

课时45视频解决手机短信验证码重复登录问题07:20

课时46视频Redis实现Session高可用集群12:10

课时47视频指定Cookie中保存SessionID名称10:14

课时48视频退出系统默认配置与退出底层源码分析13:58

课时49视频分析底层源码-退出不允许再次登录14:39

课时50视频解决退出不允许再次登录问题08:40

课时51视频解决Session超时不允许再次登录15:14

课时52视频分析底层源码记住我功能失效原因21:11

课时53视频自定义退出系统处理逻辑04:42

# 8 Spring Security 授权管理及方法级别权限控制

课时54视频整合系统管理模板页面到项目中18:04

课时55视频创建用户角色菜单模块控制层06:51

课时56视频配置左侧菜单路径与动态样式切换17:00

课时57视频详解和演示Spring EL权限表达式17:59

课时58视频权限表达式控制权限和定制403错误页面13:38

课时59视频分层分模块管理安全和应用的授权配置25:29

课时60视频详解权限注解控制方法级别权限06:50

课时61视频注解控制用户管理模块方法级权限15:14

课时62视频注解过滤方法级参数和返回值11:48

课时63视频Thyemleaf视图权限控制之认证与授权表达式的使用实战11:53

课时64视频Thyemleaf认证与授权标签视图控制角色管理权限实战15:30

#9 整合 MyBatis-Plus 实现数据库动态认证

课时65视频介绍基于RBAC模型实现角色的权限访问控制10:40

课时66视频详细介绍与创建RBAC用户角色权限表09:25

课时67视频SpringBoot整合Druid连接池和Mybatis-Plus13:30

课时68视频整合MyBatis-Plus实现用户管理Service层27:37

课时69视频采用BaseMapper通过用户名查询数据库07:21

课时70视频定义角色管理实体类与Mapper和Service07:42

课时71视频定义权限管理实体类与Mapper和Service10:27

课时72视频多表关联查询用户所拥有的权限20:50

课时73视频用户名密码数据库动态身份认证14:17

课时74视频手机号登录数据库动态身份认证05:55

课时75视频模板设计模式重构动态身份认证12:40

# 10 RBAC 权限管理项目-权限资源管理

课时76视频介绍与演示RBAC统一权限管理系统05:31

课时77视频实现权限管理列表后台逻辑代码16:11

课时78视频完成权限管理树状列表渲染功能27:11

课时79视频权限控制列表中删除和修改按钮的隐藏和显示26:17

课时80视频跳转新增和修改权限页面11:38

课时81视频通过JQuery-Tree异步加载权限树渲染24:06

课时82视频回修权限基本信息和处理父资源问题37:06

课时83视频实现权限管理修改权限功能和开启SpringBoot2.2版本对PUT方式12:06

课时84视频实现权限管理新增功能11:18

课时85视频实现删除当前权限及子权限的后台逻辑08:16

课时86视频_前端异步发送DELETE请求方式删除权限资源11:03

# 11 RBAC 权限管理项目-角色管理

课时87视频角色管理功能介绍与MyBatis-Plus实现分页查询角色13:49

课时88视频实现角色列表业务层和控制层06:53

课时89视频BootstarpTable实现角色列表分页17:24

课时90视频条件搜索和权限控制角色删除和修改按钮13:21

课时91视频查询要修改的角色与权限信息22:07

课时92视频修改页面回显角色信息06:21

课时93视频加载权限树与勾选角色所拥有的权限12:54

课时94视频获取分配给角色的权限信息11:57

课时95视频批量新增角色与权限关系表数09:18

课时96视频实现修改角色服务层与控制层10:54

课时97视频新增角色及分配角色权限08:40

课时98视频删除角色及角色权限关系数据09:44

# 12 RBAC 权限管理项目-用户管理

课时99视频用户管理功能介绍与条件分页查询用户信息16:19

课时100视频bootstrapTable插件分页渲染数据与条件查询15:26

课时101视频权限控制用户列表中修改和删除按钮.05:46

课时102视频修改用户-查询用户所拥有的角色09:41

课时103视频修改用户-Service与Controller层查询用户和角色10:32

课时104视频修改用户-回显用户信息和勾选拥有角色21:41

课时105视频处理修改后的用户信息及绑定的角色14:25

课时106视频实现假删除用户信息14:53

# 13 RBAC 权限管理项目-权限控制菜单

课时107视频分析不同权限用户动态渲染菜单实现流程05:49

课时108视频定义认证成功监听器实现加载用户权限13:59

课时109视频获取用户菜单与重组父子结构菜单数据23:44

课时110视频前端获取权限菜单动态渲染20:05

# 14 OAuth2 协议标准简介和分析认证授权模式流程

课时111视频OAuth2是什么06:06

课时112视频OAuth2要解决的问题03:40

课时113视频OAuth2中涉及的角色及作用07:51

课时114视频详解OAuth2微信认证流程图16:02

课时115视频详解OAuth2协议的四种授权模式流程图17:41

# 15 Spring Security OAuth2 认证服务器

课时116视频概述与创建OAuth2项目父工程和基础模块15:14

课时117视频创建OAuth2认证服务器模块07:20

课时118视频定义认证服务器配置和安全配置和测试授权码模式流程37:25

课时119视频实操OAuth2密码模式07:50

课时120视频实操OAuth2简化模式和客户端模式10:45

# 16 OAuth2 认证服务器高级策略配置

课时121视频配置刷新令牌实战20:04

课时122视频Redis管理访问令牌Token09:27

课时123视频JDBC管理访问令牌Token14:03

课时124视频JDBC管理授权码05:14

课时125视频JDBC管理第三方应用(客户端)信息19:41

课时126视频配置令牌端点安全策略08:01

课时127视频基于RBAC动态认证用户15:25

# 17 Spring Security OAuth2 资源服务器

课时128视频创建商品资源与配置资源服务器16:34

课时129视频请求认证服务器与资源服务器06:59

课时130视频禁用Session和控制令牌范围权限与授权规则配11:10

# 18 JWT 访问令牌使用对称与非对称加密

课时131视频JWT是什么和能解决什么问题09:48

课时132视频认证服务器对称加密JWT管理令牌06:37

课时133视频资源服务器对称加密本地验证JWT令牌04:18

课时134视频认证服务器非对称加密JWT令牌10:22

课时135视频资源服务器非对称加密JWT令牌10:14

# 19 Spring Security OAuth2 实现SSO单点登录

课时136视频时序图分析单点登录实现流程08:06

课时137视频搭建单点登录会员客户端一(模拟淘宝)20:15

课时138视频搭建单点登录会员客户端二(模拟天猫)和实现流程分析12:15

课时139视频单点登录多套系统实现统一退出11:24

# 20 Spring Cloud OAuth2 分布式认证授权

课时140视频分析要搭建Spring Cloud OAuth2分布式微服务架构图04:28

课时141视频搭建分布式微服务注册中心05:10

课时142视频认证服务器和资源服务器注册到注册中心06:15

课时143视频搭建路由网关基础环境08:01

课时144视频JWT令牌管理和配置网关的资源配置类12:32

课时145视频定义安全配置类和自定义过滤器处理封装用户信息13:26

课时146视频资源服务器解析并完成认证授权17:26

课时147视频网关整合单点登录客户端05:41

课时148视频客户端带上令牌请求资源服务器获取数据需购买观看

课时149视频全套课程大总结