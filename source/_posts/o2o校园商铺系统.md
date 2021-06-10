---
title: o2o校园商铺系统
date: 2019-03-23 10:35:23
tags: 校园商铺系统
categories: 项目开发
---

[TOC]

<!--more-->

# 1 开发准备

# 2 项目设计和框架搭建

## 2-1 系统功能模块划分

系统可以划分为3个系统

- 前端展示系统
  - 头条展示
  - 店铺类别展示
  - 店铺
    - 列表展示
    - 查询
    - 详情
  - 商品
    - 列表展示
    - 查询
    - 详情
- 店家系统
  - Local账户维护
  - 微信账户维护
  - 店铺信息维护
  - 权限验证
  - 商品类别维护
- 超级管理员系统
  - 头条信息维护
  - 店铺类别信息维护
  - 区域信息维护
  - 权限验证
  - 店铺管理
  - 用户管理



## 2-2 实体类设计与表创建

![image-20210531134406946](C:\Users\admin\Desktop\blog\source\images\2019032301.png)

```sql
CREATE DATABASE `o2o` 
USE `o2o`;

DROP TABLE IF EXISTS `tb_area`;
CREATE TABLE `tb_area` (
  `area_id` int(2) NOT NULL AUTO_INCREMENT,
  `area_name` varchar(200) NOT NULL,
  `priority` int(2) NOT NULL DEFAULT '0',
  `create_time` datetime DEFAULT NULL,
  `last_edit_time` datetime DEFAULT NULL,
  PRIMARY KEY (`area_id`),
  UNIQUE KEY `UK_AREA` (`area_name`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS `tb_head_line`;
CREATE TABLE `tb_head_line` (
  `line_id` int(100) NOT NULL AUTO_INCREMENT,
  `line_name` varchar(1000) DEFAULT NULL,
  `line_link` varchar(2000) NOT NULL,
  `line_img` varchar(2000) NOT NULL,
  `priority` int(2) DEFAULT NULL,
  `enable_status` int(2) NOT NULL DEFAULT '0',
  `create_time` datetime DEFAULT NULL,
  `last_edit_time` datetime DEFAULT NULL,
  PRIMARY KEY (`line_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS `tb_local_auth`;
CREATE TABLE `tb_local_auth` (
  `local_auth_id` int(10) NOT NULL AUTO_INCREMENT,
  `user_id` int(10) NOT NULL,
  `username` varchar(128) NOT NULL,
  `password` varchar(128) NOT NULL,
  `create_time` datetime DEFAULT NULL,
  `last_edit_time` datetime DEFAULT NULL,
  PRIMARY KEY (`local_auth_id`),
  UNIQUE KEY `uk_local_profile` (`username`),
  KEY `fk_localauth_profile` (`user_id`),
  CONSTRAINT `fk_localauth_profile` FOREIGN KEY (`user_id`) REFERENCES `tb_person_info` (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS `tb_person_info`;
CREATE TABLE `tb_person_info` (
  `user_id` int(10) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT NULL,
  `profile_img` varchar(1024) DEFAULT NULL,
  `email` varchar(1024) DEFAULT NULL,
  `gender` varchar(2) DEFAULT NULL,
  `enable_status` int(2) NOT NULL DEFAULT '0' COMMENT '0:禁止使用本商城，1:允许使用本商城',
  `user_type` int(2) NOT NULL DEFAULT '1' COMMENT '1:顾客，2:店家，3:超级管理员',
  `create_time` datetime DEFAULT NULL,
  `last_edit_time` datetime DEFAULT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS `tb_product`;
CREATE TABLE `tb_product` (
  `product_id` int(100) NOT NULL AUTO_INCREMENT,
  `product_name` varchar(100) NOT NULL,
  `product_desc` varchar(2000) DEFAULT NULL,
  `img_addr` varchar(2000) DEFAULT '',
  `normal_price` varchar(100) DEFAULT NULL,
  `promotion_price` varchar(100) DEFAULT NULL,
  `priority` int(2) NOT NULL DEFAULT '0',
  `create_time` datetime DEFAULT NULL,
  `last_edit_time` datetime DEFAULT NULL,
  `enable_status` int(2) NOT NULL DEFAULT '0',
  `product_category_id` int(11) DEFAULT NULL,
  `shop_id` int(20) NOT NULL DEFAULT '0',
  PRIMARY KEY (`product_id`),
  KEY `fk_product_procate` (`product_category_id`),
  KEY `fk_product_shop` (`shop_id`),
  CONSTRAINT `fk_product_procate` FOREIGN KEY (`product_category_id`) REFERENCES `tb_product_category` (`product_category_id`),
  CONSTRAINT `fk_product_shop` FOREIGN KEY (`shop_id`) REFERENCES `tb_shop` (`shop_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS `tb_product_category`;
CREATE TABLE `tb_product_category` (
  `product_category_id` int(11) NOT NULL AUTO_INCREMENT,
  `product_category_name` varchar(100) NOT NULL,
  `priority` int(2) DEFAULT '0',
  `create_time` datetime DEFAULT NULL,
  `shop_id` int(20) NOT NULL DEFAULT '0',
  PRIMARY KEY (`product_category_id`),
  KEY `fk_procate_shop` (`shop_id`),
  CONSTRAINT `fk_procate_shop` FOREIGN KEY (`shop_id`) REFERENCES `tb_shop` (`shop_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS `tb_product_img`;
CREATE TABLE `tb_product_img` (
  `product_img_id` int(20) NOT NULL AUTO_INCREMENT,
  `img_addr` varchar(2000) NOT NULL,
  `img_desc` varchar(2000) DEFAULT NULL,
  `priority` int(2) DEFAULT '0',
  `create_time` datetime DEFAULT NULL,
  `product_id` int(20) DEFAULT NULL,
  PRIMARY KEY (`product_img_id`),
  KEY `fk_proimg_product` (`product_id`),
  CONSTRAINT `fk_proimg_product` FOREIGN KEY (`product_id`) REFERENCES `tb_product` (`product_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS `tb_shop`;
CREATE TABLE `tb_shop` (
  `shop_id` int(10) NOT NULL AUTO_INCREMENT,
  `owner_id` int(10) NOT NULL COMMENT '店铺创建人',
  `area_id` int(5) DEFAULT NULL,
  `shop_category_id` int(11) DEFAULT NULL,
  `shop_name` varchar(256) NOT NULL,
  `shop_desc` varchar(1024) DEFAULT NULL,
  `shop_addr` varchar(200) DEFAULT NULL,
  `phone` varchar(128) DEFAULT NULL,
  `shop_img` varchar(1024) DEFAULT NULL,
  `priority` int(3) DEFAULT '0',
  `create_time` datetime DEFAULT NULL,
  `last_edit_time` datetime DEFAULT NULL,
  `enable_status` int(2) NOT NULL DEFAULT '0',
  `advice` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`shop_id`),
  KEY `fk_shop_area` (`area_id`),
  KEY `fk_shop_profile` (`owner_id`),
  KEY `fk_shop_shopcate` (`shop_category_id`),
  CONSTRAINT `fk_shop_area` FOREIGN KEY (`area_id`) REFERENCES `tb_area` (`area_id`),
  CONSTRAINT `fk_shop_profile` FOREIGN KEY (`owner_id`) REFERENCES `tb_person_info` (`user_id`),
  CONSTRAINT `fk_shop_shopcate` FOREIGN KEY (`shop_category_id`) REFERENCES `tb_shop_category` (`shop_category_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS `tb_shop_category`;
CREATE TABLE `tb_shop_category` (
  `shop_category_id` int(11) NOT NULL AUTO_INCREMENT,
  `shop_category_name` varchar(100) NOT NULL DEFAULT '',
  `shop_category_desc` varchar(1000) DEFAULT '',
  `shop_category_img` varchar(2000) DEFAULT NULL,
  `priority` int(2) NOT NULL DEFAULT '0',
  `create_time` datetime DEFAULT NULL,
  `last_edit_time` datetime DEFAULT NULL,
  `parent_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`shop_category_id`),
  KEY `fk_shop_category_self` (`parent_id`),
  CONSTRAINT `fk_shop_category_self` FOREIGN KEY (`parent_id`) REFERENCES `tb_shop_category` (`shop_category_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

DROP TABLE IF EXISTS `tb_wechat_auth`;
CREATE TABLE `tb_wechat_auth` (
  `wechat_auth_id` int(10) NOT NULL AUTO_INCREMENT,
  `user_id` int(10) NOT NULL,
  `open_id` varchar(80) NOT NULL DEFAULT '',
  `create_time` datetime DEFAULT NULL,
  PRIMARY KEY (`wechat_auth_id`),
  UNIQUE KEY `open_id` (`open_id`),
  KEY `fk_wechatauth_profile` (`user_id`),
  CONSTRAINT `fk_wechatauth_profile` FOREIGN KEY (`user_id`) REFERENCES `tb_person_info` (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```



## 2-3 配置Maven

## 2-4 逐层完成SSM的各项配置
> 快速构建SSM开发环境:https://liuurick.github.io/2020/12/01/%E5%BF%AB%E9%80%9F%E6%9E%84%E5%BB%BASSM%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/

## 2-5 升级mysql驱动相关的配置以支持mysql8

mysql5:`com.mysql.jdbc.Driver`

mysql8:`com.mysql.cj.jdbc.Driver`

## 2-6 验证Sevice与Controller
对于MVC架构而言，单元测试只需要测试service层即可

# 3 Logback配置与使用

> Logback配置与使用:https://liuurick.github.io/2018/10/31/Logback%E9%85%8D%E7%BD%AE%E4%B8%8E%E4%BD%BF%E7%94%A8/



# 4 店铺注册功能模块

## 4-1 Dao层之新增店铺

## 4-2 Dao层之更新店铺

## 4-3 Thumbnailator图片处理和封装Util

## 4-4 Dto之ShopExecution的实现

引入DTO，ShopExecution是一个数据传输对象（DTO)(Data Transfer Object)，是一种设计模式之间传输数据的软件应用系统。数据传输目标往往是数据访问对象从数据库中检索数据。

`数据传输对象`与`数据交互对象`或`数据访问对象`之间的差异是一个以不具有任何行为除了存储和检索的数据。

## 4-5 店铺注册之Service层的实现

## 4-6 店铺注册功能之Controller层的实现与改造

## 4-7 店铺注册之前端设计

> SUI mobile:http://sui.yater.net/
>
> MUI:https://dev.dcloud.net.cn/mui/

使用阿里巴巴的SUI mobile框架

## 4-8 店铺注册之js实现

## 4-9 店铺类别区域信息的获取

## 4-10 引入kaptcha实现验证码

**后端：**

1.引入依赖

```xml
<dependency>
	<groupId>com.github.penggle</groupId>
	<artifactId>kaptcha</artifactId>
	<version>${kaptcha.version}</version>
</dependency>
```

2.web.xml设定样式

```xml
<servlet>
    <servlet-name>Kaptcha</servlet-name>
    <servlet-class>com.google.code.kaptcha.servlet.KaptchaServlet</servlet-class>
    <!-- 是否有边框 -->
    <init-param>
      <param-name>kaptcha.border</param-name>
      <param-value>no</param-value>
    </init-param>
    <!-- 字体颜色 -->
    <init-param>
      <param-name>kaptcha.textproducer.font.color</param-name>
      <param-value>red</param-value>
    </init-param>
    <!-- 图片宽度 -->
    <init-param>
      <param-name>kaptcha.image.width</param-name>
      <param-value>135</param-value>
    </init-param>
    <!-- 使用哪些字符生成验证码 -->
    <init-param>
      <param-name>kaptcha.textproducer.char.string</param-name>
      <param-value>ACDEFHKPRSTWX345679</param-value>
    </init-param>
    <!-- 图片高度 -->
    <init-param>
      <param-name>kaptcha.image.height</param-name>
      <param-value>50</param-value>
    </init-param>
    <!-- 字体大小 -->
    <init-param>
      <param-name>kaptcha.textproducer.font.size</param-name>
      <param-value>43</param-value>
    </init-param>
    <!-- 干扰线的颜色 -->
    <init-param>
      <param-name>kaptcha.noise.color</param-name>
      <param-value>black</param-value>
    </init-param>
    <!-- 字符个数 -->
    <init-param>
      <param-name>kaptcha.textproducer.char.length</param-name>
      <param-value>4</param-value>
    </init-param>
    <!-- 字体 -->
    <init-param>
      <param-name>kaptcha.textproducer.font.names</param-name>
      <param-value>Arial</param-value>
    </init-param>
  </servlet>

  <servlet-mapping>
    <servlet-name>Kaptcha</servlet-name>
    <url-pattern>/Kaptcha</url-pattern>
  </servlet-mapping>
```

3.编写工具类检验验证码是否和预期相符

```java
public class CodeUtil {
	/**
	 * 检查验证码是否和预期相符
	 * 
	 * @param request
	 * @return
	 */
	public static boolean checkVerifyCode(HttpServletRequest request) {
		String verifyCodeExpected = (String) request.getSession()
				.getAttribute(com.google.code.kaptcha.Constants.KAPTCHA_SESSION_KEY);
		String verifyCodeActual = HttpServletRequestUtil.getString(request, "verifyCodeActual");
		if (verifyCodeActual == null || !verifyCodeActual.equals(verifyCodeExpected)) {
			return false;
		}
		return true;
	}
}
```

4.controller判断

```java
Map<String,Object> modelMap = new HashMap<String,Object>();
//验证码校验
if (!CodeUtil.checkVerifyCode(request)) {
	modelMap.put("success", false);
	modelMap.put("errMsg", "输入了错误的验证码");
	return modelMap;
}
```

**前端**：

```html
<!-- 验证码 Kaptcha -->
<li>
   <div class="item-content">
      <div class="item-inner">
         <div class="item-title label">验证码</div>
         <input type="text" id="j_captcha" placeholder="验证码">
         <div class="item-input">
            <img id="captcha_img" alt="点击更换" title="点击更换"
               onclick="changeVerifyCode(this)" src="../Kaptcha" />
         </div>
      </div>
   </div>
</li>
```

js

```js
function changeVerifyCode(img) {
	img.src = "../Kaptcha?" + Math.floor(Math.random() * 100);
}
```



# 5 主从库同步与读写分离

> 主从库同步与读写分离:https://liuurick.github.io/2018/10/31/%E4%B8%BB%E4%BB%8E%E5%BA%93%E5%90%8C%E6%AD%A5%E4%B8%8E%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB/

# 6 店铺编辑和列表功能

6-1 店铺信息编辑之Dao层开发
6-2 店铺信息编辑之Service层的实现
6-3 店铺信息编辑之Controller层实现
6-4 店铺信息编辑之前端实现
6-5 店铺列表展示之Dao层的实现
6-6 店铺列表展示之Service层的实现
6-7 店铺列表展示之Contoller层的实现
6-8 店铺列表展示前端开发
6-9 店铺管理页面的前端开发

# 7 商品类别模块

7-1 商铺类别展示
7-3 商品类别批量添加后端开发

```xml
<insert id="batchInsertProductCategory" parameterType="java.util.List">
        INSERT INTO
        tb_product_category(product_category_name, priority,
        create_time, shop_id)
        VALUES
        <foreach collection="list" item="productCategory" index="index"
                 separator=",">
            (
            #{productCategory.productCategoryName},
            #{productCategory.priority},
            #{productCategory.createTime},
            #{productCategory.shopId}
            )
        </foreach>
</insert>
```



7-4 商品类别批量添加的前端开发
7-5 商品类别删除后端开发
7-6 商品类别删除前端开发

# 8 商品模块

## 8-1 商品添加之Dao层的实现
## 8-2 商品添加之Service层的实现
## 8-3 商品添加之Controller层的实现
## 8-4 商品添加之前端
## 8-5 商品编辑之后端开发
## 8-6 商品编辑之前端实现
## 8-7 商品列表展示之后端开发
## 8-8 商品列表展示之前端开发

## 8-9 解除商品与某商品类别的关联的实现
将此类别下的商品里的类别id置为空，再删除掉该商品类别

```xml
<update id="updateProductCategoryToNull" parameterType="long">
   UPDATE
   tb_product
   SET
   product_category_id = null
   WHERE product_category_id =
   #{productCategoryId}
</update>
```



```java
@Override
    @Transactional
    public ProductCategoryExecution deleteProductCategory(long productCategoryId, long shopId)
            throws ProductCategoryOperationException {
        // 解除tb_product里的商品与该producategoryId的关联
        try {
            int effectedNum = productDao.updateProductCategoryToNull(productCategoryId);
            if (effectedNum < 0) {
                throw new ProductCategoryOperationException("商品类别更新失败");
            }
        } catch (Exception e) {
            throw new ProductCategoryOperationException("deleteProductCategory error: " + e.getMessage());
        }
        // 删除该productCategory
        try {
            int effectedNum = productCategoryDao.deleteProductCategory(productCategoryId, shopId);
            if (effectedNum <= 0) {
                throw new ProductCategoryOperationException("商品类别删除失败");
            } else {
                return new ProductCategoryExecution(ProductCategoryStateEnum.SUCCESS);
            }
        } catch (Exception e) {
            throw new ProductCategoryOperationException("deleteProductCategory error:" + e.getMessage());
        }
    }
```





# 9 前端展示系统

## 9-1 首页后台的开发
## 9-2 首页前端的开发
## 9-3 docBase与upload的含义

> server.xml中docBase和path的关系梳理:https://liuurick.github.io/2019/02/13/server-xml%E4%B8%ADdocBase%E5%92%8Cpath%E7%9A%84%E5%85%B3%E7%B3%BB%E6%A2%B3%E7%90%86/

## 9-4 店铺列表页后端的开发
## 9-5 店铺列表页前端的开发
## 9-6 店铺详情页的开发
## 9-7 商品详情页的开发
## 9-8 前端展示系统bug修复



# 10 阿里云部署及远程微信开发调试心得与技巧

## 10-1 阿里云初始化与执行环境安装
## 10-2 项目打包发布与域名解析
## 10-3 微信测试号的申请与连接
## 10-4 微信登录帐号的创建

# 11 项目优化

## 11-1 对关键配置信息进行DES加密

加密连接数据库明文密码，利用PropertyPlaceholderConfigurer实现对称加密。

DESUtil

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
		System.out.println(getEncryptString("123123!"));
	}

}
```

EncryptPropertyPlaceholderConfigurer

```java
public class EncryptPropertyPlaceholderConfigurer extends PropertyPlaceholderConfigurer {
	// 需要加密的字段数组
	private String[] encryptPropNames = { "jdbc.username", "jdbc.password" };

	/**
	 * 对关键的属性进行转换
	 */
	@Override
	protected String convertProperty(String propertyName, String propertyValue) {
		if (isEncryptProp(propertyName)) {
			// 对已加密的字段进行解密工作
			String decryptValue = DESUtil.getDecryptString(propertyValue);
			return decryptValue;
		} else {
			return propertyValue;
		}
	}

	/**
	 * 该属性是否已加密
	 * 
	 * @param propertyName
	 * @return
	 */
	private boolean isEncryptProp(String propertyName) {
		// 若等于需要加密的field，则进行加密
		for (String encryptpropertyName : encryptPropNames) {
			if (encryptpropertyName.equals(propertyName)) {
				return true;
			}
		}
		return false;
	}
}
```

修改spring-dao.xml配置文件

```xml
<!-- 1.配置数据库相关参数properties的属性：${url} -->
<bean class="com.liuurick.o2o.util.EncryptPropertyPlaceholderConfigurer">
		<property name="locations">
			<list>
				<value>classpath:jdbc.properties</value>
				<value>classpath:redis.properties</value>
			</list>
		</property>
		<property name="fileEncoding" value="UTF-8" />
</bean>
```

## 11-2 引入缓存技术Redis

将不经常变更的数据放在Redis，不仅可以提高访问速度，而且可以减轻数据库压力

![image-20210531202418622](C:\Users\admin\Desktop\blog\source\images\2019032302.png)

### Redis配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">

   <!-- Redis连接池的设置 -->
   <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
      <!-- 控制一个pool可分配多少个jedis实例 -->
      <property name="maxTotal" value="${redis.pool.maxActive}" />
      <!-- 连接池中最多可空闲maxIdle个连接 ，这里取值为20，表示即使没有数据库连接时依然可以保持20空闲的连接，而不被清除，随时处于待命状态。 -->
      <property name="maxIdle" value="${redis.pool.maxIdle}" />
      <!-- 最大等待时间:当没有可用连接时,连接池等待连接被归还的最大时间(以毫秒计数),超过时间则抛出异常 -->
      <property name="maxWaitMillis" value="${redis.pool.maxWait}" />
      <!-- 在获取连接的时候检查有效性 -->
      <property name="testOnBorrow" value="${redis.pool.testOnBorrow}" />
   </bean>

   <!-- 创建Redis连接池，并做相关配置 -->
   <bean id="jedisWritePool" class="com.liuurick.o2o.cache.JedisPoolWriper"
      depends-on="jedisPoolConfig">
      <constructor-arg index="0" ref="jedisPoolConfig" />
      <constructor-arg index="1" value="${redis.hostname}" />
      <constructor-arg index="2" value="${redis.port}" type="int" />
   </bean>

   <!-- 创建Redis工具类，封装好Redis的连接以进行相关的操作 -->
   <bean id="jedisUtil" class="com.liuurick.o2o.cache.JedisUtil" scope="singleton">
      <property name="jedisPool">
         <ref bean="jedisWritePool" />
      </property>
   </bean>

   <!-- Redis的key操作 -->
   <bean id="jedisKeys" class="com.liuurick.o2o.cache.JedisUtil$Keys"
      scope="singleton">
      <constructor-arg ref="jedisUtil"></constructor-arg>
   </bean>

   <!-- Redis的Strings操作 -->
   <bean id="jedisStrings" class="com.liuurick.o2o.cache.JedisUtil$Strings"
      scope="singleton">
      <constructor-arg ref="jedisUtil"></constructor-arg>
   </bean>

   <!-- Redis的Lists操作 -->
   <bean id="jedisLists" class="com.liuurick.o2o.cache.JedisUtil$Lists"
      scope="singleton">
      <constructor-arg ref="jedisUtil"></constructor-arg>
   </bean>

   <!-- Redis的Sets操作 -->
   <bean id="jedisSets" class="com.liuurick.o2o.cache.JedisUtil$Sets"
      scope="singleton">
      <constructor-arg ref="jedisUtil"></constructor-arg>
   </bean>

   <!-- Redis的HashMap操作 -->
   <bean id="jedisHash" class="com.liuurick.o2o.cache.JedisUtil$Hash"
      scope="singleton">
      <constructor-arg ref="jedisUtil"></constructor-arg>
   </bean>
</beans>    
```

redis.properties

```properties
redis.hostname=127.0.0.1
redis.port=6379
redis.database=0

redis.pool.maxActive=100
redis.pool.maxIdle=20
redis.pool.maxWait=3000
redis.pool.testOnBorrow=true
```



### 缓存操作封装

1.引入依赖

```xml
<!-- redis客户端:Jedis -->
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>${jedis.version}</version>
</dependency>
```

2.JedisPoolWriper与JedisUtil

```java
/**
 * 强指定redis的JedisPool接口构造函数，这样才能在centos成功创建jedispool
 *
 */
public class JedisPoolWriper {
   /** Redis连接池对象 */
   private JedisPool jedisPool;

   public JedisPoolWriper(final JedisPoolConfig poolConfig, final String host,
         final int port) {
      try {
         jedisPool = new JedisPool(poolConfig, host, port);
      } catch (Exception e) {
         e.printStackTrace();
      }
   }

   /**
    * 获取Redis连接池对象
    * @return
    */
   public JedisPool getJedisPool() {
      return jedisPool;
   }

   /**
    * 注入Redis连接池对象
    * @param jedisPool
    */
   public void setJedisPool(JedisPool jedisPool) {
      this.jedisPool = jedisPool;
   }

}
```



```java
package com.liuurick.o2o.cache;

import java.util.List;
import java.util.Map;
import java.util.Set;

import redis.clients.jedis.BinaryClient.LIST_POSITION;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.SortingParams;
import redis.clients.util.SafeEncoder;

public class JedisUtil {
   /**
    * 缓存生存时间
    */
   private final int expire = 60000;
   /** 操作Key的方法 */
   public Keys KEYS;
   /** 对存储结构为String类型的操作 */
   public Strings STRINGS;
   /** 对存储结构为List类型的操作 */
   public Lists LISTS;
   /** 对存储结构为Set类型的操作 */
   public Sets SETS;
   /** 对存储结构为HashMap类型的操作 */
   public Hash HASH;

   /** Redis连接池对象 */
   private JedisPool jedisPool;

   /**
    * 获取redis连接池
    * 
    * @return
    */
   public JedisPool getJedisPool() {
      return jedisPool;
   }

   /**
    * 设置redis连接池
    * 
    * @return
    */
   public void setJedisPool(JedisPoolWriper jedisPoolWriper) {
      this.jedisPool = jedisPoolWriper.getJedisPool();
   }

   /**
    * 从jedis连接池中获取获取jedis对象
    * 
    * @return
    */
   public Jedis getJedis() {
      return jedisPool.getResource();
   }

   /**
    * 设置过期时间
    * 
    * @author xiangze
    * @param key
    * @param seconds
    */
   public void expire(String key, int seconds) {
      if (seconds <= 0) {
         return;
      }
      Jedis jedis = getJedis();
      jedis.expire(key, seconds);
      jedis.close();
   }

   /**
    * 设置默认过期时间
    * 
    * @author xiangze
    * @param key
    */
   public void expire(String key) {
      expire(key, expire);
   }

   // *******************************************Keys*******************************************//
   public class Keys {

      /**
       * 清空所有key
       */
      public String flushAll() {
         Jedis jedis = getJedis();
         String stata = jedis.flushAll();
         jedis.close();
         return stata;
      }

      /**
       * 更改key
       * 
       * @param String
       *            oldkey
       * @param String
       *            newkey
       * @return 状态码
       */
      public String rename(String oldkey, String newkey) {
         return rename(SafeEncoder.encode(oldkey), SafeEncoder.encode(newkey));
      }

      /**
       * 更改key,仅当新key不存在时才执行
       * 
       * @param String
       *            oldkey
       * @param String
       *            newkey
       * @return 状态码
       */
      public long renamenx(String oldkey, String newkey) {
         Jedis jedis = getJedis();
         long status = jedis.renamenx(oldkey, newkey);
         jedis.close();
         return status;
      }

      /**
       * 更改key
       * 
       * @param String
       *            oldkey
       * @param String
       *            newkey
       * @return 状态码
       */
      public String rename(byte[] oldkey, byte[] newkey) {
         Jedis jedis = getJedis();
         String status = jedis.rename(oldkey, newkey);
         jedis.close();
         return status;
      }

      /**
       * 设置key的过期时间，以秒为单位
       * 
       * @param String
       *            key
       * @param 时间
       *            ,已秒为单位
       * @return 影响的记录数
       */
      public long expired(String key, int seconds) {
         Jedis jedis = getJedis();
         long count = jedis.expire(key, seconds);
         jedis.close();
         return count;
      }

      /**
       * 设置key的过期时间,它是距历元（即格林威治标准时间 1970 年 1 月 1 日的 00:00:00，格里高利历）的偏移量。
       * 
       * @param String
       *            key
       * @param 时间
       *            ,已秒为单位
       * @return 影响的记录数
       */
      public long expireAt(String key, long timestamp) {
         Jedis jedis = getJedis();
         long count = jedis.expireAt(key, timestamp);
         jedis.close();
         return count;
      }

      /**
       * 查询key的过期时间
       * 
       * @param String
       *            key
       * @return 以秒为单位的时间表示
       */
      public long ttl(String key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         long len = sjedis.ttl(key);
         sjedis.close();
         return len;
      }

      /**
       * 取消对key过期时间的设置
       * 
       * @param key
       * @return 影响的记录数
       */
      public long persist(String key) {
         Jedis jedis = getJedis();
         long count = jedis.persist(key);
         jedis.close();
         return count;
      }

      /**
       * 删除keys对应的记录,可以是多个key
       * 
       * @param String
       *            ... keys
       * @return 删除的记录数
       */
      public long del(String... keys) {
         Jedis jedis = getJedis();
         long count = jedis.del(keys);
         jedis.close();
         return count;
      }

      /**
       * 删除keys对应的记录,可以是多个key
       * 
       * @param String
       *            ... keys
       * @return 删除的记录数
       */
      public long del(byte[]... keys) {
         Jedis jedis = getJedis();
         long count = jedis.del(keys);
         jedis.close();
         return count;
      }

      /**
       * 判断key是否存在
       * 
       * @param String
       *            key
       * @return boolean
       */
      public boolean exists(String key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         boolean exis = sjedis.exists(key);
         sjedis.close();
         return exis;
      }

      /**
       * 对List,Set,SortSet进行排序,如果集合数据较大应避免使用这个方法
       * 
       * @param String
       *            key
       * @return List<String> 集合的全部记录
       **/
      public List<String> sort(String key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         List<String> list = sjedis.sort(key);
         sjedis.close();
         return list;
      }

      /**
       * 对List,Set,SortSet进行排序或limit
       * 
       * @param String
       *            key
       * @param SortingParams
       *            parame 定义排序类型或limit的起止位置.
       * @return List<String> 全部或部分记录
       **/
      public List<String> sort(String key, SortingParams parame) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         List<String> list = sjedis.sort(key, parame);
         sjedis.close();
         return list;
      }

      /**
       * 返回指定key存储的类型
       * 
       * @param String
       *            key
       * @return String string|list|set|zset|hash
       **/
      public String type(String key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         String type = sjedis.type(key);
         sjedis.close();
         return type;
      }

      /**
       * 查找所有匹配给定的模式的键
       * 
       * @param String
       *            key的表达式,*表示多个，？表示一个
       */
      public Set<String> keys(String pattern) {
         Jedis jedis = getJedis();
         Set<String> set = jedis.keys(pattern);
         jedis.close();
         return set;
      }
   }

   // *******************************************Strings*******************************************//
   public class Strings {
      /**
       * 根据key获取记录
       * 
       * @param String
       *            key
       * @return 值
       */
      public String get(String key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         String value = sjedis.get(key);
         sjedis.close();
         return value;
      }

      /**
       * 根据key获取记录
       * 
       * @param byte[]
       *            key
       * @return 值
       */
      public byte[] get(byte[] key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         byte[] value = sjedis.get(key);
         sjedis.close();
         return value;
      }

      /**
       * 添加记录,如果记录已存在将覆盖原有的value
       * 
       * @param String
       *            key
       * @param String
       *            value
       * @return 状态码
       */
      public String set(String key, String value) {
         return set(SafeEncoder.encode(key), SafeEncoder.encode(value));
      }

      /**
       * 添加记录,如果记录已存在将覆盖原有的value
       * 
       * @param String
       *            key
       * @param String
       *            value
       * @return 状态码
       */
      public String set(String key, byte[] value) {
         return set(SafeEncoder.encode(key), value);
      }

      /**
       * 添加记录,如果记录已存在将覆盖原有的value
       * 
       * @param byte[]
       *            key
       * @param byte[]
       *            value
       * @return 状态码
       */
      public String set(byte[] key, byte[] value) {
         Jedis jedis = getJedis();
         String status = jedis.set(key, value);
         jedis.close();
         return status;
      }

      /**
       * 添加有过期时间的记录
       * 
       * @param String
       *            key
       * @param int
       *            seconds 过期时间，以秒为单位
       * @param String
       *            value
       * @return String 操作状态
       */
      public String setEx(String key, int seconds, String value) {
         Jedis jedis = getJedis();
         String str = jedis.setex(key, seconds, value);
         jedis.close();
         return str;
      }

      /**
       * 添加有过期时间的记录
       * 
       * @param String
       *            key
       * @param int
       *            seconds 过期时间，以秒为单位
       * @param String
       *            value
       * @return String 操作状态
       */
      public String setEx(byte[] key, int seconds, byte[] value) {
         Jedis jedis = getJedis();
         String str = jedis.setex(key, seconds, value);
         jedis.close();
         return str;
      }

      /**
       * 添加一条记录，仅当给定的key不存在时才插入
       * 
       * @param String
       *            key
       * @param String
       *            value
       * @return long 状态码，1插入成功且key不存在，0未插入，key存在
       */
      public long setnx(String key, String value) {
         Jedis jedis = getJedis();
         long str = jedis.setnx(key, value);
         jedis.close();
         return str;
      }

      /**
       * 从指定位置开始插入数据，插入的数据会覆盖指定位置以后的数据<br/>
       * 例:String str1="123456789";<br/>
       * 对str1操作后setRange(key,4,0000)，str1="123400009";
       * 
       * @param String
       *            key
       * @param long
       *            offset
       * @param String
       *            value
       * @return long value的长度
       */
      public long setRange(String key, long offset, String value) {
         Jedis jedis = getJedis();
         long len = jedis.setrange(key, offset, value);
         jedis.close();
         return len;
      }

      /**
       * 在指定的key中追加value
       * 
       * @param String
       *            key
       * @param String
       *            value
       * @return long 追加后value的长度
       **/
      public long append(String key, String value) {
         Jedis jedis = getJedis();
         long len = jedis.append(key, value);
         jedis.close();
         return len;
      }

      /**
       * 将key对应的value减去指定的值，只有value可以转为数字时该方法才可用
       * 
       * @param String
       *            key
       * @param long
       *            number 要减去的值
       * @return long 减指定值后的值
       */
      public long decrBy(String key, long number) {
         Jedis jedis = getJedis();
         long len = jedis.decrBy(key, number);
         jedis.close();
         return len;
      }

      /**
       * <b>可以作为获取唯一id的方法</b><br/>
       * 将key对应的value加上指定的值，只有value可以转为数字时该方法才可用
       * 
       * @param String
       *            key
       * @param long
       *            number 要减去的值
       * @return long 相加后的值
       */
      public long incrBy(String key, long number) {
         Jedis jedis = getJedis();
         long len = jedis.incrBy(key, number);
         jedis.close();
         return len;
      }

      /**
       * 对指定key对应的value进行截取
       * 
       * @param String
       *            key
       * @param long
       *            startOffset 开始位置(包含)
       * @param long
       *            endOffset 结束位置(包含)
       * @return String 截取的值
       */
      public String getrange(String key, long startOffset, long endOffset) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         String value = sjedis.getrange(key, startOffset, endOffset);
         sjedis.close();
         return value;
      }

      /**
       * 获取并设置指定key对应的value<br/>
       * 如果key存在返回之前的value,否则返回null
       * 
       * @param String
       *            key
       * @param String
       *            value
       * @return String 原始value或null
       */
      public String getSet(String key, String value) {
         Jedis jedis = getJedis();
         String str = jedis.getSet(key, value);
         jedis.close();
         return str;
      }

      /**
       * 批量获取记录,如果指定的key不存在返回List的对应位置将是null
       * 
       * @param String
       *            keys
       * @return List<String> 值得集合
       */
      public List<String> mget(String... keys) {
         Jedis jedis = getJedis();
         List<String> str = jedis.mget(keys);
         jedis.close();
         return str;
      }

      /**
       * 批量存储记录
       * 
       * @param String
       *            keysvalues 例:keysvalues="key1","value1","key2","value2";
       * @return String 状态码
       */
      public String mset(String... keysvalues) {
         Jedis jedis = getJedis();
         String str = jedis.mset(keysvalues);
         jedis.close();
         return str;
      }

      /**
       * 获取key对应的值的长度
       * 
       * @param String
       *            key
       * @return value值得长度
       */
      public long strlen(String key) {
         Jedis jedis = getJedis();
         long len = jedis.strlen(key);
         jedis.close();
         return len;
      }
   }

   // *******************************************Sets*******************************************//
   public class Sets {

      /**
       * 向Set添加一条记录，如果member已存在返回0,否则返回1
       * 
       * @param String
       *            key
       * @param String
       *            member
       * @return 操作码,0或1
       */
      public long sadd(String key, String member) {
         Jedis jedis = getJedis();
         long s = jedis.sadd(key, member);
         jedis.close();
         return s;
      }

      public long sadd(byte[] key, byte[] member) {
         Jedis jedis = getJedis();
         long s = jedis.sadd(key, member);
         jedis.close();
         return s;
      }

      /**
       * 获取给定key中元素个数
       * 
       * @param String
       *            key
       * @return 元素个数
       */
      public long scard(String key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         long len = sjedis.scard(key);
         sjedis.close();
         return len;
      }

      /**
       * 返回从第一组和所有的给定集合之间的差异的成员
       * 
       * @param String
       *            ... keys
       * @return 差异的成员集合
       */
      public Set<String> sdiff(String... keys) {
         Jedis jedis = getJedis();
         Set<String> set = jedis.sdiff(keys);
         jedis.close();
         return set;
      }

      /**
       * 这个命令等于sdiff,但返回的不是结果集,而是将结果集存储在新的集合中，如果目标已存在，则覆盖。
       * 
       * @param String
       *            newkey 新结果集的key
       * @param String
       *            ... keys 比较的集合
       * @return 新集合中的记录数
       **/
      public long sdiffstore(String newkey, String... keys) {
         Jedis jedis = getJedis();
         long s = jedis.sdiffstore(newkey, keys);
         jedis.close();
         return s;
      }

      /**
       * 返回给定集合交集的成员,如果其中一个集合为不存在或为空，则返回空Set
       * 
       * @param String
       *            ... keys
       * @return 交集成员的集合
       **/
      public Set<String> sinter(String... keys) {
         Jedis jedis = getJedis();
         Set<String> set = jedis.sinter(keys);
         jedis.close();
         return set;
      }

      /**
       * 这个命令等于sinter,但返回的不是结果集,而是将结果集存储在新的集合中，如果目标已存在，则覆盖。
       * 
       * @param String
       *            newkey 新结果集的key
       * @param String
       *            ... keys 比较的集合
       * @return 新集合中的记录数
       **/
      public long sinterstore(String newkey, String... keys) {
         Jedis jedis = getJedis();
         long s = jedis.sinterstore(newkey, keys);
         jedis.close();
         return s;
      }

      /**
       * 确定一个给定的值是否存在
       * 
       * @param String
       *            key
       * @param String
       *            member 要判断的值
       * @return 存在返回1，不存在返回0
       **/
      public boolean sismember(String key, String member) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         boolean s = sjedis.sismember(key, member);
         sjedis.close();
         return s;
      }

      /**
       * 返回集合中的所有成员
       * 
       * @param String
       *            key
       * @return 成员集合
       */
      public Set<String> smembers(String key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         Set<String> set = sjedis.smembers(key);
         sjedis.close();
         return set;
      }

      public Set<byte[]> smembers(byte[] key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         Set<byte[]> set = sjedis.smembers(key);
         sjedis.close();
         return set;
      }

      /**
       * 将成员从源集合移出放入目标集合 <br/>
       * 如果源集合不存在或不包哈指定成员，不进行任何操作，返回0<br/>
       * 否则该成员从源集合上删除，并添加到目标集合，如果目标集合中成员已存在，则只在源集合进行删除
       * 
       * @param String
       *            srckey 源集合
       * @param String
       *            dstkey 目标集合
       * @param String
       *            member 源集合中的成员
       * @return 状态码，1成功，0失败
       */
      public long smove(String srckey, String dstkey, String member) {
         Jedis jedis = getJedis();
         long s = jedis.smove(srckey, dstkey, member);
         jedis.close();
         return s;
      }

      /**
       * 从集合中删除成员
       * 
       * @param String
       *            key
       * @return 被删除的成员
       */
      public String spop(String key) {
         Jedis jedis = getJedis();
         String s = jedis.spop(key);
         jedis.close();
         return s;
      }

      /**
       * 从集合中删除指定成员
       * 
       * @param String
       *            key
       * @param String
       *            member 要删除的成员
       * @return 状态码，成功返回1，成员不存在返回0
       */
      public long srem(String key, String member) {
         Jedis jedis = getJedis();
         long s = jedis.srem(key, member);
         jedis.close();
         return s;
      }

      /**
       * 合并多个集合并返回合并后的结果，合并后的结果集合并不保存<br/>
       * 
       * @param String
       *            ... keys
       * @return 合并后的结果集合
       * @see sunionstore
       */
      public Set<String> sunion(String... keys) {
         Jedis jedis = getJedis();
         Set<String> set = jedis.sunion(keys);
         jedis.close();
         return set;
      }

      /**
       * 合并多个集合并将合并后的结果集保存在指定的新集合中，如果新集合已经存在则覆盖
       * 
       * @param String
       *            newkey 新集合的key
       * @param String
       *            ... keys 要合并的集合
       **/
      public long sunionstore(String newkey, String... keys) {
         Jedis jedis = getJedis();
         long s = jedis.sunionstore(newkey, keys);
         jedis.close();
         return s;
      }
   }

   // *******************************************Hash*******************************************//
   public class Hash {

      /**
       * 从hash中删除指定的存储
       * 
       * @param String
       *            key
       * @param String
       *            fieid 存储的名字
       * @return 状态码，1成功，0失败
       */
      public long hdel(String key, String fieid) {
         Jedis jedis = getJedis();
         long s = jedis.hdel(key, fieid);
         jedis.close();
         return s;
      }

      public long hdel(String key) {
         Jedis jedis = getJedis();
         long s = jedis.del(key);
         jedis.close();
         return s;
      }

      /**
       * 测试hash中指定的存储是否存在
       * 
       * @param String
       *            key
       * @param String
       *            fieid 存储的名字
       * @return 1存在，0不存在
       */
      public boolean hexists(String key, String fieid) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         boolean s = sjedis.hexists(key, fieid);
         sjedis.close();
         return s;
      }

      /**
       * 返回hash中指定存储位置的值
       * 
       * @param String
       *            key
       * @param String
       *            fieid 存储的名字
       * @return 存储对应的值
       */
      public String hget(String key, String fieid) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         String s = sjedis.hget(key, fieid);
         sjedis.close();
         return s;
      }

      public byte[] hget(byte[] key, byte[] fieid) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         byte[] s = sjedis.hget(key, fieid);
         sjedis.close();
         return s;
      }

      /**
       * 以Map的形式返回hash中的存储和值
       * 
       * @param String
       *            key
       * @return Map<Strinig,String>
       */
      public Map<String, String> hgetAll(String key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         Map<String, String> map = sjedis.hgetAll(key);
         sjedis.close();
         return map;
      }

      /**
       * 添加一个对应关系
       * 
       * @param String
       *            key
       * @param String
       *            fieid
       * @param String
       *            value
       * @return 状态码 1成功，0失败，fieid已存在将更新，也返回0
       **/
      public long hset(String key, String fieid, String value) {
         Jedis jedis = getJedis();
         long s = jedis.hset(key, fieid, value);
         jedis.close();
         return s;
      }

      public long hset(String key, String fieid, byte[] value) {
         Jedis jedis = getJedis();
         long s = jedis.hset(key.getBytes(), fieid.getBytes(), value);
         jedis.close();
         return s;
      }

      /**
       * 添加对应关系，只有在fieid不存在时才执行
       * 
       * @param String
       *            key
       * @param String
       *            fieid
       * @param String
       *            value
       * @return 状态码 1成功，0失败fieid已存
       **/
      public long hsetnx(String key, String fieid, String value) {
         Jedis jedis = getJedis();
         long s = jedis.hsetnx(key, fieid, value);
         jedis.close();
         return s;
      }

      /**
       * 获取hash中value的集合
       * 
       * @param String
       *            key
       * @return List<String>
       */
      public List<String> hvals(String key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         List<String> list = sjedis.hvals(key);
         sjedis.close();
         return list;
      }

      /**
       * 在指定的存储位置加上指定的数字，存储位置的值必须可转为数字类型
       * 
       * @param String
       *            key
       * @param String
       *            fieid 存储位置
       * @param String
       *            long value 要增加的值,可以是负数
       * @return 增加指定数字后，存储位置的值
       */
      public long hincrby(String key, String fieid, long value) {
         Jedis jedis = getJedis();
         long s = jedis.hincrBy(key, fieid, value);
         jedis.close();
         return s;
      }

      /**
       * 返回指定hash中的所有存储名字,类似Map中的keySet方法
       * 
       * @param String
       *            key
       * @return Set<String> 存储名称的集合
       */
      public Set<String> hkeys(String key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         Set<String> set = sjedis.hkeys(key);
         sjedis.close();
         return set;
      }

      /**
       * 获取hash中存储的个数，类似Map中size方法
       * 
       * @param String
       *            key
       * @return long 存储的个数
       */
      public long hlen(String key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         long len = sjedis.hlen(key);
         sjedis.close();
         return len;
      }

      /**
       * 根据多个key，获取对应的value，返回List,如果指定的key不存在,List对应位置为null
       * 
       * @param String
       *            key
       * @param String
       *            ... fieids 存储位置
       * @return List<String>
       */
      public List<String> hmget(String key, String... fieids) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         List<String> list = sjedis.hmget(key, fieids);
         sjedis.close();
         return list;
      }

      public List<byte[]> hmget(byte[] key, byte[]... fieids) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         List<byte[]> list = sjedis.hmget(key, fieids);
         sjedis.close();
         return list;
      }

      /**
       * 添加对应关系，如果对应关系已存在，则覆盖
       * 
       * @param Strin
       *            key
       * @param Map
       *            <String,String> 对应关系
       * @return 状态，成功返回OK
       */
      public String hmset(String key, Map<String, String> map) {
         Jedis jedis = getJedis();
         String s = jedis.hmset(key, map);
         jedis.close();
         return s;
      }

      /**
       * 添加对应关系，如果对应关系已存在，则覆盖
       * 
       * @param Strin
       *            key
       * @param Map
       *            <String,String> 对应关系
       * @return 状态，成功返回OK
       */
      public String hmset(byte[] key, Map<byte[], byte[]> map) {
         Jedis jedis = getJedis();
         String s = jedis.hmset(key, map);
         jedis.close();
         return s;
      }

   }

   // *******************************************Lists*******************************************//
   public class Lists {
      /**
       * List长度
       * 
       * @param String
       *            key
       * @return 长度
       */
      public long llen(String key) {
         return llen(SafeEncoder.encode(key));
      }

      /**
       * List长度
       * 
       * @param byte[]
       *            key
       * @return 长度
       */
      public long llen(byte[] key) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         long count = sjedis.llen(key);
         sjedis.close();
         return count;
      }

      /**
       * 覆盖操作,将覆盖List中指定位置的值
       * 
       * @param byte[]
       *            key
       * @param int
       *            index 位置
       * @param byte[]
       *            value 值
       * @return 状态码
       */
      public String lset(byte[] key, int index, byte[] value) {
         Jedis jedis = getJedis();
         String status = jedis.lset(key, index, value);
         jedis.close();
         return status;
      }

      /**
       * 覆盖操作,将覆盖List中指定位置的值
       * 
       * @param key
       * @param int
       *            index 位置
       * @param String
       *            value 值
       * @return 状态码
       */
      public String lset(String key, int index, String value) {
         return lset(SafeEncoder.encode(key), index, SafeEncoder.encode(value));
      }

      /**
       * 在value的相对位置插入记录
       * 
       * @param key
       * @param LIST_POSITION
       *            前面插入或后面插入
       * @param String
       *            pivot 相对位置的内容
       * @param String
       *            value 插入的内容
       * @return 记录总数
       */
      public long linsert(String key, LIST_POSITION where, String pivot, String value) {
         return linsert(SafeEncoder.encode(key), where, SafeEncoder.encode(pivot), SafeEncoder.encode(value));
      }

      /**
       * 在指定位置插入记录
       * 
       * @param String
       *            key
       * @param LIST_POSITION
       *            前面插入或后面插入
       * @param byte[]
       *            pivot 相对位置的内容
       * @param byte[]
       *            value 插入的内容
       * @return 记录总数
       */
      public long linsert(byte[] key, LIST_POSITION where, byte[] pivot, byte[] value) {
         Jedis jedis = getJedis();
         long count = jedis.linsert(key, where, pivot, value);
         jedis.close();
         return count;
      }

      /**
       * 获取List中指定位置的值
       * 
       * @param String
       *            key
       * @param int
       *            index 位置
       * @return 值
       **/
      public String lindex(String key, int index) {
         return SafeEncoder.encode(lindex(SafeEncoder.encode(key), index));
      }

      /**
       * 获取List中指定位置的值
       * 
       * @param byte[]
       *            key
       * @param int
       *            index 位置
       * @return 值
       **/
      public byte[] lindex(byte[] key, int index) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         byte[] value = sjedis.lindex(key, index);
         sjedis.close();
         return value;
      }

      /**
       * 将List中的第一条记录移出List
       * 
       * @param String
       *            key
       * @return 移出的记录
       */
      public String lpop(String key) {
         return SafeEncoder.encode(lpop(SafeEncoder.encode(key)));
      }

      /**
       * 将List中的第一条记录移出List
       * 
       * @param byte[]
       *            key
       * @return 移出的记录
       */
      public byte[] lpop(byte[] key) {
         Jedis jedis = getJedis();
         byte[] value = jedis.lpop(key);
         jedis.close();
         return value;
      }

      /**
       * 将List中最后第一条记录移出List
       * 
       * @param byte[]
       *            key
       * @return 移出的记录
       */
      public String rpop(String key) {
         Jedis jedis = getJedis();
         String value = jedis.rpop(key);
         jedis.close();
         return value;
      }

      /**
       * 向List尾部追加记录
       * 
       * @param String
       *            key
       * @param String
       *            value
       * @return 记录总数
       */
      public long lpush(String key, String value) {
         return lpush(SafeEncoder.encode(key), SafeEncoder.encode(value));
      }

      /**
       * 向List头部追加记录
       * 
       * @param String
       *            key
       * @param String
       *            value
       * @return 记录总数
       */
      public long rpush(String key, String value) {
         Jedis jedis = getJedis();
         long count = jedis.rpush(key, value);
         jedis.close();
         return count;
      }

      /**
       * 向List头部追加记录
       * 
       * @param String
       *            key
       * @param String
       *            value
       * @return 记录总数
       */
      public long rpush(byte[] key, byte[] value) {
         Jedis jedis = getJedis();
         long count = jedis.rpush(key, value);
         jedis.close();
         return count;
      }

      /**
       * 向List中追加记录
       * 
       * @param byte[]
       *            key
       * @param byte[]
       *            value
       * @return 记录总数
       */
      public long lpush(byte[] key, byte[] value) {
         Jedis jedis = getJedis();
         long count = jedis.lpush(key, value);
         jedis.close();
         return count;
      }

      /**
       * 获取指定范围的记录，可以做为分页使用
       * 
       * @param String
       *            key
       * @param long
       *            start
       * @param long
       *            end
       * @return List
       */
      public List<String> lrange(String key, long start, long end) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         List<String> list = sjedis.lrange(key, start, end);
         sjedis.close();
         return list;
      }

      /**
       * 获取指定范围的记录，可以做为分页使用
       * 
       * @param byte[]
       *            key
       * @param int
       *            start
       * @param int
       *            end 如果为负数，则尾部开始计算
       * @return List
       */
      public List<byte[]> lrange(byte[] key, int start, int end) {
         // ShardedJedis sjedis = getShardedJedis();
         Jedis sjedis = getJedis();
         List<byte[]> list = sjedis.lrange(key, start, end);
         sjedis.close();
         return list;
      }

      /**
       * 删除List中c条记录，被删除的记录值为value
       * 
       * @param byte[]
       *            key
       * @param int
       *            c 要删除的数量，如果为负数则从List的尾部检查并删除符合的记录
       * @param byte[]
       *            value 要匹配的值
       * @return 删除后的List中的记录数
       */
      public long lrem(byte[] key, int c, byte[] value) {
         Jedis jedis = getJedis();
         long count = jedis.lrem(key, c, value);
         jedis.close();
         return count;
      }

      /**
       * 删除List中c条记录，被删除的记录值为value
       * 
       * @param String
       *            key
       * @param int
       *            c 要删除的数量，如果为负数则从List的尾部检查并删除符合的记录
       * @param String
       *            value 要匹配的值
       * @return 删除后的List中的记录数
       */
      public long lrem(String key, int c, String value) {
         return lrem(SafeEncoder.encode(key), c, SafeEncoder.encode(value));
      }

      /**
       * 算是删除吧，只保留start与end之间的记录
       * 
       * @param byte[]
       *            key
       * @param int
       *            start 记录的开始位置(0表示第一条记录)
       * @param int
       *            end 记录的结束位置（如果为-1则表示最后一个，-2，-3以此类推）
       * @return 执行状态码
       */
      public String ltrim(byte[] key, int start, int end) {
         Jedis jedis = getJedis();
         String str = jedis.ltrim(key, start, end);
         jedis.close();
         return str;
      }

      /**
       * 算是删除吧，只保留start与end之间的记录
       * 
       * @param String
       *            key
       * @param int
       *            start 记录的开始位置(0表示第一条记录)
       * @param int
       *            end 记录的结束位置（如果为-1则表示最后一个，-2，-3以此类推）
       * @return 执行状态码
       */
      public String ltrim(String key, int start, int end) {
         return ltrim(SafeEncoder.encode(key), start, end);
      }
   }

}
```



### 缓存编码实现

小试牛刀,将**头条信息**缓存

```
public interface HeadLineService {

    public static final String HLLISTKEY = "headlinelist";

    /**
     * 根据传入的条件返回指定的头条列表
     *
     * @param headLineCondition
     * @return
     * @throws IOException
     */
    List<HeadLine> getHeadLineList(HeadLine headLineCondition);

    /**
     * 添加头条信息，并存储头条图片
     *
     * @param headLine
     * @param thumbnail
     * @return
     */
    HeadLineExecution addHeadLine(HeadLine headLine, ImageHolder thumbnail);

    /**
     * 修改头条信息
     *
     * @param headLine
     * @param thumbnail
     * @return
     */
    HeadLineExecution modifyHeadLine(HeadLine headLine, ImageHolder thumbnail);

    /**
     * 删除单条头条
     *
     * @param headLineId
     * @return
     */
    HeadLineExecution removeHeadLine(long headLineId);

    /**
     * 批量删除头条
     *
     * @param headLineIdList
     * @return
     */
    HeadLineExecution removeHeadLineList(List<Long> headLineIdList);
}
```

```java
@Service
public class HeadLineServiceImpl implements HeadLineService {
    @Autowired
    private HeadLineDao headLineDao;
    @Autowired
    private JedisUtil.Keys jedisKeys;
    @Autowired
    private JedisUtil.Strings jedisStrings;

    private static Logger logger = LoggerFactory.getLogger(HeadLineServiceImpl.class);

    @Override
    @Transactional
    public List<HeadLine> getHeadLineList(HeadLine headLineCondition) {
        // 定义redis的key前缀
        String key = HLLISTKEY;
        // 定义接收对象
        List<HeadLine> headLineList = null;
        // 定义jackson数据转换操作类
        ObjectMapper mapper = new ObjectMapper();
        // 拼接出redis的key
        if (headLineCondition != null && headLineCondition.getEnableStatus() != null) {
            key = key + "_" + headLineCondition.getEnableStatus();
        }
        // 判断key是否存在
        if (!jedisKeys.exists(key)) {
            // 若不存在，则从数据库里面取出相应数据
            headLineList = headLineDao.queryHeadLine(headLineCondition);
            // 将相关的实体类集合转换成string,存入redis里面对应的key中
            String jsonString;
            try {
                jsonString = mapper.writeValueAsString(headLineList);
            } catch (JsonProcessingException e) {
                e.printStackTrace();
                logger.error(e.getMessage());
                throw new HeadLineOperationException(e.getMessage());
            }
            jedisStrings.set(key, jsonString);
        } else {
            // 若存在，则直接从redis里面取出相应数据
            String jsonString = jedisStrings.get(key);
            // 指定要将string转换成的集合类型
            JavaType javaType = mapper.getTypeFactory().constructParametricType(ArrayList.class, HeadLine.class);
            try {
                // 将相关key对应的value里的的string转换成对象的实体类集合
                headLineList = mapper.readValue(jsonString, javaType);
            } catch (JsonParseException e) {
                e.printStackTrace();
                logger.error(e.getMessage());
                throw new HeadLineOperationException(e.getMessage());
            } catch (JsonMappingException e) {
                e.printStackTrace();
                logger.error(e.getMessage());
                throw new HeadLineOperationException(e.getMessage());
            } catch (IOException e) {
                e.printStackTrace();
                logger.error(e.getMessage());
                throw new HeadLineOperationException(e.getMessage());
            }
        }
        return headLineList;
    }

    @Override
    @Transactional
    public HeadLineExecution addHeadLine(HeadLine headLine, ImageHolder thumbnail) {
        // 空值判断
        if (headLine != null) {
            // 设定默认值
            headLine.setCreateTime(new Date());
            headLine.setLastEditTime(new Date());
            // 若传入的头条图片为非空，则存储图片并在实体类里将图片的相对路径设置上
            if (thumbnail != null) {
                addThumbnail(headLine, thumbnail);
            }
            try {
                // 往数据库里插入头条信息
                int effectedNum = headLineDao.insertHeadLine(headLine);
                if (effectedNum > 0) {
                    deleteRedis4HeadLine();
                    return new HeadLineExecution(HeadLineStateEnum.SUCCESS, headLine);
                } else {
                    return new HeadLineExecution(HeadLineStateEnum.INNER_ERROR);
                }
            } catch (Exception e) {
                throw new HeadLineOperationException("添加区域信息失败:" + e.toString());
            }
        } else {
            return new HeadLineExecution(HeadLineStateEnum.EMPTY);
        }
    }

    @Override
    @Transactional
    public HeadLineExecution modifyHeadLine(HeadLine headLine, ImageHolder thumbnail) {
        // 空值判断，主要是判断头条Id是否为空
        if (headLine.getLineId() != null && headLine.getLineId() > 0) {
            // 设定默认值
            headLine.setLastEditTime(new Date());
            if (thumbnail != null) {
                // 若需要修改的图片流不为空，则需要处理现有的图片
                HeadLine tempHeadLine = headLineDao.queryHeadLineById(headLine.getLineId());
                if (tempHeadLine.getLineImg() != null) {
                    // 若原来是存储有图片的，则将原先图片删除
                    ImageUtil.deleteFileOrPath(tempHeadLine.getLineImg());
                }
                // 添加新的图片，并将新的图片相对路径设置到实体类里
                addThumbnail(headLine, thumbnail);
            }
            try {
                // 更新头条信息
                int effectedNum = headLineDao.updateHeadLine(headLine);
                if (effectedNum > 0) {
                    deleteRedis4HeadLine();
                    return new HeadLineExecution(HeadLineStateEnum.SUCCESS, headLine);
                } else {
                    return new HeadLineExecution(HeadLineStateEnum.INNER_ERROR);
                }
            } catch (Exception e) {
                throw new HeadLineOperationException("更新头条信息失败:" + e.toString());
            }
        } else {
            return new HeadLineExecution(HeadLineStateEnum.EMPTY);
        }
    }

    @Override
    @Transactional
    public HeadLineExecution removeHeadLine(long headLineId) {
        // 空值判断，主要判断头条Id是否为非空
        if (headLineId > 0) {
            try {
                // 根据Id查询头条信息
                HeadLine tempHeadLine = headLineDao.queryHeadLineById(headLineId);
                if (tempHeadLine.getLineImg() != null) {
                    // 若头条原先存有图片，则将该图片文件删除
                    ImageUtil.deleteFileOrPath(tempHeadLine.getLineImg());
                }
                // 删除数据库里对应的头条信息
                int effectedNum = headLineDao.deleteHeadLine(headLineId);
                if (effectedNum > 0) {
                    deleteRedis4HeadLine();
                    return new HeadLineExecution(HeadLineStateEnum.SUCCESS);
                } else {
                    return new HeadLineExecution(HeadLineStateEnum.INNER_ERROR);
                }
            } catch (Exception e) {
                throw new HeadLineOperationException("删除头条信息失败:" + e.toString());
            }
        } else {
            return new HeadLineExecution(HeadLineStateEnum.EMPTY);
        }
    }

    @Override
    @Transactional
    public HeadLineExecution removeHeadLineList(List<Long> headLineIdList) {
        // 空值判断
        if (headLineIdList != null && headLineIdList.size() > 0) {
            try {
                // 根据传入的id列表获取头条列表
                List<HeadLine> headLineList = headLineDao.queryHeadLineByIds(headLineIdList);
                for (HeadLine headLine : headLineList) {
                    // 遍历头条列表，若头条的图片非空，则将图片删除
                    if (headLine.getLineImg() != null) {
                        ImageUtil.deleteFileOrPath(headLine.getLineImg());
                    }
                }
                // 批量删除数据库中的头条信息
                int effectedNum = headLineDao.batchDeleteHeadLine(headLineIdList);
                if (effectedNum > 0) {
                    deleteRedis4HeadLine();
                    return new HeadLineExecution(HeadLineStateEnum.SUCCESS);
                } else {
                    return new HeadLineExecution(HeadLineStateEnum.INNER_ERROR);
                }
            } catch (Exception e) {
                throw new HeadLineOperationException("删除头条信息失败:" + e.toString());
            }
        } else {
            return new HeadLineExecution(HeadLineStateEnum.EMPTY);
        }
    }

    /**
     * 存储图片
     *
     * @param headLine
     * @param thumbnail
     */
    private void addThumbnail(HeadLine headLine, ImageHolder thumbnail) {
        String dest = PathUtil.getHeadLineImagePath();
        String thumbnailAddr = ImageUtil.generateNormalImg(thumbnail, dest);
        headLine.setLineImg(thumbnailAddr);
    }

    /**
     * 移除跟实体类相关的redis key-value
     */
    private void deleteRedis4HeadLine() {
        String prefix = HLLISTKEY;
        // 获取跟头条相关的redis key
        Set<String> keySet = jedisKeys.keys(prefix + "*");
        for (String key : keySet) {
            // 逐条删除
            jedisKeys.del(key);
        }
    }
}
```

缓存清除

依据key前缀删除匹配该模式下的所有key-value 如传入:shopcategory,则shopcategory_allfirstlevel等
以shopcategory打头的key_value都会被清空


```
@Service
public class CacheServiceImpl implements CacheService {
   @Autowired
   private JedisUtil.Keys jedisKeys;

   @Override
   public void removeFromCache(String keyPrefix) {
      Set<String> keySet = jedisKeys.keys(keyPrefix + "*");
      for (String key : keySet) {
         jedisKeys.del(key);
      }
   }
}
```



## 11-3 添加平台帐号体系

dao层：

```java
public interface PersonInfoDao {
   /**
    * 根据查询条件分页返回用户信息列表
    * 
    * @param personInfoCondition
    * @param rowIndex
    * @param pageSize
    * @return
    */
   List<PersonInfo> queryPersonInfoList(@Param("personInfoCondition")PersonInfo personInfoCondition, @Param("rowIndex")int rowIndex, @Param("pageSize")int pageSize);

   /**
    * 根据查询条件返回总数，配合queryPersonInfoList使用
    * 
    * @param personInfoCondition
    * @return
    */
   int queryPersonInfoCount(@Param("personInfoCondition")PersonInfo personInfoCondition);

   /**
    * 通过用户Id查询用户
    * 
    * @param userId
    * @return
    */
   PersonInfo queryPersonInfoById(long userId);

   /**
    * 添加用户信息
    * 
    * @param personInfo
    * @return
    */
   int insertPersonInfo(PersonInfo personInfo);

   /**
    * 修改用户信息
    * 
    * @param personInfo
    * @return
    */
   int updatePersonInfo(PersonInfo personInfo);

}
```

```java
public interface WechatAuthDao {
   /**
    * 通过openId查询对应本平台的微信帐号
    * 
    * @param openId
    * @return
    */
   WechatAuth queryWechatInfoByOpenId(String openId);

   /**
    * 添加对应本平台的微信帐号
    * 
    * @param wechatAuth
    * @return
    */
   int insertWechatAuth(WechatAuth wechatAuth);

}
```



service层：

```java
public interface PersonInfoService {

   /**
    * 根据用户Id获取personInfo信息
    * 
    * @param userId
    * @return
    */
   PersonInfo getPersonInfoById(Long userId);

   /**
    * 根据查询条件分页返回用户信息列表
    * 
    * @param personInfoCondition
    * @param pageIndex
    * @param pageSize
    * @return
    */
   PersonInfoExecution getPersonInfoList(PersonInfo personInfoCondition, int pageIndex, int pageSize);

   /**
    * 根据传入的PersonInfo修改对应的用户信息
    * 
    * @param pi
    * @return
    */
   PersonInfoExecution modifyPersonInfo(PersonInfo pi);
}
```

```java
public interface WechatAuthService {

   /**
    * 通过openId查找平台对应的微信帐号
    * 
    * @param openId
    * @return
    */
   WechatAuth getWechatAuthByOpenId(String openId);

   /**
    * 注册本平台的微信帐号
    * 
    * @param wechatAuth
    * @param profileImg
    * @return
    * @throws RuntimeException
    */
   WechatAuthExecution register(WechatAuth wechatAuth) throws WechatAuthOperationException;

}
```

controller层：

```java
@Controller
@RequestMapping("wechat")
public class WechatController {

	private static Logger log = LoggerFactory.getLogger(WechatController.class);

	@RequestMapping(method = { RequestMethod.GET })
	public void doGet(HttpServletRequest request, HttpServletResponse response) {
		log.debug("weixin get...");
		// 微信加密签名，signature结合了开发者填写的token参数和请求中的timestamp参数、nonce参数。
		String signature = request.getParameter("signature");
		// 时间戳
		String timestamp = request.getParameter("timestamp");
		// 随机数
		String nonce = request.getParameter("nonce");
		// 随机字符串
		String echostr = request.getParameter("echostr");

		// 通过检验signature对请求进行校验，若校验成功则原样返回echostr，表示接入成功，否则接入失败
		PrintWriter out = null;
		try {
			out = response.getWriter();
			if (SignUtil.checkSignature(signature, timestamp, nonce)) {
				log.debug("weixin get success....");
				out.print(echostr);
			}
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			if (out != null){
				out.close();
			}

		}
	}
}
```



```java
@Controller
@RequestMapping("wechatlogin")
public class WechatLoginController {

   private static Logger log = LoggerFactory.getLogger(WechatLoginController.class);
   private static final String FRONTEND = "1";
   private static final String SHOPEND = "2";

   @Autowired
   private PersonInfoService personInfoService;

   @Autowired
   private WechatAuthService wechatAuthService;

   @RequestMapping(value = "/logincheck", method = { RequestMethod.GET })
   public String doGet(HttpServletRequest request, HttpServletResponse response) {
      log.debug("weixin login get...");
      // 获取微信公众号传输过来的code,通过code可获取access_token,进而获取用户信息
      String code = request.getParameter("code");
      // 这个state可以用来传我们自定义的信息，方便程序调用，这里也可以不用
      String roleType = request.getParameter("state");
      log.debug("weixin login code:" + code);
      WechatUser user = null;
      String openId = null;
      WechatAuth auth = null;
      if (null != code) {
         UserAccessToken token;
         try {
            // 通过code获取access_token
            token = WechatUtil.getUserAccessToken(code);
            log.debug("weixin login token:" + token.toString());
            // 通过token获取accessToken
            String accessToken = token.getAccessToken();
            // 通过token获取openId
            openId = token.getOpenId();
            // 通过access_token和openId获取用户昵称等信息
            user = WechatUtil.getUserInfo(accessToken, openId);
            log.debug("weixin login user:" + user.toString());
            request.getSession().setAttribute("openId", openId);
            auth = wechatAuthService.getWechatAuthByOpenId(openId);
         } catch (IOException e) {
            log.error("error in getUserAccessToken or getUserInfo or findByOpenId: " + e.toString());
            e.printStackTrace();
         }
      }
      // 若微信帐号为空则需要注册微信帐号，同时注册用户信息
      if (auth == null) {
         PersonInfo personInfo = WechatUtil.getPersonInfoFromRequest(user);
         auth = new WechatAuth();
         auth.setOpenId(openId);
         if (FRONTEND.equals(roleType)) {
            personInfo.setUserType(1);
         } else {
            personInfo.setUserType(2);
         }
         auth.setPersonInfo(personInfo);
         WechatAuthExecution we = wechatAuthService.register(auth);
         if (we.getState() != WechatAuthStateEnum.SUCCESS.getState()) {
            return null;
         } else {
            personInfo = personInfoService.getPersonInfoById(auth.getPersonInfo().getUserId());
            request.getSession().setAttribute("user", personInfo);
         }
      } else {
         request.getSession().setAttribute("user", auth.getPersonInfo());
      }
      // 若用户点击的是前端展示系统按钮则进入前端展示系统
      if (FRONTEND.equals(roleType)) {
         return "frontend/index";
      } else {
         return "shop/shoplist";
      }
   }
}
```

## 11-4 添加拦截器

## 11-5 定期备份数据的实现

> https://www.jb51.net/article/107574.htm

```sql
mysqldump -u账号 -p密码 数据库 > /root/backup/sql/o2o 'date +%y%m%d%H%M%S'.sql 
```

另外使用crontab实现定时备份




# 12 项目2.0设计

 12-1 项目2.0功能总览
 12-2 实体类及对应表的设计
 12-3 给商品增加积分字段及相应的程序改动

# 13 框架大换血
项目框架由ssm迭代成SpringBoot
## 13-1 SpringBoot的理论知识
## 13-2 SpringBoot的搭建与启动
## 13-3 pom的迁移
## 13-4 dao的迁移
## 13-6 service的迁移
## 13-7 web层的迁移
## 13-8 前端页面的迁移
## 13-9 验证码的迁移
## 13-10 替代docBase配置以实现图片的加载
## 13-11 拦截器的迁移
## 13-12 PathUtil的改进
## 13-13 项目的打包与部署



#  14 店家管理系统增强

  ## 14-1 AwardDao的开发与测试
  ## 14-2 UserAwardMapDao的开发与测试
  ## 14-3 UserProductDao的开发与测试
  ## 14-4 ProductSellDailyDao的开发与测试
  ## 14-5 UserShopMapDao的开发与测试
  ## 14-6 ShopAuthMapDao的开发与测试
  ## 14-7 店铺授权之service层编码及最终效果展示
  ## 14-8 店铺授权之二维码工具类的编写
  ## 14-9 店铺授权之列表展示和授权修改的实现
  ## 14-10 店铺授权之访问微信获取用户信息的URL的剥离
  ## 14-11 店铺授权之短链接的实现
  ## 14-12 店铺授权之授权二维码的生成
  ## 14-13 店铺授权之添加授权的编码实现
  ## 14-14 店铺授权之部署以及远程调试
  ## 14-15 2.0框架升级
  ## 14-16 添加统一异常处理模块(适合项目1.0和2.0)
  ## 14-17 统一异常处理模块的验证



# 15 前端展示系统增强和超级管理员模块

 ##  15-1 将Quartz引入到框架里
 ##  15-2 定时统计店铺的商品日销量
 ##  15-3 店铺销量基础service和controller的编写
 ##  15-4 店铺销量统计前端开发
 ##  15-5 Echarts的动态化改
 ##  15-6 Echarts的动态化验证

 ##  15-7 店家管理系统剩余功能开发之消费记录展示更改
 ##  15-8 前端展示系统补强之店铺详情页的修改

 ## 15-9 店家管理系统剩余功能开发之顾客积分页的开发

 ## 15-10 店家管理系统剩余功能开发之奖品领取页的开发

 ## 15-11 店家管理系统剩余功能开发之奖品管理页的开发

 ## 15-12 店家管理系统剩余功能开发之奖品操作页的开发

 ## 15-13 前端展示系统补强之店铺奖品列表页的开发

 ## 15-14 前端展示系统补强之奖品兑换记录列表页的开发

 ## 15-15 前端展示系统补强之消费记录二维码的生成以及消费记录的添加

 ## 15-16 前端展示系统补强之奖品兑换记录详情页的开发

 ## 15-17 前端展示系统补强之消费记录列表页的开发

 ## 15-18 前端展示系统补强之顾客各店铺积分列表页的开发
 ## 15-19 前端展示系统补强之奖品详情页的开发
 ## 15-20 超级管理员系统提点



