---
layout: post
title: Mybatis - 使用注解配置SQL映射器 
category : [JavaEE]
tagline: "Supporting tagline"
tags : [Mybatis]
---
{% include JB/setup %}
# Mybatis - 使用注解配置SQL映射器 
---

> 引用：   
> [Java Persistence with MyBatis 3(中文版) 第四章 使用注解配置SQL映射器](http://blog.csdn.net/luanlouis/article/details/35780175) 

<!--break-->

## 在映射器 Mapper 接口上使用 @Mapper 注解 


## 映射语句 
MyBatis提供了多种注解来支持不同类型的语句(statement)如SELECT,INSERT,UPDATE,DELETE。

### @Insert 
我们可以使用 `@Insert` 注解来定义一个INSERT映射语句： 
``` 
package com.mybatis3.mappers;  
public interface StudentMapper {  
    @Insert("INSERT INTO STUDENTS(STUD_ID,NAME,EMAIL,ADDR_ID, PHONE)  
            VALUES(#{studId},#{name},#{email},#{address.addrId},#{phone})")  
    int insertStudent(Student student);  
}
``` 

使用了 `@Insert` 注解的 insertMethod() 方法将返回 insert 语句执行后影响的行数。 

#### 自动生成主键 
我们可以使用 `@Options` 注解的 `userGeneratedKeys` 和 `keyProperty` 属性让数据库产生 auto_increment（自增长）列的值，然后将生成的值设置到输入参数对象的属性中。 
``` 
@Insert("INSERT INTO STUDENTS(NAME,EMAIL,ADDR_ID, PHONE)  
        VALUES(#{name},#{email},#{address.addrId},#{phone})")  
@Options(useGeneratedKeys = true, keyProperty = "studId")  
int insertStudent(Student student);  
``` 

这里STUD_ID列值将会通过MySQL数据库自动生成。并且生成的值将会被设置到student对象的studId属性中。 

``` 
StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);  
mapper.insertStudent(student);  
long studentId = student.getStudId(); 
``` 

有一些数据库如 Oracle，并不支持 AUTO_INCREMENT 列属性，它使用序列（SEQUENCE）来产生主键的值。
我们可以使用 `@SelectKey` 注解来为任意 SQL 语句来指定主键值，作为主键列的值。

假设我们有一个名为 `STUD_ID_SEQ` 的序列来生成 `STUD_ID` 主键值。 

``` 
@Insert("INSERT INTO STUDENTS(STUD_ID,NAME,EMAIL,ADDR_ID, PHONE)   
VALUES(#{studId},#{name},#{email},#{address.addrId},#{phone})")  
@SelectKey(statement="SELECT STUD_ID_SEQ.NEXTVAL FROM DUAL",   
keyProperty="studId", resultType=long.class, before=true)  
int insertStudent(Student student); 
``` 

这里我们使用了 `@SelectKey` 来生成主键值，并且存储到了 student 对象的 studId 属性上。由于我们设置了 `before=true`，该语句将会在执行 INSERT 语句之前执行。

如果你使用序列作为触发器来设置主键值，我们可以在 INSERT 语句执行后，从 `sequence_name.currval` 获取数据库产生的主键值。 
``` 
@Insert("INSERT INTO STUDENTS(NAME,EMAIL,ADDR_ID, PHONE)   
VALUES(#{name},#{email},#{address.addrId},#{phone})")  
@SelectKey(statement="SELECT STUD_ID_SEQ.CURRVAL FROM DUAL",   
keyProperty="studId", resultType=long.class, before=false)  
int insertStudent(Student student);  
``` 

### @Update 
我们可以使用 `@Update` 注解来定义一个UPDATE映射语句，如下所示： 
``` 
@Update("UPDATE STUDENTS SET NAME=#{name}, EMAIL=#{email},   
PHONE=#{phone} WHERE STUD_ID=#{studId}")  
int updateStudent(Student student); 
``` 

使用了 `@Update` 的 `updateStudent()` 方法将会返回执行了 update 语句后影响的行数。 

``` 
StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);  
int noOfRowsUpdated = mapper.updateStudent(student); 
``` 

### @Delete 
我们可以使用 `@Delete` 注解来定义一个 DELETE 映射语句，如下所示： 
``` 
@Delete("DELETE FROM STUDENTS WHERE STUD_ID=#{studId}")  
int deleteStudent(long studId); 
``` 

使用了 `@Delete` 的 `deleteStudent()` 方法将会返回执行了 delete 语句后影响的行数。

### @Select 
我们可以使用 `@Select` 注解来定义一个 SELECT 映射语句，如下所示： 
``` 
package com.mybatis3.mappers;  
public interface StudentMapper  
{  
    @Select("SELECT STUD_ID AS STUDID, NAME, EMAIL, PHONE FROM  
            STUDENTS WHERE STUD_ID=#{studId}")  
    Student findStudentById(long studId);  
}
``` 

为了将列名和 Student bean 属性名匹配，我们为 stud_id 起了一个 studId 的别名。如果返回了多行结果，将抛出 `TooManyResultsException` 异常。 

## 结果映射 












