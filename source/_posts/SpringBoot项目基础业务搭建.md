---
title: SpringBoot项目基础业务搭建
date: 2021-05-29 16:41:14
tags: 业务搭建
categories: 项目总结
---

SpringBoot项目基础模块构建

- 创建SpringBoot项目
- 配置MyBatis
- 整合高级功能
  - 自定义异常
  - 封装R对象
  - 集成Swagger
  - 添加后端验证
  - 抵御XSS攻击
  - Shiro与JWT
  - 精简返回给客户端的异常

<!--more-->

# 1  创建SpringBoot项目

1.利用IDEA的Spring Initilizer可以图形化创建项目

2.填写好Maven项目信息

3.勾选若干依赖库

4.配置SpringBoot项目

- 配置Tomcat
- 配置MySQL
- 配置Redis
- 配置MongoDB
- 。。。



# 2 配置MyBatis

使用MyBatis/MyBatis-Plus

- 创建数据库连接

- 选中数据库，生成MyBatis文件

- 修改yml文件，添加MyBatis配置信息

  这里我使用的是MyBatis-plus

  ```yml
  #MyBatis-plus配置
  mybatis-plus:
    # xml扫描，多个目录用逗号或者分号分隔（告诉 Mapper 所对应的 XML 文件位置）
    mapper-locations: classpath:mapper/*.xml
    # 以下配置均有默认值,可以不设置
    global-config:
      db-config:
        #主键类型 AUTO:"数据库ID自增" INPUT:"用户输入ID",ID_WORKER:"全局唯一ID (数字类型唯一ID)", UUID:"全局唯一ID UUID";
        id-type: auto
        #字段策略 IGNORED:"忽略判断"  NOT_NULL:"非 NULL 判断")  NOT_EMPTY:"非空判断"
        field-strategy: NOT_EMPTY
        #数据库类型
        db-type: MYSQL
    configuration:
      # 是否开启自动驼峰命名规则映射:从数据库列名到Java属性驼峰命名的类似映射
      map-underscore-to-camel-case: true
      # 如果查询结果中包含空值的列，则 MyBatis 在映射的时候，不会映射这个字段
      call-setters-on-nulls: true
      # 这个配置会将执行的sql打印出来，在开发或测试的时候可以用
      log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  ```

  

# 3 日志管理

对于SpringBoot项目，**日志最佳组合**：slf4j+logback（SpringBoot使用）/   slf4j+log4j2

设置日志输出

```yml
logging:
  level:
    root: info
    com.liuurick.emos.wx.db.dao : warn
  pattern:
    console: "%d{HH:mm:ss}  %-5level  %msg%n"
```

logback.xml

>Logback配置与使用：https://liuurick.github.io/2018/10/31/Logback%E9%85%8D%E7%BD%AE%E4%B8%8E%E4%BD%BF%E7%94%A8/

AOP+注解

基于数据库Binlog

ELK日志系统

# 4 整合高级功能

## 4.1 自定义异常

为什么要继承RuntimeException?

- Exception类型的异常必须手动处理

- RuntimeException异常既可以自动处理，也可以手动处理

包含的属性

- 状态码
- 异常信息

```java
@Data
public class CustomException extends RuntimeException{

    private int code = 500;
    private String msg;

    public CustomException(String msg) {
        super(msg);
        this.msg = msg;
    }

    public CustomException(String msg,Throwable e) {
        super(msg,e);
        this.msg = msg;
    }

    public CustomException(int code, String msg) {
        super(msg);
        this.code = code;
        this.msg = msg;
    }
}
```



## 4.2 封装R对象

JavaWeb项目需要统一数据返回格式

- 业务状态码
- 业务消息
- 业务数据



导入`httpcomponents`库

- 定义了很多HTTP状态码
- 免去我们自定义状态码常量



R类继承自HashMap



封装方法

- ok方法
- error方法



```java
public class R extends HashMap<String,Object> {

    public R(){
        put("code", HttpStatus.SC_OK);
        put("msg","success");
    }

    @Override
    public R put(String key, Object value){
        super.put(key,value);
        return this;
    }

    public static R ok(){
        return new R();
    }

    public static R ok(String msg){
        R r=new R();
        r.put("msg",msg);
        return r;
    }

    public static R ok(Map<String,Object> map){
        R r=new R();
        r.putAll(map);
        return r;
    }

    public static R error(int code,String msg){
        R r=new R();
        r.put("code",code);
        r.put("msg",msg);
        return r;
    }

    public static R error(String msg){
        return error(HttpStatus.SC_INTERNAL_SERVER_ERROR,msg);
    }

    public static R error(){
        return error(HttpStatus.SC_INTERNAL_SERVER_ERROR,"未知异常，请联系管理员");
    }
}

```



## 4.3 集成Swagger

项目一般会使用Swagger2，这里我使用最近新出的Knife

1.添加依赖库

```xml
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.9.2</version>
</dependency>
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.9.2</version>
</dependency>
或者
<!--Swagger-bootstrap-->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>2.0.7</version>
</dependency>
```

2.配置Swagger

（ApiInfoBuilder ）定义Swagger页面基本信息

（ApiSelectorBuilder ）哪些类中的方法会出现在Swagger上面

开启对JWT的支持

```
List<ApiKey>：用户需要输入什么参数
AuthorizationScope[]：JWT认证在Swagger中的作用域
List <SecurityReference > ：令牌的作用域
List<SecurityContext >：令牌上下文
```

编写Web接口

TestController

类声明要加上@API注解

Web方法要加上@ApiOperation注解

http://127.0.0.1:8080/emos-wx-api/swagger-ui.html



```java
@Configuration
@EnableSwagger2WebMvc
public class Knife4jConfiguration {

    @Bean(value = "defaultApi2")
    public Docket defaultApi2() {
        Docket docket=new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(new ApiInfoBuilder()
                        .description("在线协同办公小程序RESTful APIs")
                        .termsOfServiceUrl("http://www.xx.com/")
                        .contact("liuurick@gmail.com")
                        .version("1.0")
                        .build())
                //分组名称
                .groupName("1.0版本")
                .select()
                //这里指定Controller扫描包路径
                .apis(RequestHandlerSelectors.basePackage("com.liuurick.emos.wx.controller"))
                .paths(PathSelectors.any())
                .build();

        //开启对JWT的支持
        ApiKey apiKey = new ApiKey("token","token","header");
        List<ApiKey> apiKeyList = new ArrayList<>();
        apiKeyList.add(apiKey);
        docket.securitySchemes(apiKeyList);

        AuthorizationScope scope = new AuthorizationScope("global","accessEverything");
        AuthorizationScope[] scopes = {scope};
        SecurityReference reference = new SecurityReference("token",scopes);
        List refList = new ArrayList();
        refList.add(reference);

        SecurityContext context = SecurityContext.builder().securityReferences(refList).build();
        List cxtList = new ArrayList();
        cxtList.add(context);
        docket.securityContexts(cxtList);

        return docket;
    }
}
```

## 4.4 添加后端验证

1.使用Validation库

2.添加依赖库

3.创建Form类

- 类声明要添加@ApiModel
- 属性声明要添加@ApiModelProperty
- 属性声明要添加验证注解

4.验证数据要使用@Valid注解

## 4.5 抵御XSS攻击

**原因**

XSS攻击通常指的是通过利用网站系统保存数据的漏洞，通过巧妙的方法把恶意指令注入到网页，用户加载网页的时候就会自动执行恶意脚本。

<script>alert('xss')</script>

如果黑客能在你的浏览器上执行javaScript，那么就能窃取Cookie或者Token

导入Hutool依赖

```xml
<dependency>
	<groupId>cn.hutool</groupId>
	<artifactId>hutool-all</artifactId>
	<version>5.4.0</version>
</dependency>
```



对Http请求中的数据转义

- 设置过滤器

- 覆盖Http请求的方法
  - HttpServletRequest是接口，各家服务器厂商会实现它
  - 如果直接继承各厂商的请求父类，那么我们的程序就跟厂商绑定在一起
  - HttpServletRequestWrapper类
    - 使用了装饰器模式
    - 装饰器封装了厂商的Request实现类
    - 只需要覆盖Wrapper类的方法，就能做到覆盖厂商请求对象里方法
  - 创建过滤器，把Request对象传入Wrapper对象
  
  ```java
  import cn.hutool.core.util.StrUtil;
  import cn.hutool.http.HtmlUtil;
  import cn.hutool.json.JSONUtil;
  
  import javax.servlet.ReadListener;
  import javax.servlet.ServletInputStream;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletRequestWrapper;
  import java.io.*;
  import java.nio.charset.Charset;
  import java.util.HashMap;
  import java.util.LinkedHashMap;
  import java.util.Map;
  
  public class XssHttpServletRequestWrapper extends HttpServletRequestWrapper {
  
      public XssHttpServletRequestWrapper(HttpServletRequest request) {
          super(request);
      }
  
      @Override
      public String getParameter(String name) {
          String value = super.getParameter(name);
          if (!StrUtil.hasEmpty(value)) {
              value = HtmlUtil.filter(value);
          }
          return value;
      }
  
      @Override
      public String[] getParameterValues(String name) {
          String[] values = super.getParameterValues(name);
          if (values != null) {
              for (int i = 0; i < values.length; i++) {
                  String value = values[i];
                  if (!StrUtil.hasEmpty(value)) {
                      value = HtmlUtil.filter(value);
                  }
                  values[i] = value;
              }
          }
          return values;
      }
  
      @Override
      public Map<String, String[]> getParameterMap() {
          Map<String, String[]> parameters = super.getParameterMap();
          Map<String, String[]> map = new LinkedHashMap<>();
          if (parameters != null) {
              for (String key : parameters.keySet()) {
                  String[] values = parameters.get(key);
                  for (int i = 0; i < values.length; i++) {
                      String value = values[i];
                      if (!StrUtil.hasEmpty(value)) {
                          value = HtmlUtil.filter(value);
                      }
                      values[i] = value;
                  }
                  map.put(key, values);
              }
          }
          return map;
      }
  
      @Override
      public String getHeader(String name) {
          String value = super.getHeader(name);
          if (!StrUtil.hasEmpty(value)) {
              value = HtmlUtil.filter(value);
          }
          return value;
      }
  
      @Override
      public ServletInputStream getInputStream() throws IOException {
          InputStream in = super.getInputStream();
          StringBuffer body = new StringBuffer();
          InputStreamReader reader = new InputStreamReader(in, Charset.forName("UTF-8"));
          BufferedReader buffer = new BufferedReader(reader);
          String line = buffer.readLine();
          while (line != null) {
              body.append(line);
              line = buffer.readLine();
          }
          buffer.close();
          reader.close();
          in.close();
  
          Map<String, Object> map = JSONUtil.parseObj(body.toString());
          Map<String, Object> resultMap = new HashMap(map.size());
          for (String key : map.keySet()) {
              Object val = map.get(key);
              if (map.get(key) instanceof String) {
                  resultMap.put(key, HtmlUtil.filter(val.toString()));
              } else {
                  resultMap.put(key, val);
              }
          }
          String str = JSONUtil.toJsonStr(resultMap);
          final ByteArrayInputStream bain = new ByteArrayInputStream(str.getBytes());
          return new ServletInputStream() {
              @Override
              public int read() throws IOException {
                  return bain.read();
              }
  
              @Override
              public boolean isFinished() {
                  return false;
              }
  
              @Override
              public boolean isReady() {
                  return false;
              }
  
              @Override
              public void setReadListener(ReadListener listener) {
              }
          };
      }
  
  }
  ```



## 4.6 DES加密敏感信息

DES加密与解密

```java
/**
 * DES是一种对称加密算法，所谓对称加密算法即：加密和解密使用相同密钥的算法。
 * 
 * @author liubin
 *
 */
public class DESUtil {

	private static Key key;
	// 设置密钥key
	private static String KEY_STR = "myKey";
	private static String CHARSETNAME = "UTF-8";
	private static String ALGORITHM = "DES";

	static {
		try {
			// 生成DES算法对象
			KeyGenerator generator = KeyGenerator.getInstance(ALGORITHM);
			// 运用SHA1安全策略
			SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
			// 设置上密钥种子
			secureRandom.setSeed(KEY_STR.getBytes());
			// 初始化基于SHA1的算法对象
			generator.init(secureRandom);
			// 生成密钥对象
			key = generator.generateKey();
			generator = null;
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}

	/**
	 * 获取加密后的信息
	 * 
	 * @param str
	 * @return
	 */
	public static String getEncryptString(String str) {
		// 基于BASE64编码，接收byte[]并转换成String
		BASE64Encoder base64encoder = new BASE64Encoder();
		try {
			// 按UTF8编码
			byte[] bytes = str.getBytes(CHARSETNAME);
			// 获取加密对象
			Cipher cipher = Cipher.getInstance(ALGORITHM);
			// 初始化密码信息
			cipher.init(Cipher.ENCRYPT_MODE, key);
			// 加密
			byte[] doFinal = cipher.doFinal(bytes);
			// byte[]to encode好的String并返回
			return base64encoder.encode(doFinal);
		} catch (Exception e) {
			// TODO: handle exception
			throw new RuntimeException(e);
		}
	}

	/**
	 * 获取解密之后的信息
	 * 
	 * @param str
	 * @return
	 */
	public static String getDecryptString(String str) {
		// 基于BASE64编码，接收byte[]并转换成String
		BASE64Decoder base64decoder = new BASE64Decoder();
		try {
			// 将字符串decode成byte[]
			byte[] bytes = base64decoder.decodeBuffer(str);
			// 获取解密对象
			Cipher cipher = Cipher.getInstance(ALGORITHM);
			// 初始化解密信息
			cipher.init(Cipher.DECRYPT_MODE, key);
			// 解密
			byte[] doFinal = cipher.doFinal(bytes);
			// 返回解密之后的信息
			return new String(doFinal, CHARSETNAME);
		} catch (Exception e) {
			// TODO: handle exception
			throw new RuntimeException(e);
		}
	}

	public static void main(String[] args) {
		System.out.println(getEncryptString("root"));
		System.out.println(getEncryptString("root"));
	}

}
```

yml配置

```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3307/emos?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
      username: WnplV/ietfQ=
      password: WnplV/ietfQ=
      initial-size: 8
      max-active: 16
      min-idle: 8
      max-wait: 60000
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
```

配置类：

```java
@Configuration
public class DataSourceConfiguration {

   @Value("${spring.datasource.druid.driver-class-name}")
   private String jdbcDriver;
   @Value("${spring.datasource.druid.url}")
   private String jdbcUrl;
   @Value("${spring.datasource.druid.username}")
   private String jdbcUsername;
   @Value("${spring.datasource.druid.password}")
   private String jdbcPassword;

   /**
    * 生成与spring-dao.xml对应的bean dataSource
    * 
    * @return
    * @throws PropertyVetoException
    */
   @Bean(name = "dataSource")
   public DruidDataSource createDataSource() throws PropertyVetoException {
      // 生成datasource实例
      DruidDataSource dataSource = new DruidDataSource();
      // 跟配置文件一样设置以下信息
      // 驱动
      dataSource.setDriverClassName(jdbcDriver);
      // 数据库连接URL
      dataSource.setUrl(jdbcUrl);
      // 设置用户名
      dataSource.setUsername(DESUtil.getDecryptString(jdbcUsername));
      // 设置用户密码
      dataSource.setPassword(DESUtil.getDecryptString(jdbcPassword));
      // 配置Druid连接池的私有属性
      // 连接池最大线程数
      dataSource.setMaxActive(16);
      // 连接池最小线程数
      dataSource.setMinIdle(10);
      dataSource.setInitialSize(8);
      dataSource.setTestOnBorrow(false);
      dataSource.setTestOnReturn(false);
      dataSource.setTestWhileIdle(true);
      return dataSource;
   }
}
```

## 4.7 Shiro与JWT

> SpringBoot中整合Shiro+JWT，实现RBAC权限模型，并且自动为Token令牌续期，解决令牌过期的难题。

Shiro简介

- 什么是认证
- 什么是授权
- Shiro靠什么做认证与授权

JWT简介

- 单点系统中的认证方式
- 集群环境中的JWT认证方式



创建JWT工具类

- 导入依赖库
- 生成令牌
  - 密钥
  - 过期时间
  - 用户ID
- 验证令牌的有效性
  - 内容是否有效
  - 是否过期



把令牌封装成认证对象

- Shiro框架的认证需要用到认证对象
- 把令牌字符串做简单的封装



实现认证与授权

- 创建AuthorizingRealm类的子类
- 实现认证与授权的方法



如何设计令牌的刷新机制？

- 为什么要刷新令牌？
  - 令牌一旦生成就保存在客户端
  - 即便用户一直在登陆使用系统，也不会重新生成令牌
  - 令牌到期，用户必须重新登录
  - 令牌应该自动续期
- 双令牌机制
  - 设置长短日期的令牌
  - 短日期令牌失效，就用长日期的令牌
- 缓存令牌机制
  - 令牌缓存到Redis上面
  - 缓存的令牌过期时间是客户端令牌的一倍
  - 如果客户端令牌过期，缓存令牌没有过期，则生成新的令牌
  - 如果客户端令牌过期，缓存令牌也过期了，则用户必须重新登录



创建ThreadLocalToken类

- 该类是用于在过滤器和AOP之间传递Token
- 因为使用了ThreadLocal，所以是线程安全的



创建过滤器

- 判断哪些请求应该被Shiro处理
  - options请求直接放行
    - 提交application/json数据
    - 请求被分成options和post两次
  - 其余所有请求都要被Shiro处理
- 判断Token是真过期还是假过期
  - 真过期，返回提示信息，让用户重新登录
  - 假过期，就生成新的令牌，返回给客户端
- 存储新令牌
  - ThreadLocalToken
  - Redis



创建ShiroConfig

- 把Filter和Realm添加到Shiro框架
- 创建四个对象返回给SpringBoot
  - SecurityManager--用于封装Realm对象
  - ShiroFilterFactoryBean
    - 用于封装Filter对象
    - 设置Filter拦截路径
  - LifecycleBeanPostProcessor
    - 管理Shiro对象生命周期
  - AuthorizationAttributeSourceAdvisor
    - AOP切面类
    - Web方法执行前，验证权限



创建AOP切面类

- 拦截所有的Web方法返回值
- 判断是否刷新生成新令牌
  - 检查ThreadLocal中是否保存令牌
  - 把新令牌绑定到R对象中
  
  

## 4.8 精简返回给客户端的异常

`@ControllerAdvice`可以全局捕获SpringMVC异常

判断异常的类型

- 后端数据验证异常---精简异常内容
- 未授权异常---"你不具有相关权限"
- EmosException---精简异常内容
- 普通异常---"后端执行异常"



全局异常处理：

```java
@Slf4j
@RestControllerAdvice
public class ExceptionAdvice {

    @ResponseBody
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler(Exception.class)
    public String exceptionHandler(Exception e){
        log.error("执行异常",e);
        if(e instanceof MethodArgumentNotValidException){
            MethodArgumentNotValidException exception= (MethodArgumentNotValidException) e;
            return exception.getBindingResult().getFieldError().getDefaultMessage();
        }
        else if(e instanceof CustomException){
            CustomException exception= (CustomException) e;
            return exception.getMsg();
        }
        else if(e instanceof UnauthorizedException){
            return "你不具备相关权限";
        }
        else{
            return "后端执行异常";
        }
    }
}
```