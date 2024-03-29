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

Spring Security 是基于 Spring 的身份认证（Authentication）和用户授权（Authorization）框架，提供了一 套 Web 应用安全性的完整解决方案。其中核心技术使用了 Servlet 过滤器、IOC 和 AOP 等。 

### 什么是身份认证 

认证（Authentication）：认证解决“我是谁”的问题

身份认证指的是用户去访问系统资源时，系统要求验证用户的身份信息，用户身份合法才访问对应资源。 常见的身份认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。 

### 什么是用户授权 

授权（Authorization）：授权解决“我能做什么”的问题

当身份认证通过后，去访问系统的资源，系统会判断用户是否拥有访问该资源的权限，只允许访问有权限的 系统资源，没有权限的资源将无法访问，这个过程叫用户授权。 比如 会员管理模块有增删改查功能，有的用户只能进行查询，而有的用户可以进行修改、删除。一般来说， 系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。

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

AdminLTE是一款建立在 bootstrap 和 jquery 之上的开源前端模板，它提供了一系列响应的、可重复使用的组件， 并内置了多个模板页面；同时自适应多种屏幕分辨率，兼容PC和移动端。 通过AdminLTE，我们可以快速的创建一个响应式的Html5网站。 AdminLTE框架在网页架构与设计上，有很大的辅助作用，尤其是前端架构设计师，用好 AdminLTE 不但美观，而 且可以免去写很大CSS与JS的工作量。 

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

![image-20210316162802532](/images/2021031306.png)



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

![image-20210319173850288](/images/2021031307.png)

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

重点关注 `AuthenticationSuccessHandler` 接口 

当前登录成功后，跳转到之前请求的 url , 而现在希望登录成功后，实现其他的业务逻辑。比如累计积分、通过 Ajax 请求响应一个JSON数据，前端接收到响应的数据进行跳转。那可以使用自定义登录成功处理逻辑。

### 5.7.2 编码实现

1. 将Result.java 工具类拷贝到 security-base 模块中的 com.liuurick.base.result 包下。用于封装响应JSON数据。 

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

重点关注 `AuthenticationFailureHandler` 接口

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

   ![image-20210319204052982](/images/2021031308.png)

3. 重启项目，访问 http://localhost:8080
4. 输入错误用户名和密码后，页面响应数据



## 5.9 升级异步请求认证处理 

1. 如果是通过ajax发送请求, 应该响应 JSON 通知前端认证成功或失败 
2. 否则直接重定向回来源请求 

要实现这个效果应该在 `CustomAuthenticationFailureHandler` 和 `CustomAuthenticationSuccessHandler` 加上一个类型判断，且我们将这些类型都可配置的。

### 5.9.1 创建响应类型枚举类

在 security-core 中创建枚举类 com.liuurick.security.properties.LoginResponseType

```java
public enum LoginResponseType {

    /**
     * 响应JSON字符串
     */
    JSON,

    /**
     * 重定向地址
     */
    REDIRECT
}
```



### 5.9.2 配置 application.yml 

1. 在security-web\src\main\resources\application.yml 文件中配置登录类型 
   - `REDIRECT` 表示重向一个页面 
   - JSON 响应JSON字符串

### 5.9.3 读取自定义配置数据

在 com.liuurick.security.properties.AuthenticationProperties 添加 loginType 属性

```java
	/**
     * 登录成功后响应 JSON , 还是重定向
     * 如果application.yml 中没有配置，则取此初始值 REDIRECT
    */
    private LoginResponseType loginType = LoginResponseType.REDIRECT;

    public LoginResponseType getLoginType() {
        return loginType;
    }
    public void setLoginType(LoginResponseType loginType) {
        this.loginType = loginType;
    }
```



### 5.9.4 重构成功处理器

重构 `com.liuurick.security.authentication.CustomAuthenticationSuccessHandler` 

1. 将实现改为继承 `extends SavedRequestAwareAuthenticationSuccessHandler` 默认实现类 
2. 加上判断响应 JSON 还是 重定向回认证来源请求 
3. 采用 `SecurityProperties.authentication.loginType` 配置值进行判断认证响应类型。

4. 重启测试，当 application.yml 配置不同登录类型，登录用户信息正确时，是否响应效果不一样



### 5.9.5 重构失败处理器 

1. 将实现改为继承 `extends SimpleUrlAuthenticationFailureHandler `类 
2. 加上判断响应 JSON 还是 重定向回认证来源请求 
3. 采用 `SecurityProperties.authentication.loginType` 配置值进行判断认证响应类型。 
4. 要指定重写向回登录页时，要指定上次请求地址加 `?error`

5. 重启测试，当 application.yml 配置不同登录类型，登录用户信息错误时，是否响应效果不一样。



## 5.10 分析用户名密码认证底层源码

![image-20210320170108290](/images/2021031309.png)





# 6 图形验证码/记住我/手机短信认证项目实战

## 6.1 图形验证码认证功能 

### 6.1.1 分析实现流程

![image-20210320172616741](/images/2021031310.png)



### 6.1.2 生成一张图形验证码 

Kaptcha 是谷歌提供的一个生成图形验证码的 jar 包, 只要简单配置属性就可以生成。

> 参考 ：https://github.com/penggle/kaptcha

1. 添加 Kaptcha 依赖，在 security-core\pom.xml 中

```xml
<dependency>
	<groupId>com.github.penggle</groupId>
	<artifactId>kaptcha</artifactId>
</dependency>
```

2. 生成验证码配置类，在 security-core 模块中创建 `com.liuurick.security.authentication.code.KaptchaImageCodeConfig`

   ```java
   @Configuration
   public class KaptchaImageCodeConfig {
       
       @Bean
       public DefaultKaptcha getDefaultKaptcha(){
   
           DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
           Properties properties = new Properties();
           properties.setProperty(Constants.KAPTCHA_BORDER, "yes");
           properties.setProperty(Constants.KAPTCHA_BORDER_COLOR, "192,192,192");
           properties.setProperty(Constants.KAPTCHA_IMAGE_WIDTH, "110");
           properties.setProperty(Constants.KAPTCHA_IMAGE_HEIGHT, "36");
           properties.setProperty(Constants.KAPTCHA_TEXTPRODUCER_FONT_COLOR, "blue");
           properties.setProperty(Constants.KAPTCHA_TEXTPRODUCER_FONT_SIZE, "28");
           properties.setProperty(Constants.KAPTCHA_TEXTPRODUCER_FONT_NAMES, "宋体");
           properties.setProperty(Constants.KAPTCHA_TEXTPRODUCER_CHAR_LENGTH, "4");
           // 图片效果
           properties.setProperty(Constants.KAPTCHA_OBSCURIFICATOR_IMPL, "com.google.code.kaptcha.impl.ShadowGimpy");
           Config config = new Config(properties);
           defaultKaptcha.setConfig(config);
           return defaultKaptcha;
       }
   }
   ```

3. 在 CustomLoginController 提供请求接口，将验证码图片数据流写出

   ```java
   @Autowired
       private DefaultKaptcha defaultKaptcha;
   
       /**
        * 获取图形验证码
        */
       @RequestMapping("/code/image")
       public void imageCode(HttpServletRequest request, HttpServletResponse response) throws
               IOException {
           // 1. 获取验证码字符串
           String code = defaultKaptcha.createText();
           log.info("生成的图形验证码是：" + code);
           // 2. 字符串把它放到session中
           request.getSession().setAttribute(SESSION_KEY , code);
           // 3. 获取验证码图片
           BufferedImage image = defaultKaptcha.createImage(code);
           // 4. 将验证码图片把它写出去
           ServletOutputStream out = response.getOutputStream();
           ImageIO.write(image, "jpg", out);
       }
   ```

   

4. 在 SpringSecurityConfig.configure(HttpSecurity http) 放行 /code/image 资源权限

   ```java
   antMatchers(securityProperties.getAuthentication().getLoginPage(),
               "/code/image").permitAll()
   ```

5. 重构 security-web 模块的 login.html 页面，调用验证码接口渲染图片

```html
<div class="row mb-2 ">
	<div class="col-6">
		<input name="code" type="text" class="form-control" placeholder="验证码">
	</div>
	<div class="col-6">
		<img onclick="this.src='/code/image?'+Math.random()" src="/code/image" alt="验证码" />
	</div>
</div>

```

**kaptcha参数说明:**

![img](/images/2021031311.png)

### 6.1.3 实现验证码校验过滤器

1. 创建 `com.liuurick.security.authentication.code.ImageCodeValidateFilter` ，继承 `OncePerRequestFilter` （在 所有请求前都被调用一次） 
2. 在类上加上注解 `@Component("imageCodeValidateFilter") `
3. 如果是登录请求（请求地址： /login/form ，请求方式： post ），校验验证码输入是否正确 校验不合法时，提示信息通过自定义异常 `ValidateCodeExcetipn` 抛出 , 此异常要继承 `org.springframework.security.core.AuthenticationException` ，它是认证的父异常类。 捕获 `ImageCodeException` 异常，交给失败处理器 `CustomAuthenticationFailureHandler` 。 
4. 如果非登录请求，则放行请求 `filterChain.doFilter(request, response)`



### 6.1.4 创建验证码异常类

创建 `com.liuurick.security.authentication.exception.ValidateCodeExcetipn` 异常类，它继承 `AuthenticationException`

> 特别注意是：org.springframework.security.core.AuthenticationException

```java
public class ValidateCodeExcetipn extends AuthenticationException {
    public ValidateCodeExcetipn(String msg, Throwable t) {
        super(msg, t);
    }
    public ValidateCodeExcetipn(String msg) {
        super(msg);
    }
}
```





### 6.1.5 重构 SpringSecurityConfig

将校验过滤器 `imageCodeValidateFilter` 添加到 `UsernamePasswordAuthenticationFilter `前面 

在 `com.liuurick.security.config.SpringSecurityConfig` 中完成以下操作： 

1. 注入 ImageCodeValidateFilter 实例 

   ```java
   / 验证码校验过滤器
   @Autowired
   ImageCodeValidateFilter imageCodeValidateFilter;
   ```

2. 把 `ImageCodeValidateFilter` 添加 `UsernamePasswordAuthenticationFilter` 实例前

![image-20210323205427643](/images/2021031312.png)



##　6.2 Remember-Me 记住我功能

效果: 登录后会记住用户令牌，不用反复登录 。 

### 6.2.1 分析 Remember-Me 实现流程 

1. 用户选择了“记住我”成功登录后，将会把username、随机生成的序列号、生成的token存入一个数据库表 中，同时将它们的组合生成一个cookie发送给客户端浏览器。 
2. 当没有登录的用户访问系统时，首先检查 remember-me 的 cookie 值 ，有则检查其值包含的 username、 序列号和 token 与数据库中是否一致，一致则通过验证。 并且系统还会重新生成一个新的 token 替换数据库中对应旧的 token，序列号 series 保持不变 ，同时删除旧 的 cookie，重新生成 cookie 值（新的 token + 旧的序列号 + username）发送给客户端。 
3. 如果对应cookie不存在，或者包含的username、序列号和token 与数据库中保存的不一致，那么将会引导用 户到登录页面。 因为cookie被盗用后还可以在用户下一次登录前顺利的进行登录，所以如果你的应用对安全性要求比较高就 不要使用Remember-Me功能。



### 6.2.2 实现用户名密码 Remember-Me 功能

1. security-core 的 pom.xml 引入依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency
```

2. liuurick-security-web 的 application.yml 配置数据源,

```yml
spring:
  thymeleaf:
    cache: false #关闭thymeleaf缓存
  # 数据源配置
  datasource:
  username: root
  password: root
  url: jdbc:mysql://127.0.0.1:3306/study-security?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8
  #mysql8版本以上驱动包指定新的驱动类
  driver-class-name: com.mysql.cj.jdbc.Driver
```

3. 在数据库中创建一个 study-security

4. 使用 JdbcTokenRepository 实现类

```java
@Autowired
DataSource dataSource;

@Bean
public JdbcTokenRepositoryImpl jdbcTokenRepository() {
    JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
    jdbcTokenRepository.setDataSource(dataSource);
    return jdbcTokenRepository;
}
```

5. 安全配置 SpringSecurityConfig#configure(HttpSecurity http)

```java
.and()
.rememberMe() //记住我
.tokenRepository(jdbcTokenRepository()) //保持登陆信息
.tokenValiditySeconds(60*60*24*7) //保持登陆时间
```

6. 修改security-web 模块中的 login.html 记住我的 name="remember-me"

7. 测试效果 
   1. 重启，数据库会自动创建表 `persistent_logins `
   2. 访问首页跳转到登录页面，登录成功后，查看浏览器 cookies，有保存 token 值。 数据库中也有数据。
   3. 再重启，Session会把清除。再次访问首页，不会跳转到登录页，因为上次已经记录了。这时浏览器会带 着Cookie中保存的token从数据库查找用户名，然后进行自动登录。



### 6.2.3 分析 Remember-Me 底层源码实现

![img](file:///C:\Users\admin\Documents\Tencent Files\2826803629\Image\C2C\U{8GEBK3PKJBZP[FDLBEE]G.png)

- `UsernamePasswordAuthenticationFilter `拥有一个 `RememberMeServices` 的引用，默认是一个空实现的 `NullRememberMeServices` ，而实际当我们通过 `rememberMe()` 启用 Remember-Me 时，它是一个具体的实现。 

- 用户的请求会先通过 `UsernamePasswordAuthenticationFilter` ，当认证成功后会调用 `RememberMeServices` 的 `loginSuccess()` 方法，否则调用 `RememberMeServices` 的 `loginFail()` 方法。`UsernamePasswordAuthenticationFilter `不会调用 `RememberMeServices` 的 `autoLogin()` 方法进行自动登录 的。 

- 当执行到 `RememberMeAuthenticationFilter` 时，如果检测到还没有认证成功时，那么 `RememberMeAuthenticationFilter` 会尝试着调用所包含的 `RememberMeServices` 的 `autoLogin()` 方法进行自 动登录。 
- `PersistentTokenBasedRememberMeServices` 是 `RememberMeServices` 的启动`Remember-Me` 默认实现 , 它会通过 cookie 值进行查询数据库存储的记录, 来实现自动登录 ，并重新生成新的 cookie 存储。



## 6.3 手机短信验证码认证功能

### 6.3.1 分析实现流程

手机号登录是不需要密码的，通过短信验证码实现免密登录功能。 

1. 向手机发送手机验证码，使用第三方短信平台 SDK 发送，如: 阿里云短信服务（阿里大于） 
2. 登录表单输入短信验证码 
3. 使用自定义过滤器 `MobileValidateFilter `
4. 当验证码校验通过后，进入自定义手机认证过滤器 `MobileAuthenticationFilter` 校验手机号是否存在 
5. 自定义 `MobileAuthenticationToken` 提供给 `MobileAuthenticationFilter `
6. 自定义 `MobileAuthenticationProvider` 提供给 `ProviderManager` 处理 
7. 创建针对手机号查询用户信息的 `MobileUserDetailsService` ，交给 `MobileAuthenticationProvider`
8. 自定义 `MobileAuthenticationConfig` 配置类将上面组件连接起来，添加到容器中 
9. 将 `MobileAuthenticationConfig` 添加到 `SpringSecurityConfig` 安全配置的过滤器链上。

![image-20210327180908660](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210327180908660.png)

### 6.3.2 创建短信发送服务接口

1. 定义短信发送服务接口 `com.liuurick.security.authentication.mobile.SmsSend`

   ```java
   public interface SmsSend {
   	boolean sendSms(String mobile, String content);
   }
   ```

2. 实现短信发送服务接口 `com.liuurick.security.authentication.mobile.SmsCodeSender`

   ```java
   @Slf4j
   public class SmsCodeSender implements SmsSend {
       @Override
       public boolean sendSms(String mobile, String content) {
           String sendcontent = String.format("尊敬的客户，您的验证码为%s，请勿泄露他人。", content);
           log.info("验证码"+content);
           return true;
       }
   }
   ```

3. 在 `com.liuurick.security.config` 创建 `SeurityConfigBean` 将 `SmsCodeSender` 添加到Spring容器。 也可以直接在类上加上 `@Component` 注解，但是不利于应用的扩展，因为短信服务提供商有非常多，实现就不一样，所以采用下面方式添加到容器：

   ```java
   @Configuration
   public class SeurityConfigBean {
   	/**
   	* @ConditionalOnMissingBean(SmsSend.class)
   	* 默认采用SmsCodeSender实例 ，但如果容器中有其他 SmsSend 类型的实例，则当前实例失效
   	*/
       @Bean
       @ConditionalOnMissingBean(SmsSend.class)
       public SmsSend getSmsSend(){
           return new SmsCodeSender();
       }
   }
   ```

4. 在 security-web 中创建一个 `MobileSmsCodeSender` 短信服务接口的实现，来替换默认实现 `SmsCodeSender`  ,后面再进行测试这个接口。

   ```java
   @Slf4j
   public class MobileSmsCodeSender implements SmsSend {
   
       @Override
       public boolean sendSms(String mobile, String content) {
           String sendcontent = String.format("尊敬的客户，您的验证码为%s，请勿泄露他人。", content);
           log.info("验证码"+content);
           return false;
       }
   }
   ```

### 6.3.3 手机登录页与发送短信验证码

1. 创建 `com.liuurick.security.controller.CustomMobileController` 添加如下代码

   ```java
   @Controller
   public class MobileLoginController {
   
       public static final String SESSION_KEY = "SESSION_KEY_MOBILE_CODE";
   
       /**
        * 前往手机验证码登录页
        * @return
        */
       @RequestMapping("/mobile/page")
       public String toMobilePage() {
           // templates/login-mobile.html
           return "login-mobile";
       }
   
       @Autowired
       SmsSend smsSend;
   
       /**
        * 发送手机验证码
        * @return
        */
       @ResponseBody
       @RequestMapping("/code/mobile")
       public Result smsCode(HttpServletRequest request) {
           // 1. 生成一个手机验证码
           String code = RandomStringUtils.randomNumeric(4);
           // 2. 将手机验证码保存到session中
           request.getSession().setAttribute(SESSION_KEY, code);
           // 3. 发送验证码到用户手机上
           String mobile = request.getParameter("mobile");
           smsSend.sendSms(mobile, code);
   
           return Result.ok();
       }
   }
   ```

2. 在 security-web 模块添加手机登录页面 login-mobile.html 到 templates 目录下 手机登录表单核心代码 `th:action="@{/mobile/form}"`

   ` th:attr="code_url=@{/code/mobile?mobile=}"`

3. 在 SpringSecurityConfig 放行手机登录相关请求URL

   ```java
   antMatchers(securityProperties.getAuthentication().getLoginPage(),
   "/code/image", "/mobile/page", "/code/mobile").permitAll()
   ```

   

### 6.3.4 实现短信验证码校验过滤器 MobileValidateFilter

校验输入的验证码与发送的短信验证是否一致。 创建 `com.liuurick.security.authentication.mobile.MobileValidateFilter` ， 与图形证验码过滤器 `ImageCodeValidateFilter` 实现类似。 不要少了 @Component

```java
@Component
public class MobileValidateFilter extends OncePerRequestFilter {
    
    @Autowired
    CustomAuthenticationFailureHandler customAuthenticationFailureHandler;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // 1. 判断 请求是否为手机登录，且post请求
        if("/mobile/form".equals(request.getRequestURI())
            && "post".equalsIgnoreCase(request.getMethod())) {
            try {
                // 校验验证码合法性
                validate(request);
            }catch (AuthenticationException e) {
                // 交给失败处理器进行处理异常
                customAuthenticationFailureHandler.onAuthenticationFailure(request, response, e);
                // 一定要记得结束
                return;
            }
        }

        // 放行
        filterChain.doFilter(request, response);
    }

    private void validate(HttpServletRequest request) {
        // 先获取seesion中的验证码
        String sessionCode =
                (String)request.getSession().getAttribute(MobileLoginController.SESSION_KEY);
        // 获取用户输入的验证码
        String inpuCode = request.getParameter("code");
        // 判断是否正确
        if(StringUtils.isBlank(inpuCode)) {
            throw new ValidateCodeException("验证码不能为空");
        }

        if(!inpuCode.equalsIgnoreCase(sessionCode)) {
            throw new ValidateCodeException("验证码输入错误");
        }
    }
}
```

### 6.3.5 实现手机认证过滤器 MobileAuthenticationFilter

创建 `com.liuurick.security.authentication.mobile.MobileAuthenticationFilter` ， 模仿 `UsernamePasswordAuthenticationFilter` 代码进行改造

```java
public class MobileAuthenticationFilter extends AbstractAuthenticationProcessingFilter {

    private String mobileParameter = "mobile";
    private boolean postOnly = true;


    public MobileAuthenticationFilter() {
        super(new AntPathRequestMatcher("/mobile/form", "POST"));
    }

    // ~ Methods
    // ========================================================================================================

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request,
                                                HttpServletResponse response) throws AuthenticationException {
        if (postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException(
                    "Authentication method not supported: " + request.getMethod());
        }

        String mobile = obtainMobile(request);

        if (mobile == null) {
            mobile = "";
        }

        mobile = mobile.trim();

        MobileAuthenticationToken authRequest = new MobileAuthenticationToken(mobile);

        // sessionID, hostname
        setDetails(request, authRequest);

        return this.getAuthenticationManager().authenticate(authRequest);
    }

    /**
     * 从请求中获取手机号码
     */
    @Nullable
    protected String obtainMobile(HttpServletRequest request) {
        return request.getParameter(mobileParameter);
    }

    /**
     * 将 sessionID和hostname添加 到MobileAuthenticationToken
     */
    protected void setDetails(HttpServletRequest request,
                              MobileAuthenticationToken authRequest) {
        authRequest.setDetails(authenticationDetailsSource.buildDetails(request));
    }

    /**
     * 设置是否为post请求
     */
    public void setPostOnly(boolean postOnly) {
        this.postOnly = postOnly;
    }

    public String getMobileParameter() {
        return mobileParameter;
    }

    public void setMobileParameter(String mobileParameter) {
        this.mobileParameter = mobileParameter;
    }
}
```

### 6.3.6 封装手机认证Token MobileAuthenticationToken

创建 `com.liuurick.security.authentication.mobile.MobileAuthenticationToken` 提供给上面自定义的 MobileAuthenticationFilter 使用

```java
public class MobileAuthenticationToken extends AbstractAuthenticationToken {

    private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

    /**
     * 认证前是手机号码,认证后是用户信息
     */
    private final Object principal;

    /**
     * 认证之前使用的构造 方法, 此方法会标识未认证
     */
    public MobileAuthenticationToken(Object principal) {
        super(null);
        // 手机号码
        this.principal = principal;
        // 未认证
        setAuthenticated(false);
    }

    /**
     * 认证通过后,会重新创建MobileAuthenticationToken实例 ,来进行封装认证信息
     * @param principal 用户信息
     * @param authorities 权限资源
     */
    public MobileAuthenticationToken(Object principal, Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.principal = principal;
        // must use super, as we override
        super.setAuthenticated(true);
    }

    /**
     * 因为它是父类中的抽象方法,,所以要实现,直接返回null即可
     * @return
     */
    @Override
    public Object getCredentials() {
        return null;
    }


    @Override
    public Object getPrincipal() {
        return this.principal;
    }

    @Override
    public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
        if (isAuthenticated) {
            throw new IllegalArgumentException(
                    "Cannot set this token to trusted - use constructor which takes a GrantedAuthority list instead");
        }
        super.setAuthenticated(false);
    }

    @Override
    public void eraseCredentials() {
        super.eraseCredentials();
    }
}
```

### 6.3.7 实现手机认证提供者 MobileAuthenticationProvider

创建 `com.liuurick.security.authentication.mobile.MobileAuthenticationProvider` ， 提供给底层 `ProviderManager` 使用

```java
public class MobileAuthenticationProvider implements AuthenticationProvider {

    private UserDetailsService userDetailsService;

    public void setUserDetailsService(UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }

    /**
     * 认证处理:
     *  1. 通过手机号码 查询用户信息( UserDetailsService实现)
     *  2. 当查询到用户信息, 则认为认证通过,封装Authentication对象
     * @param authentication
     * @return
     * @throws AuthenticationException
     */
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        MobileAuthenticationToken mobileAuthenticationToken =
                (MobileAuthenticationToken)authentication;
        // 获取手机号码
        String mobile = (String)mobileAuthenticationToken.getPrincipal();
        // 通过 手机号码 查询用户信息( UserDetailsService实现)
        UserDetails userDetails =
                userDetailsService.loadUserByUsername(mobile);

        // 未查询到用户信息
        if(userDetails == null) {
            throw new AuthenticationServiceException("该手机号未注册");
        }

        // 认证通过
        // 封装到 MobileAuthenticationToken
        MobileAuthenticationToken authenticationToken =
                new MobileAuthenticationToken(userDetails, userDetails.getAuthorities());
        authenticationToken.setDetails(mobileAuthenticationToken.getDetails());
        //最终返回认证信息
        return authenticationToken;
    }

    /**
     * 通过这个方法,来选择对应的Provider, 即选择MobileAuthenticationProivder
     * @param authentication
     * @return
     */
    @Override
    public boolean supports(Class<?> authentication) {
        return MobileAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

### 6.3.8 手机号获取用户信息 MobileUserDetailsService

在 `security-web` 模块中创建 UserDetailsService 实现类： `com.liuurick.security.MobileUserDetailsService` 

**注意：**不要注入 PasswordEncoder

```java
@Component("mobileUserDetailsService")
@Slf4j
public class MobileUserDetailsService implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String mobile) throws UsernameNotFoundException {
        log.info("请求的手机号是：" + mobile);
        // 1. 通过手机号查询用户信息
        // 2. 如果有用户信息，则再获取权限资源
        // 3. 封装用户信息

        return new User("liubin", "", true, true, true, true,
                AuthorityUtils.commaSeparatedStringToAuthorityList("ADMIN"));
    }
}
```



### 6.3.9 自定义管理认证配置 MobileAuthenticationConfig

创建 `com.liuurick.security.authentication.mobile.MobileAuthenticationConfig` 类 将上面定义的组件绑定起来，添加到容器中：

```java
@Component
public class MobileAuthenticationConfig
        extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {

    @Autowired
    CustomAuthenticationSuccessHandler customAuthenticationSuccessHandler;

    @Autowired
    CustomAuthenticationFailureHandler customAuthenticationFailureHandler;

    @Autowired
    UserDetailsService mobileUserDetailsService;


    @Override
    public void configure(HttpSecurity http) throws Exception {

        MobileAuthenticationFilter mobileAuthenticationFilter = new MobileAuthenticationFilter();
        // 获取容器中已经存在的AuthenticationManager对象，并传入 mobileAuthenticationFilter 里面
        mobileAuthenticationFilter.setAuthenticationManager(
                http.getSharedObject(AuthenticationManager.class));

        // 为了实现手机登录也拥有记住我的功能,将RememberMeServices传入
//        smsCodeAuthenticationFilter.setRememberMeServices(
//                http.getSharedObject(RememberMeServices.class));


        // 指定记住我功能
        mobileAuthenticationFilter.setRememberMeServices(http.getSharedObject(RememberMeServices.class));

        // 传入 失败与成功处理器
        mobileAuthenticationFilter.setAuthenticationSuccessHandler(customAuthenticationSuccessHandler);
        mobileAuthenticationFilter.setAuthenticationFailureHandler(customAuthenticationFailureHandler);

        // 构建一个MobileAuthenticationProvider实例，接收 mobileUserDetailsService 通过手机号查询用户信息
        MobileAuthenticationProvider provider = new MobileAuthenticationProvider();
        provider.setUserDetailsService(mobileUserDetailsService);

        // 将provider绑定到 HttpSecurity上，并将 手机号认证过滤器绑定到用户名密码认证过滤器之后
        http.authenticationProvider(provider)
                .addFilterAfter(mobileAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

    }
}
```



### 6.3.10 绑定到安全配置 SpringSecurityConfig

1. 向 `SpringSecurityConfig` 中注入 `MobileValidateFilter` 和 `MobileAuthenticationConfig` 实例 

2. 将 `MobileValidateFilter` 实例添加到 `UsernamePasswordAuthenticationFilter` 前面 

   ```java
   http.addFilterBefore(mobileValidateFilter,
   	UsernamePasswordAuthenticationFilter.class)
   .addFilterBefore(imageCodeValidateFilter,
   	UsernamePasswordAuthenticationFilter.class)
   ```

3. 在 `SpringSecurityConfig#configure(HttpSecurity http)` 方法体最后 调用 apply 添加 `smsCodeAuthenticationConfig `

   ```java
   http.apply(mobileAuthenticationConfig);
   ```

4. SpringSecurityConfig 完整代码

   ```java
   @Configuration
   @EnableWebSecurity
   @Slf4j
   public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {
   
       @Autowired
       private SecurityProperties securityProperties;
   
       @Autowired
       private UserDetailsService customUserDetailsService;
   
       /**
        * 验证码校验过滤器
         */
       @Autowired
       private ImageCodeValidateFilter imageCodeValidateFilter;
   
       /**
        * 注入自定义的认证成功处理器
         */
       @Autowired
       private AuthenticationSuccessHandler customAuthenticationSuccessHandler;
   
       @Autowired
       private AuthenticationFailureHandler customAuthenticationFailureHandler;
   
       @Bean
       public PasswordEncoder passwordEncoder(){
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
   
           /*String password = passwordEncoder().encode("123456");
           log.info("加密密码为："+password);
           auth.inMemoryAuthentication()
                   .withUser("liubin")
                   .password(password)
                   .authorities("admin");*/
   
           //用户信息存储在数据库中
           auth.userDetailsService(customUserDetailsService);
       }
   
       /**
        * 记住我功能
        */
       @Autowired
       DataSource dataSource;
   
       @Bean
       public JdbcTokenRepositoryImpl jdbcTokenRepository() {
           JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
           jdbcTokenRepository.setDataSource(dataSource);
           return jdbcTokenRepository;
       }
   
       @Autowired
       private MobileValidateFilter mobileValidateFilter;
   
       @Autowired
       private MobileAuthenticationConfig mobileAuthenticationConfig;
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
               //http.httpBasic()
               http.addFilterBefore(mobileValidateFilter, UsernamePasswordAuthenticationFilter.class)
                   .addFilterBefore(imageCodeValidateFilter, UsernamePasswordAuthenticationFilter.class)
                   .formLogin() // 表单登录方式
                   .loginPage("/login/page").permitAll()
                   .loginProcessingUrl("/login/form")
                   .usernameParameter("name")
                   .passwordParameter("pwd")
                   // 认证成功处理器
                   .successHandler(customAuthenticationSuccessHandler)
                   // 认证失败处理器
                   .failureHandler(customAuthenticationFailureHandler)
                   .and()
                   .authorizeRequests() // 认证请求
                   .antMatchers(securityProperties.getAuthentication().getLoginPage(), "/code/image","/mobile/page", "/code/mobile").permitAll()
                   .anyRequest().authenticated() // 所有进入应用的HTTP请求都要进行认证
                   .and()
                   .rememberMe() //记住我
                   //保持登陆信息
                   .tokenRepository(jdbcTokenRepository())
                   //保持登陆时间
                   .tokenValiditySeconds(60*60*24*7)
           ;
           //将手机认证添加到过滤器链上
           http.apply(mobileAuthenticationConfig);
       }
   
       /**
        * 针对静态资源放行
        */
       @Override
       public void configure(WebSecurity web) {
           web.ignoring().antMatchers(securityProperties.getAuthentication().getStaticPaths());
       }
   
   
       /**
        * 针对静态资源放行
        */
   //    @Override
   //    public void configure(WebSecurity web) {
   //        web.ignoring().antMatchers("/dist/**", "/modules/**", "/plugins/**");
   //    }
   }
   
   ```

### 6.3.11 编译报错未知的枚举常量

**问题：** Warning:java: 未知的枚举常量 javax.annotation.meta.When.MAYBE

**解决方法：**`原因是找不到默认的 javax.annotation.meta.When的类文件，缺少对应第三方依赖包，添加对应依赖包即可。`

```xml
<dependency>
	<groupId>com.google.code.findbugs</groupId>
	<artifactId>annotations</artifactId>
	<version>3.0.1</version>
</dependency>
```

### 6.3.12 重构失败处理器回到手机登录页

验证码认证失败后，会回到用户名密码登录页 ，原因是在失败处理器写死了。应该动态的重写向回上一次请求路 径。

```java
@Component("customAuthenticationFailureHandler")
//public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {
public class CustomAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    /**
     * @param exception 认证失败时抛出异常
     */
//    @Override
//    public void onAuthenticationFailure(HttpServletRequest request,
//            HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
//        // 认证失败响应JSON字符串，
//        Result result = Result.build(HttpStatus.UNAUTHORIZED.value(), exception.getMessage());
//        response.setContentType("application/json;charset=UTF-8");
//        response.getWriter().write(result.toJsonString());
//    }

    @Autowired
    SecurityProperties securityProperties;

    /**
     * @param exception 认证失败时抛出异常
     */
    @Override
    public void onAuthenticationFailure(HttpServletRequest request,
                                        HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        if(LoginResponseType.JSON.equals(securityProperties.getAuthentication().getLoginType())) {
            // 认证失败响应JSON字符串，
            Result result = Result.build(HttpStatus.UNAUTHORIZED.value(), exception.getMessage());
            response.setContentType("application/json;charset=UTF-8");
            response.getWriter().write(result.toJsonString());
        }else {
            // 重写向回认证页面，注意加上 ?error
            // super.setDefaultFailureUrl(securityProperties.getAuthentication().getLoginPage()+"?error");
            // 获取上一次请求路径
            String referer = request.getHeader("Referer");
            logger.info("referer:" + referer);
            String lastUrl = StringUtils.substringBefore(referer,"?");
            logger.info("上一次请求的路径 ：" + lastUrl);
            super.setDefaultFailureUrl(lastUrl+"?error");
            super.onAuthenticationFailure(request, response, exception);
        }
    }
}

```

## 6.4 实现手机登录RememberMe功能

### 6.4.1 分析实现 

1. 在 UsernamePasswordAuthenticationFilter 拥有一个 RememberMeServices 的引用，其实这个接收引用的是 其父抽象类 AbstractAuthenticationProcessingFilter 提供的 setRememberMeServices 方法。 
2. 而在实现手机短信验证码登录时，我们自定了一个  MobileAuthenticationFilter  也一样的继承 了  AbstractAuthenticationProcessingFilter 它，我们只要向其 setRememberMeServices 方法手动注入一 个 RememberMeServices 实例即可。

### 6.4.2 编码实现

1. 在自定义的 `com.liuurick.security.authentication.mobile.MobileAuthenticationConfig` 中 向  `MobileAuthenticationFilter` 注入 `RememberMeServices` 实例，该实例从共享对象中就可以获取到。

   ```java
   // 为了实现手机登录也拥有记住我的功能,将RememberMeServices传入
   smsCodeAuthenticationFilter.setRememberMeServices(
   	http.getSharedObject(RememberMeServices.class));
   ```

2. 检查 记住我 的 input 标签的 name="remember-me"

3. MobileAuthenticationConfig 完整代码

### 6.4.3 测试

1. 重启项目 
2. 访问 http://localhost/mobile/page 输入手机号与验证码, 勾选 记住我  , 点击登录
3. 查看数据库中 persistent_logins 表的记录
4. 关闭浏览器, 再重新打开浏览器访问 http://localhost/index , 发现会跳转回用户名密码登录页, 而正常应该勾选了 记住我  , 这一步应该是可以正常访问的.

**上面要求认证的原因是:** 

数据库中 username 为 手机号 16888888888, 当你访问 http://localhost/index 默认RememberMeServices 是调 用 CustomUserDetailsService  通过用户名查询, 而当前在 CustomUserDetailsService 判断了用户名为 meng 才通过认证, 而此时传入的用户名是 16888888888 , 所以查询不到 16888888888 用户数据

**解决方案:** 

1. 数据库中的 persistent_logins 表为什么存储的是手机号? 

   原因是当前在 MobileUserDetailsService 中返回的 User 对象中的 username 属性设置的是手机号 mobile, 而应该设置这个手机号所对应的那个用户名. 比如当前username 的值写死为 liubin

   ```java
   @Component("mobileUserDetailsService")
   @Slf4j
   public class MobileUserDetailsService implements UserDetailsService {
   
       @Override
       public UserDetails loadUserByUsername(String mobile) throws UsernameNotFoundException {
           log.info("请求的手机号是：" + mobile);
           // 1. 通过手机号查询用户信息
           // 2. 如果有用户信息，则再获取权限资源
           // 3. 封装用户信息
   
           return new User("liubin", "", true, true, true, true,
                   AuthorityUtils.commaSeparatedStringToAuthorityList("ADMIN"));
       }
   }
   ```

2. 勾选 `记住我` 重新登录，发现表中的username字段值就是 liubin

3. 关闭浏览再打开访问http://localhost/ 就无需手动登录认证了。 因为默认采用的 `CustomUserDetailsService` 查询可查询到用户名为 liubin 的信息，即认证通过。

## 6.5 获取当前用户认证信息

### 6.5.1 概要 

在任意地方（Controller/Service等），通过 `SecurityContextHolder` 类获取 `getContext` 上下文， `getAuthentication` 获取当前用户认证信息 , getPrincipal 获取 UserDetails 。

### 6.5.2 三种获取方式 

重构 `security-web` 模块的 `com.liuurick.web.controller.MainController`

方式一：

```java
	@RequestMapping({"/index", "/", ""})
    public String index(Map<String, Object> map) {
        // 方式1: 获取登录用户信息
        Object principal =
                SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        if(principal != null && principal instanceof UserDetails) {
            UserDetails userDetails = (UserDetails) principal;
            map.put("username", userDetails.getUsername());
        }
        return "index";
    }
```

方式二：

```java
	/**
     * 获取当前登录用户信息, 方式2 注入 Authentication
     * @return
     */
    @RequestMapping("/user/info")
    @ResponseBody
    public Object userInfo(Authentication authentication) {
        return authentication.getPrincipal();
    }
```

方式三：

```java
	/**
     * 获取当前登录用户信息, 方式3 注入 UesrDetails
     * @return
     */
    @RequestMapping("/user/info2")
    @ResponseBody
    public Object userInfo2(@AuthenticationPrincipal UserDetails userDetails) {
        return userDetails;
    }
```

### 6.5.3 重构左侧菜单显示用户名

打开 fragments\main-sidebar.html 页面 `Ctrl + F `搜索: 梦老师  , 在标签上使用 th:text="${username}" 获取用户名

```html
<div class="info">
	<a th:text="${username}" href="#" class="d-block">梦老师</a>
</div>
```

## 6.6 重构实现路径可配置

1. 在 application.yml 添加四个属性 `imageCodeUrl` `mobileCodeUrl` `mobilePage` `tokenValiditySeconds`

   ```yml
   imageCodeUrl: /code/image # 获取图形验证码 url
   mobileCodeUrl: /code/mobile # 发送手机验证码 url
   mobilePage: /mobile/page # 前往手机登录页面地址
   tokenValiditySeconds: 604800 # 记住我有效时长，单位秒， 注意不要用乘法*，会被认为字符串
   ```

2. 在 `AuthenticationProperties` 添加以下属性, 在类上加了 `@Data` 注解, 把 `setter / getter` 方法全部删除

   ```java
   @Data
   public class AuthenticationProperties {
   
       private String loginPage = "/login/page";
       private String loginProcessingUrl = "/login/form";
       private String usernameParameter = "name";
       private String passwordParameter = "pwd";
       private String[] staticPaths = {"/dist/**", "/modules/**", "/plugins/**"};
   
   
       /**
        * 登录成功后响应 JSON , 还是重定向
        * 如果application.yml 中没有配置，则取此初始值 REDIRECT
       */
       private LoginResponseType loginType = LoginResponseType.REDIRECT;
   
       public LoginResponseType getLoginType() {
           return loginType;
       }
       public void setLoginType(LoginResponseType loginType) {
           this.loginType = loginType;
       }
   
       /**
        * 获取图形验证码 url
        */
       private String imageCodeUrl = "/code/image";
       /**
        * 发送手机验证码 url
        */
       private String mobileCodeUrl = "/code/mobile";
       /**
        * 前往手机登录页面地址
        */
       private String mobilePage = "/mobile/page";
       /**
        * 记住我有效时长
        */
       private Integer tokenValiditySeconds = 60*60*24*7;
   
   }
   ```


# 7 Session 会话管理与Redis搭建Session集群

## 7.1 配置 Session 会话超时

### 7.1.1 application.yml 配置超时时间

```yml
server:
  port: 8080
  servlet:
    session:
      timeout: 1m # session超时时间默认30m (30分钟)，至少设置1分钟
```





### 7.1.2 底层源码分析 

TomcatServletWebServerFactory#configureSession

```java
private void configureSession(Context context) {
	// 获取超时时长
	long sessionTimeout = this.getSessionTimeoutInMinutes();
}
private long getSessionTimeoutInMinutes() {
// Math.max(sessionTimeout.toMinutes(), 1L) 至少1分钟 , 配置小于1分钟也取1分钟
	Duration sessionTimeout = this.getSession().getTimeout();
	return this.isZeroOrLess(sessionTimeout) ? 0L : Math.max(sessionTimeout.toMinutes(), 1L);
}
```



## 7.2 自定义 Session 失效处理

默认情况下，当 session 失效后会请求回认证页面。我们可以自定义 session 失效后，响应不同的结果。 

1. 添加session失效处理：com.liuurick.security.config.SpringSecurityConfig

   ```java
   注入session失败策略
   @Autowired
   private InvalidSessionStrategy invalidSessionStrategy;
   配置 session 管理
   .and()
   .sessionManagement()
   .invalidSessionStrategy(invalidSessionStrategy) // session失效后处理逻辑
   ```

2. 在 security-core 工程中创建 com.liuurick.security.authentication.session.CustomInvalidSessionStrategy

   ```java
   public class CustomInvalidSessionStrategy implements InvalidSessionStrategy {
       @Override
       public void onInvalidSessionDetected(HttpServletRequest request,
                                            HttpServletResponse response) throws IOException {
           // 将浏览器的sessionid清除，不关闭浏览器cookie不会被删除，一直请求都提示：Session失效
           cancelCookie(request, response);
           Result result = Result.build(
                   HttpStatus.UNAUTHORIZED.value(), "登录已超时, 请重新登录");
           response.setContentType("application/json;charset=utf-8");
           response.getWriter().write(result.toJsonString());
       }
   
       /**
        *  参考记住我功能的 AbstractRememberMeServices 代码
         */
       protected void cancelCookie(HttpServletRequest request, HttpServletResponse response) {
           Cookie cookie = new Cookie("JSESSIONID", null);
           cookie.setMaxAge(0);
           cookie.setPath(getCookiePath(request));
           response.addCookie(cookie);
       }
   
       private String getCookiePath(HttpServletRequest request) {
           String contextPath = request.getContextPath();
           return contextPath.length() > 0 ? contextPath : "/";
       }
   }
   ```

3. 注入Session失效策略实例, 在 com.liuurick.security.config.SecurityConfigBean 添加方法注入方便web应用覆盖此实现,定义不同逻辑

   ```java
   @Configuration
   public class SecurityConfigBean {
   
       @Bean
       @ConditionalOnMissingBean(SessionInformationExpiredStrategy.class)
       public SessionInformationExpiredStrategy sessionInformationExpiredStrategy() {
           return new CustomSessionInformationExpiredStrategy();
       }
   
       /**
        * 注入Session失效策略实例
        * @return
        */
       @Bean
       @ConditionalOnMissingBean(InvalidSessionStrategy.class)
       public InvalidSessionStrategy invalidSessionStrategy() {
           return new CustomInvalidSessionStrategy();
       }
   
       /**
        * @ConditionalOnMissingBean(SmsSend.class)
        * 默认情况下，采用的是SmsCodeSender实例 ，
        * 但是如果容器当中有其他的SmsSend类型的实例，
        * 那当前的这个SmsCodeSender就失效 了
        * @return
        */
       @Bean
       @ConditionalOnMissingBean(SmsSend.class)
       public SmsSend smsSend() {
           return new SmsCodeSender();
       }
   }
   ```

4. 测试 

   1. 先登录
   2. 再重启项目让session失效 
   3. 再访问: 提示超时

## 7.3 用户只允许一个地方登录 

只允许一个用户在一个地方登录，也是每个用户在系统中只能有一个Session。 

### 7.3.1 情景一 

说明 如果同一用户在第2个地方登录，则将第1个踢下线。 

实现步骤 重构 com.liuurick.security.config.SpringSecurityConfig

```java
注入session失败策略
@Autowired
private InvalidSessionStrategy invalidSessionStrategy;
@Autowired
private SessionInformationExpiredStrategy sessionInformationExpiredStrategy;
配置 Session 管理
.and()
.sessionManagement()
.invalidSessionStrategy(invalidSessionStrategy) // session失效后处理逻辑
.maximumSessions(1) // 每个用户在系统中的最大session数
.expiredSessionStrategy(sessionInformationExpiredStrategy)// 当用户达到最大session数后，则调用此处的实现
```

自定义 SessionInformationExpiredStrategy 实现类来定制策略

```java
public class CustomSessionInformationExpiredStrategy implements SessionInformationExpiredStrategy {
    @Autowired
    CustomAuthenticationFailureHandler customAuthenticationFailureHandler;

    @Override
    public void onExpiredSessionDetected(SessionInformationExpiredEvent event) throws IOException {
        // 1.获取用户名
        UserDetails userDetails =
                (UserDetails)event.getSessionInformation().getPrincipal();
        AuthenticationException exception =
                new AuthenticationServiceException(String.format("[%s]用户在另外一台电脑登录，您已被下线",
                        userDetails.getUsername()));
        try {
            event.getRequest().setAttribute("toAuthentication", true);
            customAuthenticationFailureHandler.onAuthenticationFailure(
                    event.getRequest(), event.getResponse(), exception);
        } catch (ServletException e) {
            e.printStackTrace();
        }
    }
}
```

重构 com.liuurick.security.authentication.CustomAuthenticationFailureHandler 处理器: 如果isAuthentication值为true，说明是session数量超过，则回到 /login/page

```java
String referer = request.getHeader("Referer");
logger.info("referer:" + referer);

// 如果下面有值,则认为是多端登录,直接返回一个登录地址
Object toAuthentication = request.getAttribute("toAuthentication");
String lastUrl = toAuthentication != null ? securityProperties.getAuthentication().getLoginPage()
    : StringUtils.substringBefore(referer,"?");

logger.info("上一次请求的路径 ：" + lastUrl);
super.setDefaultFailureUrl(lastUrl+"?error");
super.onAuthenticationFailure(request, response, exception);
```

com.liuurick.security.config.SecurityConfigBean 添加方法注入SessionInformationExpiredStrategy实现 方便web应用覆盖此实现,定义不同逻辑

```java
@Bean
@ConditionalOnMissingBean(SessionInformationExpiredStrategy.class)
public SessionInformationExpiredStrategy sessionInformationExpiredStrategy() {
	return new CustomSessionInformationExpiredStrategy();
}
```

测试： 

1. 谷歌浏览器用户名密码登录 
2. 再QQ浏览器用户名密码登录 
3. 回到谷歌浏览器刷新请求，发现回到登录页面，提示被下线



### 7.3.2 情景二

**说明** 

如果同一用户在第2个地方登录时，则不允许他二次登录。 

**实现步骤** 

1. 在 SpringSecurityConfig 添加 maxSessionsPreventsLogin(true) 后不允许二次登录

   ```java
   .and()
   .sessionManagement()
   .invalidSessionStrategy(invalidSessionStrategy) // session失效后处理逻辑
   .maximumSessions(1) // 每个用户在系统中的最大session数
   .expiredSessionStrategy(sessionInformationExpiredStrategy)// 当用户达到最大session数后，则调用此处的实现
   .maxSessionsPreventsLogin(true)// 当一个用户达到最大session数，则不允许后面进行登录
   ```

2. 测试 

   1. 谷歌浏览器用户名密码登录 
   2. 再QQ浏览器用户名密码登录，发现不允许登录



### 7.3.3 解决同一用户的手机重复登录问题 

问题描述： 使用了 admin 用户名登录后，还可以使用这个用户的手机号登录

正常应该是同一个用户，系统中只能用用户名或手机号登录一次。

**解决问题**： 原因是 MobileAuthenticationFilter 继承的类 AbstractAuthenticationProcessingFilter 中，默认使用 了 NullAuthenticatedSessionStrategy 实例管理 Session，我们应该指定用户名密码过滤器中所使用的那个 SessionAuthenticationStrategy 实现类 CompositeSessionAuthenticationStrategy 。 

在 com.liuurick.security.authentication.mobile.MobileAuthenticationConfig 指定即可解决：

```java
// session 会话管理功能
mobileAuthenticationFilter.setSessionAuthenticationStrategy(http.getSharedObject(SessionAuthenticationStrategy.class));
```

## 7.4 Redis 实现 Session 高可用集群

### 7.4.1 概述 

如果采用默认的单机版 Session 存储身份信息时, 一旦这台服务器挂了就没法使用. 那我们可以为项目搭建集群, 部署到AB 两台服务器上, 但是部署到AB 两台服务器上, Session就没有保证一致, 假设 user1 第1次登录访问的是 A服务器, 后面访问资源时请求到了B服务器,而当前B并没有存储用户session,这样 又会要求用户再次登录 , 所以目前方式也是不可以的,不能让用户到两台机器上各认证一次,应该在一台机器上认证即可. 不管用户访问哪台服务器, 我们将用户的session 都放到 redis , 后面请求都从 redis 获取 session, 这样可以保证一 致性.

### 7.4.2 操作步骤 

所支持的存储方式参见: `org.springframework.boot.autoconfigure.session.StoreType `

1. 以 管理员身份运行 Redis-x64-3.2.100\redis-server.exe 

2. security-core\pom.xml 添加对 redis 的支持

   ```xml
   <!--采用redis来管理session-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.session</groupId>
       <artifactId>spring-session-data-redis</artifactId>
   </dependency>
   ```

3. application.yml 指定存储方式 redis

   ```yml
   spring:
     thymeleaf:
       cache: false #关闭thymeleaf缓存
     # 数据源配置
     datasource:
       username: root
       password: 123456
       url: jdbc:mysql://127.0.0.1:3306/study-security?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8
       #mysql8版本以上驱动包指定新的驱动类
       driver-class-name: com.mysql.cj.jdbc.Driver
     session:
       store-type: redis
   ```

4. 重启项目, 进行登录 

5. 管理员身份运行客户端 Redis-x64-3.2.100\redis-cli.exe 执行 keys * 命令查看保存的数据



### 7.4.3 指定 Cookie 中保存SessionID名称

**说明** 

1. 默认情况下, 浏览器的 Cookie 中保存 SessionID 名称是 `JSESSIONID` ,

2. 当使用redis保存session信息后, 浏览器的 Cookie 中保存 SessionID 名称是 `SESSION` ,

这样会导致当session失效后，在 CustomInvalidSessionStrategy 中只将 JSESSIONID 清除是不行的，不管刷新请求多少次，都是提示：登录已超时，请重新登录。



**现象** 

1. 先登录 
2. redis 使用 flushall 命令清除所有数据，即将session值失效 
3. 再刷新多次浏览器，一直提示：登录已超时，请重新登录



**解决** 

统一指定浏览器中 Cookie 保存的 SessionID 名称为 JSESSIONID

```yml
erver:
 port: 80
 servlet:
   session:
     timeout: 10m # session会话超时时间，默认情况 下是30分钟（m）,不能小于1分钟
     cookie:
       name: JSESSIONID # 统一指定浏览器中 Cookie  保存的SessionID名称
```

如果想关闭 redis 存储 session, 作如下配置：

```yml
spring:
 session:
   store-type: none
```

未关闭，则每次要启动redis服务器， Redis-x64-3.2.100\redis-server.exe



## 7.5 退出系统 

### 7.5.1 退出系统默认配置

1. 默认请求 /logout 即可退出系统，在 templates/fragments/main-header.html 指定退出URL ，如下：

   ```html
   <a th:href="@{/logout}" href="login.html" class="dropdown-item" >
       <i class="fa fa-sign-out mr-2"></i> 退出系统
   </a>
   ```

2. 点击 退出系统 会报404 

会报-报404原因: 

默认情况下 /logout 必须使用POST提交才可以起作用, 原因是防止 csrf (跨站请求伪造 ) 功能默认开启了 

**解决问题:** 

在 SpringSecurityConfig 关闭 csrf , 即可使用 get 方式请求 /logout

```
; // 注意不要少了分号
http.csrf().disable(); // 关闭跨站请求伪造
```

退出后的效果 ： `/login/page?error`

3. 底层默认退出处理操作： 
   1. 清除浏览器 remember-me 的 Cookie、通过用户名删除 remember-me 数据库记录 
   2. 将当前用户的 Session 失效，清空当前用户的 Authentication 
   3. 重写向到登录页面，带上error请求参数，地址： /login/page?error



### 7.5.3 解决退出不允许再次登录

**现象** 

配置了 `.maxSessionsPreventsLogin(true)` 开启了前面已登录，不允许再重复登录 上面默认情况，如果登录后，然后请求 `/logout` 退出，再重新登录时，会提示不能重复登录。

**解决方案** 

添加一个退出处理器 `com.liuurick.security.authentication.session.CustomLogoutHandler` ,将用户信息从缓存中清除







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





# 9 整合 MyBatis-Plus 实现数据库动态认证

## 9.1 什么是RBAC模型

基于RBAC模型（Role-Based Access Control）角色的权限访问控制

- 用户表( sys_user )：保存用户信息
- 角色表 ( sys_role )：保存角色信息
- 权限表 ( sys_permission )：保存系统资源信息。如：菜单、按钮 和对应 URL
  - 它们的关系 ：用户表与角色表是 多对多关系 ，角色表与资源表是多对多关系。
- 用户角色关系表（sys_user_role）：用于维护用户和角色的关系
- 角色资源关系表（sys_role_permission）：用于维护角色与资源的关系

![image-20210418170832529](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210418170832529.png)

## 9.2 整合 MyBatis-Plus 和 Druid

MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具，在 MyBatis 的基础上**只做增强不做改变**，为简化开发、提 高效率而生。 

> MyBatis 参考：https://mybatis.org/mybatis-3/zh/index.html 
>
> MyBatis-Plus 参考：https://mp.baomidou.com/

### 9.2.1 创建数据库 

```sql
-- ----------------------------
-- Table structure for sys_permission 
-- ----------------------------
DROP TABLE IF EXISTS `sys_permission`;
CREATE TABLE `sys_permission` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '权限 ID',
  `parent_id` bigint(20) DEFAULT NULL COMMENT '父权限 ID (0为顶级菜单)',
  `name` varchar(64) NOT NULL COMMENT '权限名称',
  `code` varchar(64) DEFAULT NULL COMMENT '授权标识符',
  `url` varchar(255) DEFAULT NULL COMMENT '授权路径',
  `type` int(2) NOT NULL DEFAULT '1' COMMENT '类型(1菜单，2按钮)',
  `icon` varchar(200) DEFAULT NULL COMMENT '图标',
  `remark` varchar(200) DEFAULT NULL COMMENT '备注',
  `create_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=33 DEFAULT CHARSET=utf8 COMMENT='权限表';

-- ----------------------------
-- Records of sys_permission
-- ----------------------------
INSERT INTO `sys_permission` VALUES ('11', '0', '首页', 'sys:index', '/', '1', 'fa fa-dashboard', '', '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('17', '0', '系统管理', 'sys:manage', null, '1', 'fa fa-cogs', null, '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('18', '17', '用户管理', 'sys:user', '/user', '1', 'fa fa-users', null, '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('19', '18', '列表', 'sys:user:list', '', '2', '', '员工列表', '2023-08-08 11:11:11', '2023-08-08 11:11:11');
INSERT INTO `sys_permission` VALUES ('20', '18', '新增', 'sys:user:add', '', '2', '', '新增用户', '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('21', '18', '修改', 'sys:user:edit', '', '2', '', '修改用户', '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('22', '18', '删除', 'sys:user:delete', '', '2', '', '删除用户', '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('23', '17', '角色管理', 'sys:role', '/role', '1', 'fa fa-user-secret', null, '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('24', '23', '列表', 'sys:role:list', null, '2', null, '角色列表', '2023-08-08 11:11:11', '2023-08-08 11:11:11');
INSERT INTO `sys_permission` VALUES ('25', '23', '新增', 'sys:role:add', '', '2', '', '新增角色', '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('26', '23', '修改', 'sys:role:edit', '', '2', '', '修改角色', '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('27', '23', '删除', 'sys:role:delete', '', '2', '', '删除角色', '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('28', '17', '权限管理', 'sys:permission', '/permission', '1', 'fa fa-cog', null, '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('29', '28', '列表', 'sys:permission:list', null, '2', null, '权限列表', '2023-08-08 11:11:11', '2023-08-08 11:11:11');
INSERT INTO `sys_permission` VALUES ('30', '28', '新增', 'sys:permission:add', '', '2', null, '新增权限', '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('31', '28', '修改', 'sys:permission:edit', '', '2', null, '修改权限', '2023-08-08 11:11:11', '2023-08-09 15:26:28');
INSERT INTO `sys_permission` VALUES ('32', '28', '删除', 'sys:permission:delete', '', '2', '', '删除权限', '2023-08-08 11:11:11', '2023-08-09 15:26:28');

-- ----------------------------
-- Table structure for sys_role
-- ----------------------------
DROP TABLE IF EXISTS `sys_role`;
CREATE TABLE `sys_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '角色 ID',
  `name` varchar(64) NOT NULL COMMENT '角色名称',
  `remark` varchar(200) DEFAULT NULL COMMENT '角色说明',
  `create_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8 COMMENT='角色表';

-- ----------------------------
-- Records of sys_role
-- ----------------------------
INSERT INTO `sys_role` VALUES ('9', '超级管理员', '拥有所有的权限', '2023-08-08 11:11:11', '2023-08-08 11:11:11');
INSERT INTO `sys_role` VALUES ('10', '普通管理员', '拥有查看权限', '2023-08-08 11:11:11', '2023-08-08 11:11:11');

-- ----------------------------
-- Table structure for sys_role_permission
-- ----------------------------
DROP TABLE IF EXISTS `sys_role_permission`;
CREATE TABLE `sys_role_permission` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键 ID',
  `role_id` bigint(20) NOT NULL COMMENT '角色 ID',
  `permission_id` bigint(20) NOT NULL COMMENT '权限 ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=26 DEFAULT CHARSET=utf8 COMMENT='角色权限表';

-- ----------------------------
-- Records of sys_role_permission
-- ----------------------------
INSERT INTO `sys_role_permission` VALUES ('1', '9', '11');
INSERT INTO `sys_role_permission` VALUES ('2', '9', '17');
INSERT INTO `sys_role_permission` VALUES ('3', '9', '18');
INSERT INTO `sys_role_permission` VALUES ('4', '9', '19');
INSERT INTO `sys_role_permission` VALUES ('5', '9', '20');
INSERT INTO `sys_role_permission` VALUES ('6', '9', '21');
INSERT INTO `sys_role_permission` VALUES ('7', '9', '22');
INSERT INTO `sys_role_permission` VALUES ('8', '9', '23');
INSERT INTO `sys_role_permission` VALUES ('9', '9', '24');
INSERT INTO `sys_role_permission` VALUES ('10', '9', '25');
INSERT INTO `sys_role_permission` VALUES ('11', '9', '26');
INSERT INTO `sys_role_permission` VALUES ('12', '9', '27');
INSERT INTO `sys_role_permission` VALUES ('13', '9', '28');
INSERT INTO `sys_role_permission` VALUES ('14', '9', '29');
INSERT INTO `sys_role_permission` VALUES ('15', '9', '30');
INSERT INTO `sys_role_permission` VALUES ('16', '9', '31');
INSERT INTO `sys_role_permission` VALUES ('17', '9', '32');
INSERT INTO `sys_role_permission` VALUES ('18', '10', '11');
INSERT INTO `sys_role_permission` VALUES ('19', '10', '17');
INSERT INTO `sys_role_permission` VALUES ('20', '10', '18');
INSERT INTO `sys_role_permission` VALUES ('21', '10', '19');
INSERT INTO `sys_role_permission` VALUES ('22', '10', '23');
INSERT INTO `sys_role_permission` VALUES ('23', '10', '24');
INSERT INTO `sys_role_permission` VALUES ('24', '10', '28');
INSERT INTO `sys_role_permission` VALUES ('25', '10', '29');

-- ----------------------------
-- Table structure for sys_user
-- ----------------------------
DROP TABLE IF EXISTS `sys_user`;
CREATE TABLE `sys_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '用户 ID',
  `username` varchar(50) NOT NULL COMMENT '用户名',
  `password` varchar(64) NOT NULL COMMENT '密码，加密存储, admin/1234',
  `is_account_non_expired` int(2) DEFAULT '1' COMMENT '帐户是否过期(1 未过期，0已过期)',
  `is_account_non_locked` int(2) DEFAULT '1' COMMENT '帐户是否被锁定(1 未过期，0已过期)',
  `is_credentials_non_expired` int(2) DEFAULT '1' COMMENT '密码是否过期(1 未过期，0已过期)',
  `is_enabled` int(2) DEFAULT '1' COMMENT '帐户是否可用(1 可用，0 删除用户)',
  `nick_name` varchar(64) DEFAULT NULL COMMENT '昵称',
  `mobile` varchar(20) DEFAULT NULL COMMENT '注册手机号',
  `email` varchar(50) DEFAULT NULL COMMENT '注册邮箱',
  `create_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`) USING BTREE,
  UNIQUE KEY `mobile` (`mobile`) USING BTREE,
  UNIQUE KEY `email` (`email`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8 COMMENT='用户表';

-- ----------------------------
-- Records of sys_user
-- ----------------------------
INSERT INTO `sys_user` VALUES ('9', 'admin', '$2a$10$rDkPvvAFV8kqwvKJzwlRv.i.q.wz1w1pz0SFsHn/55jNeZFQv/eCm', '1', '1', '1', '1', '梦学谷', '16888888888', 'mengxuegu888@163.com', '2023-08-08 11:11:11', '2019-12-16 10:25:53');
INSERT INTO `sys_user` VALUES ('10', 'test', '$2a$10$rDkPvvAFV8kqwvKJzwlRv.i.q.wz1w1pz0SFsHn/55jNeZFQv/eCm', '1', '1', '1', '1', '测试', '16888886666', 'test11@qq.com', '2023-08-08 11:11:11', '2023-08-08 11:11:11');

-- ----------------------------
-- Table structure for sys_user_role
-- ----------------------------
DROP TABLE IF EXISTS `sys_user_role`;
CREATE TABLE `sys_user_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键 ID',
  `user_id` bigint(20) NOT NULL COMMENT '用户 ID',
  `role_id` bigint(20) NOT NULL COMMENT '角色 ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8 COMMENT='用户角色表';

-- ----------------------------
-- Records of sys_user_role
-- ----------------------------
INSERT INTO `sys_user_role` VALUES ('1', '9', '9');
INSERT INTO `sys_user_role` VALUES ('2', '10', '10');

-- 获取id=9的用户权限信息
SELECT
	DISTINCT p.id,	p.parent_id, p.name, p.code, p.url, p.type,
	p.icon, p.remark, p.create_date, p.update_date
FROM
  sys_user AS u
  LEFT JOIN sys_user_role AS ur
	ON u.id = ur.user_id
  LEFT JOIN sys_role AS r
	ON r.id = ur.role_id
  LEFT JOIN sys_role_permission AS rp
	ON r.id = rp.role_id
  LEFT JOIN sys_permission AS p
	ON p.id = rp.permission_id
WHERE u.id = 9
ORDER BY p.id
```

### 9.2.2 添加依赖

```xml
<!--mybatis-plus启动器-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
</dependency>
<!--druid连接池-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
</dependency>
```

### 9.2.3 配置数据源 application.yml

重构 security-web\resources\application.yml

```
spring:
 session:
   store-type: none # session存储方式采用 redis
 redis: # 如果是本地redis可不配置
   port: 6379
 thymeleaf:
   cache: false #关闭thymeleaf缓存
# 数据源配置
 datasource:
   username: root
   password: 123456
   url: jdbc:mysql://127.0.0.1:3306/study-security?
serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8
    #mysql8版本以上驱动包指定新的驱动类
   driver-class-name: com.mysql.cj.jdbc.Driver
   type: com.alibaba.druid.pool.DruidDataSource
    #   数据源其他配置, 在 DruidConfig配置类中手动绑定
   initialSize: 8
   minIdle: 5
   maxActive: 20
   maxWait: 60000
   timeBetweenEvictionRunsMillis: 60000
   minEvictableIdleTimeMillis: 300000
   validationQuery: SELECT 1 FROM DUAL
    
mybatis-plus:
  # 指定实体类所有包
 type-aliases-package: com.mengxuegu.web.entities
# 日志级别，会打印sql语句
logging:
 level:
   com.liuurick.web.mapper: debug
server:
 port: 80
 servlet:
   session:
     timeout: 30m # session会话超时时间，默认情况 下是30分钟（m）,不能小于1分钟
     cookie:
       name: JSESSIONID # 指定浏览器Cookie中关于SessionID保存的那个名称

liuurick:
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
           loginType: REDIRECT # 认证之后 响应的类型：JSON/REDIRECT # "/code/image","/mobile/page", "/code/mobile"
           imageCodeUrl: /code/image # 获取图形验证码地址
           mobileCodeUrl: /code/mobile # 发送手机验证码地址
           mobilePage: /mobile/page # 前往手机登录页面
           tokenValiditySeconds: 604800 # 记住我功能有效时长

```



### 9.2.4 创建 DruidConfig

指定Druid数据源，并注入数据源配置。 在 security-web 模块下创建 `com.liuurick.web.config.DruidConfig`

```java
@Configuration
public class DruidConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druid() {
        return new DruidDataSource();
    }
}
```

创建 MybatisPlusConfig 在 security-web 中创建配置类  `com.liuurick.web.config.MybatisPlusConfig`

```java
@EnableTransactionManagement
@MapperScan("com.liuurick.web.mapper")
@Configuration
public class MybatisPlusConfig {
    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        return paginationInterceptor;
    }
}
```

**测试** 

启动项目，查看控制台没有报错则整合成功



## 9.3 编码用户管理 SysUser

### 9.3.1 创建实体类 SysUser

创建 `com.liuurick.web.entities.SysUser` 实体类，实现 `UserDetails` 认证用户信息封装接口 

**注意** ：在 `authorities` 属性上面加上 @TableField(exist = false) ，因为它不是 sys_user 数据库表字段。



### 9.3.2 编写 Mapper 接口

创建 `com.liuurick.web.mapper.SysUserMapper` 继承 `BaseMapper` 接口 实现MyBatis-Plus封装的 BaseMapper 接口,它有很多对 T 表的数据操作方法

```java
**
* 实现MyBatis-Plus封装的 BaseMapper<T> 接口,它有很多对 T 表的数据操作方法
* @Auther: 梦学谷 www.mengxuegu.com
*/
public interface SysUserMapper extends BaseMapper<SysUser> {
}
```

### 9.3.3 编写 Service 类 

创建接口`com.liuurick.web.service.SysUserService`继承 IService 接口 实现 IService 接口，提供了常用更复杂的对 T 数据表的操作，比如：支持 Lambda 表达式，批量删除、自动新增或更新操 作等方法定义一个通过用户名查询用户信息的抽象方法 `findByUsername`

```java
/**
* 实现 IService<T> 接口，提供了常用更复杂的对 T 数据表的操作，
* 比如：支持 Lambda 表达式，批量删除、自动新增或更新操作
* @Auther: 梦学谷 www.mengxuegu.com
*/
public interface SysUserService extends IService<SysUser> {
    /**
     * 通过用户名查询
     * @param username 用户名
     * @return 用户信息
     */
    SysUser findByUsername(String username);
}
```

### 9.3.4 创建实现类 

`com.liuurick.web.service.impl.SysUserServiceImpl` 继承 `ServiceImpl` 类 , 并且实现 `SysUserService` 接口. 

ServiceImpl<M extends BaseMapper<T>, T>是对 IService 接口中方法的实现: 

- 第1个泛型 M 指定继承了 BaseMapper接口的子接口 
- 第2个泛型 T 指定实体类 注意：类上不要少了 @Service baseMapper 引用的就是 SysUserMapper 实例

**注意**：

类上不要少了 @Service 

baseMapper 引用的就是 SysUserMapper 实例

```java
/**
* ServiceImpl<M extends BaseMapper<T>, T> 是对 IService接口中方法的实现
*     第1个泛型 M 指定继承了 BaseMapper接口的子接口
*     第2个泛型 T 指定实体类
* @Auther: 梦学谷 www.mengxuegu.com
*/
@Service
public class SysUserServiceImpl extends ServiceImpl<SysUserMapper, SysUser> implements SysUserService {
    @Override
    public SysUser findByUsername(String username) {
        QueryWrapper<SysUser> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("username", username);
        // baseMapper 对应的是 SysUserMapper 实例
        return baseMapper.selectOne(queryWrapper);
	}
}
```

单元测试类





## 9.4 编码角色管理 SysRole



## 9.5 编码权限管理 SysPermission

## 9.6 数据库动态认证用户名密码

## 9.7 数据库动态认证手机号

## 9.8 重构UserDetailsService实现可复用





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