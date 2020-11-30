---
title: SpringBoot集成MyBatis的分页插件PageHelper
date: 2020-10-25 11:11:35
tags: [Java,SpringBoot]
categories: 分页功能
---

## 

<!--more-->

首先说说MyBatis框架的PageHelper插件吧，它是一个非常好用的分页插件，通常我们的项目中如果集成了MyBatis的话，几乎都会用到它，因为分页的业务逻辑说复杂也不复杂，但是有插件我们何乐而不为？通常引入它们只需三步骤，不管是Spring集成还是SpringBoot集成都是老套路，这里就分开总结了。。。



## Spring集成PageHelper：

**第一步：pom文件引入依赖**

```xml
 <!-- mybatis的分页插件 -->
 <dependency>
     <groupId>com.github.pagehelper</groupId>
     <artifactId>pagehelper</artifactId>
 </dependency>
```

**第二步：MyBatis的核心配置文件中引入配置项**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!-- 【mybatis的核心配置文件】 -->

    <!-- 批量设置别名(可以不配) 作用：就是在mapper.xml文件中直接写类名，也可以不用写全路径名。 -->
    <typeAliases>
        <package name="com.liuurick.manager.po" />
    </typeAliases>

    <!-- 配置mybatis的分页插件PageHelper -->
    <plugins>
        <!-- com.github.pagehelper为PageHelper类所在包名 -->
        <plugin interceptor="com.github.pagehelper.PageHelper">
            <!-- 设置数据库类型Oracle,Mysql,MariaDB,SQLite,Hsqldb,PostgreSQL六种数据库 -->
            <property name="dialect" value="mysql"/>
        </plugin>
    </plugins>

</configuration>
```

**第三步：业务逻辑实现分页功能，我们只需将当前查询的页数page和每页显示的总条数rows传进去，然后Pagehelper已经帮我们分好数据了，只需在web层获取数据即可。**

```java
//分页查询商品列表：
@Override
public DatagridResult itemList(Integer page, Integer rows) {
    //为了程序的严谨性，判断非空：
    if(page == null){
        page = 1;   //设置默认当前页
    }
    if(page <= 0){
        page = 1;
    }
    if(rows == null){
        rows = 30;    //设置默认每页显示的商品数(因为jsp页面上默认写的就是30条)
    }
    
    //1、设置分页信息，包括当前页数和每页显示的总计数
    PageHelper.startPage(page, rows);
    //2、执行查询
    TbItemExample example = new TbItemExample();
    List<TbItem> list = tbItemMapper.selectByExample(example);
    //3、获取分页查询后的数据
    PageInfo<TbItem> pageInfo = new PageInfo<>(list);
    //4、封装需要返回的分页实体
    DatagridResult result = new DatagridResult();
    //设置获取到的总记录数total：
    result.setTotal(pageInfo.getTotal());
    //设置数据集合rows：
    result.setRows(list);
    
    return result;
}
```

## springboot集成PageHelper：

**第一步：pom文件还是需要引入依赖**

```xml
<dependency>
   <groupId>com.github.pagehelper</groupId>
   <artifactId>pagehelper</artifactId>
   <version>4.1.6</version>
</dependency>
```

**第二步：这次直接是在项目的入口类application.java中直接设置PageHelper插件即可**

```java
//配置mybatis的分页插件pageHelper
@Bean
public PageHelper pageHelper(){
    PageHelper pageHelper = new PageHelper();
    Properties properties = new Properties();
    properties.setProperty("offsetAsPageNum","true");
    properties.setProperty("rowBoundsWithCount","true");
    properties.setProperty("reasonable","true");
    properties.setProperty("dialect","mysql");    //配置mysql数据库的方言
    pageHelper.setProperties(properties);
    return pageHelper;
}
```

**第三步：同理，使用插件实现分页功能，方式还是一样，只需将当前查询的页数和每页显示的条数穿进去即可，直接源码**

这是需要用到的分页实体，各位可以直接享用。

```java
package com.riemann.utils;

/**
 * 分页bean
 */

import java.util.List;

public class PageBean<T> {
    // 当前页
    private Integer currentPage = 1;
    // 每页显示的总条数
    private Integer pageSize = 10;
    // 总条数
    private Integer totalNum;
    // 是否有下一页
    private Integer isMore;
    // 总页数
    private Integer totalPage;
    // 开始索引
    private Integer startIndex;
    // 分页结果
    private List<T> items;

    public PageBean() {
        super();
    }

    public PageBean(Integer currentPage, Integer pageSize, Integer totalNum) {
        super();
        this.currentPage = currentPage;
        this.pageSize = pageSize;
        this.totalNum = totalNum;
        this.totalPage = (this.totalNum+this.pageSize-1)/this.pageSize;
        this.startIndex = (this.currentPage-1)*this.pageSize;
        this.isMore = this.currentPage >= this.totalPage?0:1;
    }

    public Integer getCurrentPage() {
        return currentPage;
    }

    public void setCurrentPage(Integer currentPage) {
        this.currentPage = currentPage;
    }

    public Integer getPageSize() {
        return pageSize;
    }

    public void setPageSize(Integer pageSize) {
        this.pageSize = pageSize;
    }

    public Integer getTotalNum() {
        return totalNum;
    }

    public void setTotalNum(Integer totalNum) {
        this.totalNum = totalNum;
    }

    public Integer getIsMore() {
        return isMore;
    }

    public void setIsMore(Integer isMore) {
        this.isMore = isMore;
    }

    public Integer getTotalPage() {
        return totalPage;
    }

    public void setTotalPage(Integer totalPage) {
        this.totalPage = totalPage;
    }

    public Integer getStartIndex() {
        return startIndex;
    }

    public void setStartIndex(Integer startIndex) {
        this.startIndex = startIndex;
    }

    public List<T> getItems() {
        return items;
    }

    public void setItems(List<T> items) {
        this.items = items;
    }
}
```

分页功能源码(web层和service层)。

```java
@Override
public List<Item> findItemByPage(int currentPage,int pageSize) {
    //设置分页信息，分别是当前页数和每页显示的总记录数【记住：必须在mapper接口中的方法执行之前设置该分页信息】
    PageHelper.startPage(currentPage, pageSize);
    
    List<Item> allItems = itemMapper.findAll();        //全部商品
    int countNums = itemMapper.countItem();            //总记录数
    PageBean<Item> pageData = new PageBean<>(currentPage, pageSize, countNums);
    pageData.setItems(allItems);
    return pageData.getItems();
}
/**
 * 商品分页功能(集成mybatis的分页插件pageHelper实现)
 * 
 * @param currentPage    :当前页数
 * @param pageSize        :每页显示的总记录数
 * @return
 */
@RequestMapping("/itemsPage")
@ResponseBody
public List<Item> itemsPage(int currentPage,int pageSize){
    return itemService.findItemByPage(currentPage, pageSize);
}
```