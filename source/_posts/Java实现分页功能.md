---
title: Java实现分页功能
date: 2019-2-24 10:29:15
tags: 分页功能
categories: Java
---

总结一下分页功能的实现方法

<!--more-->

# sql尝试

先建立一个表

```sql
CREATE TABLE `test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8
```





```
select * FROM test
ORDER BY id
LIMIT 3,3
```









一、limit关键字

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

## 