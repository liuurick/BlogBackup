---
title: Java实现分页功能常见的几种方法
date: 2020-10-25 10:59:11
tags: 分页功能
categories: Java
---

总结一下分页功能的实现方法

<!--more-->

## 一、limit关键字

**service层**

```java
@Service
@Transactional
public class ImplStudentService implements StudentService {
 
@Resource
private  StudentDao  studentDao;
 
    @Override
    public List<Student>  selectAllStudent(String province, Integer offset, Integer limit) {
        return studentDao.selectAll(province,offset,limit);
    }
}
```

**sql语句**

```
select * from student where province = #{province} limit #{offset},#{limit}
```

## 二、hibernate分页

**service层**

```java
  @Override
  public List getStudents(Integer  pageNo，Integer  pageSize) throws Exception {
  // 分页数据
  int[] startIdAndCount = new int[2];
  startIdAndCount[0] = pageNo * pageSize;
  startIdAndCount[1] = pageSize;
  return studentDao.selectStudentsByPage(startIdAndCount);
 }

```

**dao层**

```java
@Override
public List findByHqlPage(int[] startIdAndCount) throws Exception {
	String hql = "...";
	try {
		Query query = getSession().createQuery(hql);
		// 设置分页
		if (startIdAndCount != null && startIdAndCount.length > 0) {
			int rowStartIdx = Math.max(0, startIdAndCount[0]);
			if (rowStartIdx > 0) {
				query.setFirstResult(rowStartIdx);// 设置开始取值的索引
			}
			if (startIdAndCount.length > 1) {
				int rowCount = Math.max(0, startIdAndCount[1]);
				if (rowCount > 0) {
				query.setMaxResults(rowCount);// 设置结束取值的索引
				}
			}
		}
		return query.list();
	} catch (RuntimeException re) {
		log.error("分页查询失败！", re);
		throw re;
	}
}
```

## 三、截取List查询结果分页(简单粗暴)

```java
...
List<StudentEnroll> students = studentlDao.getAllStudents();
int count = 0;
if(studentEnrolls != null && studentEnrolls.size() > 0) {
	count = studentEnrolls.size();
	int fromIndex = pageNo * pageSize;
	int toIndex = (pageNo + 1) * pageSize;
	if(toIndex > count) {
		toIndex = count;
	}
	List<StudentEnroll> pageList = studentEnrolls.subList(fromIndex, toIndex);
...
```

## 四、mybatis框架pageHelper插件分页

Spring整合：

导入pom.xml

```xml
<!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper -->
 <dependency>
      <groupId>com.github.pagehelper</groupId>
      <artifactId>pagehelper</artifactId>
      <version>5.1.2</version>
 </dependency>
```

配置项目配置文件(我在spring和mybatis整合的配置文件中配置的，如果在mybatis核心配置文件中配置，百度一下)

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 依赖数据源 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 注册加载myBatis映射文件 -->
        <property name="mapperLocations">
            <array>
                <value>classpath*:com/yyz/mapper/*Mapper.xml</value>
            </array>
        </property>
        <!-- PageHelper分页配置 -->
        <property name="plugins">
            <array>
                <bean class="com.github.pagehelper.PageInterceptor">
                    <property name="properties">
                        <!--使用下面的方式配置参数，一行配置一个，后面会有所有的参数介绍 -->
                        <value>
                    <!--helperDialect属性来指定分页插件使用哪种方言。-->
                            helperDialect=mysql
                    <!--分页合理化参数，设置为true时，pageNum<=0时会查询第一页,pageNum>pages(超过总数时),会查询最后一页。-->
                            reasonable=true
                    <!--为了支持startPage(Object params)方法，增加了该参数来配置参数映射，用于从对象中根据属性名取值，
                        可以配置 pageNum,pageSize,count,pageSizeZero,reasonable-->
                            params=count=countSql
                    <!--支持通过Mapper接口参数来传递分页参数，默认值false，分页插件会从查询方法的参数值中，自动根据上面 params 配
                     置的字段中取值，查找到合适的值时就会自动分页。-->
                            supportMethodsArguments=true
                    <!--默认值为 false。设置为 true 时，允许在运行时根据多数据源自动识别对应方言的分页-->
                            autoRuntimeDialect=true
                        </value>
                    </property>
                </bean>
            </array>
        </property>
        <!-- 给数据库实体起别名 -->
        <property name="typeAliasesPackage" value="com.yyz.entity;"/>
 </bean>
```

SpringBoot整合：

```xml
<!--分页插件-->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>最新版本</version>
</dependency>
```

配置项目application.yml文件

```yml
#bybatis分页插件配置
pagehelper:
  helper-dialect: mysql  #数据库
  reasonable: true
  support-methods-arguments: true
  params: count=countSql
```

> ## 标题分页插件参数：
>
> 分页插件提供了多个可选参数，这些参数使用时，按照上面配置方式中的示例配置即可。
>
> 分页插件可选参数如下：
>
> **dialect**：默认情况下会使用 PageHelper 方式进行分页，如果想要实现自己的分页逻辑，可以实现
> **Dialect**(com.github.pagehelper.Dialect) 接口，然后配置该属性为实现类的全限定名称。 使用自定义
> dialect 实现时，下面的参数没有任何作用。
>
> **helperDialect**：分页插件会自动检测当前的数据库链接，自动选择合适的分页方式。
> oracle,mysql,mariadb,sqlite,hsqldb,postgresql,db2,sqlserver,informix,h2,sqlserver2012,derby
> **特别注意**：使用 SqlServer2012 数据库时，需要手动指定为 sqlserver2012，否则会使用 SqlServer2005
> 的方式进行分页。
>
> **offsetAsPageNum**：默认值为 false，该参数对使用 RowBounds 作为分页参数时有效。 当该参数设置为 true
> 时，会将 RowBounds 中的 offset 参数当成 pageNum 使用，可以用页码和页面大小两个参数进行分页。
>
> **rowBoundsWithCount**：默认值为false，该参数对使用 RowBounds 作为分页参数时有效。
> 当该参数设置为true时，使用 RowBounds 分页会进行 count 查询。
>
> **pageSizeZero**：默认值为 false，当该参数设置为 true 时，如果 pageSize=0 或者
> RowBounds.limit = 0 就会查询出全部的结果（相当于没有执行分页查询，但是返回结果仍然是 Page 类型）。
>
> **reasonable**：分页合理化参数，默认值为false。当该参数设置为 true 时，pageNum<=0
> 时会查询第一页，pageNum>pages（超过总数时），会查询最后一页。默认false 时，直接根据参数进行查询。
>
> **params**：为了支持startPage(Object params)方法，增加了该参数来配置参数映射，用于从对象中根据属性名取值，
> 可以配置 pageNum,pageSize,count,pageSizeZero,reasonable，不配置映射的用默认值，
> 默认值为pageNum=pageNum;pageSize=pageSize;count=countSql;reasonable=reasonable;pageSizeZero=pageSizeZero。
>
> **supportMethodsArguments**：支持通过 Mapper
> 接口参数来传递分页参数，默认值false，分页插件会从查询方法的参数值中，自动根据上面 params
> 配置的字段中取值，查找到合适的值时就会自动分页。
>
> **aggregateFunctions**：默认为所有常见数据库的聚合函数，允许手动添加聚合函数（影响行数），所有以聚合函数开头的函数，在进行
> count 转换时，会套一层。其他函数和列会被替换为 count(0)，其中count列可以自己配置。

## 重要提示：

当 offsetAsPageNum=false 的时候，由于 PageNum 问题，RowBounds查询的时候 reasonable 会强制为 false。使用 PageHelper.startPage 方法不受影响。

service层

```java
@Override
public ResponseResult selectAllStudent(Integer pageNum, Integer pageSize) {
    Map<String,Object> map = new HashMap<>();
    PageHelper.startPage(pageNum,pageSize);
    List<Student>  students = studentMapper.selectAllStudents();
    PageInfo pageInfo = new PageInfo(students);
    long total = pageInfo.getTotal();
    map.put("result",pageInfo);
    map.put("count",total);
    return ResponseResultUtil.success(map);
}
```

详细请看 [SpringBoot集成MyBatis的分页插件PageHelper](https://blog.csdn.net/riemann_/article/details/90454451)

## 五、springData分页

service层

```java
...
Sort.Order travelDate = new Sort.Order(Sort.Direction.DESC, "travelDate");
Sort.Order createdTime = new Sort.Order(Sort.Direction.DESC, "createdTime");
Sort sort = new Sort(travelDate, createdTime);
Pageable pageable = new PageRequest(page, pageSize, sort);
List<TravelItem> items = null;
try {
    items = travelRepository.getTravelItemsByTravelDateBetweenAndUserId(theStartDate, theEndDate, openId, pageable);
} catch (Exception e) {
    throw new DatabaseRelatedException("TravelRepository异常");
}
 ...
```

dao层：接口继承的是PagingAndSortingRepository接口，注意要加@Repository注解