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
MyBatis 提供了多种注解来支持不同类型的语句(statement)如 SELECT，INSERT，UPDATE，DELETE。

### @Insert 
我们可以使用 `@Insert` 注解来定义一个 INSERT 映射语句： 
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

这里 STUD_ID 列值将会通过MySQL数据库自动生成。并且生成的值将会被设置到 student 对象的 studId 属性中。 

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
我们可以将查询结果通过别名或者是 `@Results` 注解与 JavaBean 属性映射起来。 
现在让我们看看怎样使用 `@Results` 注解将指定列与指定 JavaBean 属性映射器来，执行 SELECT 查询的： 
``` 
package com.mybatis3.mappers;  
public interface StudentMapper  
{  
    @Select("SELECT * FROM STUDENTS")  
    @Results(  
    {  
        @Result(id = true, column = "stud_id", property = "studId"),  
        @Result(column = "name", property = "name"),  
        @Result(column = "email", property = "email"),  
        @Result(column = "addr_id", property = "address.addrId")  
    })  
    List<Student> findAllStudents();  
} 
``` 

> `@Results` 注解和映射器 XML 配置文件元素 `<resultMap>` 相对应，然而，`Mybatis3.2.2` 不能为 `@Results` 注解赋予一个 ID。所以，不像 `<resultMap>` 元素，我们不应该在不同的映射语句中重用 `@Results` 声明。
> 这意味着即使 `@Results` 注解完全相同，我们也需要（在不同的映射接口中）重复 `@Results` 声明。 

例如，看下面的 findStudentById() 和 findAllStudents() 方法： 
``` 
@Select("SELECT * FROM STUDENTS WHERE STUD_ID=#{studId}")  
@Results(  
{  
    @Result(id = true, column = "stud_id", property = "studId"),  
    @Result(column = "name", property = "name"),  
    @Result(column = "email", property = "email"),  
    @Result(column = "addr_id", property = "address.addrId")  
})  
Student findStudentById(int studId);  
    
@Select("SELECT * FROM STUDENTS")  
@Results(  
{  
    @Result(id = true, column = "stud_id", property = "studId"),  
    @Result(column = "name", property = "name"),  
    @Result(column = "email", property = "email"),  
    @Result(column = "addr_id", property = "address.addrId")  
})  
List<Student> findAllStudents(); 
```  

这里两个语句的` @Results` 配置完全相同，但是我必须得重复它。这里有一个解决方法。我们可以创建一个映射器 Mapper 配置文件， 然后配置 `<resultMap>` 元素，然后使用 `@ResultMap` 注解引用此 `<resultMap>`。

在 StudentMapper.xml 中定义一个 ID 为 StudentResult 的 `<resultMap>` ： 
``` 
<mapper namespace="com.mybatis3.mappers.StudentMapper">  
  <resultMap type="Student" id="StudentResult">  
    <id property="studId" column="stud_id" />  
    <result property="name" column="name" />  
    <result property="email" column="email" />  
    <result property="phone" column="phone" />  
  </resultMap>  
</mapper> 
``` 

在 StudentMapper.java 中，使用 `@ResultMap` 引用名为 StudentResult 的 resultMap： 
``` 
public interface StudentMapper {  
    @Select("SELECT * FROM STUDENTS WHERE STUD_ID=#{studId}")  
    @ResultMap("com.mybatis3.mappers.StudentMapper.StudentResult")  
    Student findStudentById(long studId);  
      
    @Select("SELECT * FROM STUDENTS")  
    @ResultMap("com.mybatis3.mappers.StudentMapper.StudentResult")  
    List<Student> findAllStudents();  
}  
``` 

### 一对一映射 
MyBatis 提供了 `@One` 注解来使用嵌套 select 语句（Nested-Select）加载一对一关联查询数据。让我们看看怎样使 `用@One` 注解获取学生及其地址信息。
``` 
public interface StudentMapper {  
    @Select("SELECT ADDR_ID AS ADDRID, STREET, CITY, STATE, ZIP, COUNTRY  
            FROM ADDRESSES WHERE ADDR_ID=#{id}")  
    Address findAddressById(long id);  
       
    @Select("SELECT * FROM STUDENTS WHERE STUD_ID=#{studId} ")  
    @Results(  
    {  
        @Result(id = true, column = "stud_id", property = "studId"),  
        @Result(column = "name", property = "name"),  
        @Result(column = "email", property = "email"),  
        @Result(property = "address", column = "addr_id",  
        one = @One(select = "com.mybatis3.mappers.StudentMapper.findAddressById"))  
    })  
    Student selectStudentWithAddress(long studId);  
} 
```  

这里我们使用了 `@One` 注解的 select 属性来指定一个使用了完全限定名的方法上，该方法会返回一个 Address 对象。使用 `column=”addr_id”` ，则 STUDENTS 表中列 addr_id 的值将会作为输入参数传递给 findAddressById() 方法。如果 `@One` SELECT 查询返回了多行结果，则会抛出 `TooManyResultsException` 异常。

``` 
long studId = 1;  
StudentMapper studentMapper =   
    sqlSession.getMapper(StudentMapper.class);  
Student student = studentMapper.selectStudentWithAddress(studId);  
System.out.println("Student :"+student);  
System.out.println("Address :"+student.getAddress()); 
```  

在使用 XML 配置 SQL 映射器中，我们可以通过基于 XML 的映射器配置，使用嵌套结果 ResultMap 来加载一对一关联的查询。而 `MyBatis3.2.2` 版本，并没有对应的注解支持。但是我们可以在映射器 Mapper 配置文件中配置 `<resultMap>` 并且使用 `@ResultMap` 注解来引用它。
在 StudentMapper.xml 中配置 `<resultMap>` ，如下所示： 
``` 
<mapper namespace="com.mybatis3.mappers.StudentMapper">  
  <resultMap type="Address" id="AddressResult">  
    <id property="addrId" column="addr_id" />  
    <result property="street" column="street" />  
    <result property="city" column="city" />  
    <result property="state" column="state" />  
    <result property="zip" column="zip" />  
    <result property="country" column="country" />  
  </resultMap>  
  <resultMap type="Student" id="StudentWithAddressResult">  
    <id property="studId" column="stud_id" />  
    <result property="name" column="name" />  
    <result property="email" column="email" />  
    <association property="address" resultMap="AddressResult" />  
  </resultMap>  
</mapper> 
``` 

``` 
public interface StudentMapper  
{  
    @Select("select stud_id, name, email, a.addr_id, street, city,  
            state, zip, country" + " FROM students s left outer join addresses a  
            on s.addr_id=a.addr_id" + " where stud_id=#{studId} ")  
    @ResultMap("com.mybatis3.mappers.StudentMapper.  
               StudentWithAddressResult")  
    Student selectStudentWithAddress(long id);  
} 
```  

### 一对多映射 
MyBatis 提供了 `@Many` 注解，用来使用嵌套 Select 语句加载一对多关联查询。 
现在让我们看一下如何使用 `@Many` 注解获取一个讲师及其教授课程列表信息： 
``` 
public interface TutorMapper  
{  
    @Select("select addr_id as addrId, street, city, state, zip,  
            country from addresses where addr_id=#{id}")  
    Address findAddressById(int id);  
       
    @Select("select * from courses where tutor_id=#{tutorId}")  
    @Results(  
    {  
        @Result(id = true, column = "course_id", property = "courseId"),  
        @Result(column = "name", property = "name"),  
        @Result(column = "description", property = "description"),  
        @Result(column = "start_date" property = "startDate"),  
        @Result(column = "end_date" property = "endDate")  
    })  
    List<Course> findCoursesByTutorId(int tutorId);  
        
    @Select("SELECT tutor_id, name as tutor_name, email, addr_id  
            FROM tutors where tutor_id=#{tutorId}")  
    @Results(  
    {  
        @Result(id = true, column = "tutor_id", property = "tutorId"),  
        @Result(column = "tutor_name", property = "name"),  
        @Result(column = "email", property = "email"),  
        @Result(property = "address", column = "addr_id",  
        one = @One(select = " com.mybatis3.  
        mappers.TutorMapper.findAddressById")),  
        @Result(property = "courses", column = "tutor_id",  
        many = @Many(select = "com.mybatis3.mappers.TutorMapper.  
        findCoursesByTutorId"))  
    })  
    Tutor findTutorById(int tutorId);  
} 
```

这里我们使用了 `@Many` 注解的 select 属性来指向一个完全限定名称的方法，该方法将返回一个 List<Course> 对象。使用 `column=”tutor_id”` ，TUTORS 表中的 tutor_id 列值将会作为输入参数传递给 findCoursesByTutorId() 方法。

使用 XML 配置 SQL 映射器中，我们可以通过基于 XML 的映射器配置，使用嵌套结果 ResultMap 来加载一对多关联的查询。而 `MyBatis3.2.2` 版本，并没有对应的注解支持。但是我们可以在映射器 Mapper 配置文件中配置 `<resultMap>` 并且使用 `@ResultMap` 注解来引用它。
在 TutorMapper.xml 中配置 `<resultMap>` ，如下所示： 
``` 
<mapper namespace="com.mybatis3.mappers.TutorMapper">  
  <resultMap type="Address" id="AddressResult">  
    <id property="addrId" column="addr_id" />  
    <result property="street" column="street" />  
    <result property="city" column="city" />  
    <result property="state" column="state" />  
    <result property="zip" column="zip" />  
    <result property="country" column="country" />  
  </resultMap>  
  <resultMap type="Course" id="CourseResult">  
    <id column="course_id" property="courseId" />  
    <result column="name" property="name" />  
    <result column="description" property="description" />  
    <result column="start_date" property="startDate" />  
    <result column="end_date" property="endDate" />  
  </resultMap>  
  <resultMap type="Tutor" id="TutorResult">  
    <id column="tutor_id" property="tutorId" />  
    <result column="tutor_name" property="name" />  
    <result column="email" property="email" />  
    <association property="address" resultMap="AddressResult" />  
    <collection property="courses" resultMap="CourseResult" />  
  </resultMap>  
</mapper> 
``` 

``` 
public interface TutorMapper  
{  
    @Select("SELECT T.TUTOR_ID, T.NAME AS TUTOR_NAME, EMAIL,  
            A.ADDR_ID, STREET, CITY, STATE, ZIP, COUNTRY, COURSE_ID, C.NAME,  
            DESCRIPTION, START_DATE, END_DATE  FROM TUTORS T LEFT OUTER  
            JOIN ADDRESSES A ON T.ADDR_ID=A.ADDR_ID LEFT OUTER JOIN COURSES  
            C ON T.TUTOR_ID=C.TUTOR_ID WHERE T.TUTOR_ID=#{tutorId}")  
    @ResultMap("com.mybatis3.mappers.TutorMapper.TutorResult")  
    Tutor selectTutorById(int tutorId);  
} 
``` 

## 动态 SQL
有时候我们需要根据输入条件动态地构建 SQL 语句。MyBatis 提供了各种注解如 `@InsertProvider` ， `@UpdateProvider` ， `@DeleteProvider` 和 `@SelectProvider` ，来帮助构建动态 SQL 语句，然后让 MyBatis 执行这些 SQL 语句。 

### @SelectProvider 
现在让我们来看一个使用 `@SelectProvider` 注解来创建一个简单的 SELECT 映射语句的例子。 
创建一个 TutorDynaSqlProvider.java 类，以及 findTutorByIdSql() 方法，如下所示： 
``` 
package com.mybatis3.sqlproviders;  
import org.apache.ibatis.jdbc.SQL;  
public class TutorDynaSqlProvider {  
    public String findTutorByIdSql(long tutorId) {  
        return "SELECT TUTOR_ID AS tutorId, NAME, EMAIL FROM TUTORS  
               WHERE TUTOR_ID=" + tutorId;  
    }  
}  
```  

在 TutorMapper.java 接口中创建一个映射语句，如下： 
``` 
@SelectProvider(type=TutorDynaSqlProvider.class, method="findTutorByIdSql")  
Tutor findTutorById(long tutorId);  
``` 

这里我们使用了 `@SelectProvider` 来指定了一个类，及其内部的方法，用来提供需要执行的 SQL 语句。 

但是使用字符串拼接的方法来构建 SQL 语句是非常困难的，并且容易出错。所以MyBaits提供了一个SQL工具类不使用字符串拼接的方式，简化构造动态SQL语句。

现在，让我们看看如何使用 `org.apache.ibatis.jdbc.SQL` 工具类来准备相同的 SQL 语句： 
``` 
package com.mybatis3.sqlproviders;  
import org.apache.ibatis.jdbc.SQL;  
public class TutorDynaSqlProvider {  
    public String findTutorByIdSql(final long tutorId) {  
        return new SQL() 
        {  
            {  
                SELECT("tutor_id as tutorId, name, email");  
                FROM("tutors");  
                WHERE("tutor_id=" + tutorId);  
            }  
        }.toString();  
    }  
}
``` 

SQ L工具类会处理以合适的空格前缀和后缀来构造 SQL 语句。

动态 SQL provider 方法可以接收以下其中一种参数： 
- 无参数 
- 和映射器 Mapper 接口的方法同类型的参数 
- `java.util.Map` 

#### 使用不带参数的 SQL Provider 
``` 
public String findTutorByIdSql()  
{  
    return new SQL()  
    {  
        {  
            SELECT("tutor_id as tutorId, name, email");  
            FROM("tutors");  
            WHERE("tutor_id = #{tutorId}");  
        }  
    }.toString();  
}  
``` 

这里我们没有使用输入参数构造 SQL 语句，所以它可以是一个无参方法。 

#### 使用和映射器 Mapper 接口的方法同类型的参数 
如果映射器 Mapper 接口方法只有一个参数，那么可以定义 SQL Provider 方法，它接受一个与 Mapper 接口方法相同类型的参数。
例如映射器 Mapper 接口有如下定义： 
``` 
Tutor findTutorById(long tutorId); 
``` 

这里 findTutorById(int) 方法只有一个 long 类型的参数。我们可以定义 findTutorByIdSql(long) 方法作为 SQL provider 方法： 
``` 
public String findTutorByIdSql(final long tutorId)  
{  
    return new SQL()  
    {  
        {  
            SELECT("tutor_id as tutorId, name, email");  
            FROM("tutors");  
            WHERE("tutor_id=" + tutorId);  
        }  
    }.toString();  
} 
``` 

#### 使用 `java.util.Map` 参数类型的 SQL Provider 
如果映射器 Mapper 接口有多个输入参数，我们可以使用参数类型为 `java.util.Map` 的方法作为 SQL Provider 方法。然后映射器 Mapper 接口方法所有的输入参数将会被放到 map 中，以 param1，param2 等等作为 key，将输入参数按序作为 value。你也可以使用 0，1，2 等作为 key 值来取的输入参数： 
``` 
@SelectProvider(type = TutorDynaSqlProvider.class,  
                method = "findTutorByNameAndEmailSql")  
Tutor findTutorByNameAndEmail(String name, String email);  
    
public String findTutorByNameAndEmailSql(Map<String, Object> map) {  
    String name = (String) map.get("param1");  
    String email = (String) map.get("param2");  
    //you can also get those values using 0,1 keys  
    //String name = (String) map.get("0");  
    //String email = (String) map.get("1");  
    return new SQL()  
    {  
        {  
            SELECT("tutor_id as tutorId, name, email");  
            FROM("tutors");  
            WHERE("name=#{name} AND email=#{email}");  
        }  
    }.toString();  
} 
``` 

SQL 工具类也提供了其他的方法来表示 `JOINS` ， `ORDER_BY` ， `GROUP_BY` 等等。 
让我们看一个使用 `LEFT_OUTER_JOIN` 的例子： 
``` 
public class TutorDynaSqlProvider {  
    public String selectTutorById() {  
        return new SQL()  
        {  
            {  
                SELECT("t.tutor_id, t.name as tutor_name, email");  
                SELECT("a.addr_id, street, city, state, zip, country");  
                SELECT("course_id, c.name as course_name, description,  
                       start_date, end_date");  
                FROM("TUTORS t");  
                LEFT_OUTER_JOIN("addresses a on t.addr_id=a.addr_id");  
                LEFT_OUTER_JOIN("courses c on t.tutor_id=c.tutor_id");  
                WHERE("t.TUTOR_ID = #{id}");  
            }  
        }.toString();  
    }  
}  
  
public interface TutorMapper {  
    @SelectProvider(type = TutorDynaSqlProvider.class,  
                    method = "selectTutorById")  
    @ResultMap("com.mybatis3.mappers.TutorMapper.TutorResult")  
    Tutor selectTutorById(int tutorId);  
} 
``` 

由于没有支持使用内嵌结果 ResultMap 的一对多关联映射的注解支持，我们可以使用基于 XML 的 `<resultMap>` 配置，然后与 `@ResultMap` 映射： 
``` 
<mapper namespace="com.mybatis3.mappers.TutorMapper">  
  <resultMap type="Address" id="AddressResult">  
    <id property="id" column="addr_id" />  
    <result property="street" column="street" />  
    <result property="city" column="city" />  
    <result property="state" column="state" />  
    <result property="zip" column="zip" />  
    <result property="country" column="country" />  
  </resultMap>  
  <resultMap type="Course" id="CourseResult">  
    <id column="course_id" property="id" />  
    <result column="course_name" property="name" />  
    <result column="description" property="description" />  
    <result column="start_date" property="startDate" />  
    <result column="end_date" property="endDate" />  
  </resultMap>  
  <resultMap type="Tutor" id="TutorResult">  
    <id column="tutor_id" property="id" />  
    <result column="tutor_name" property="name" />  
    <result column="email" property="email" />  
    <association property="address" resultMap="AddressResult" />  
    <collection property="courses" resultMap="CourseResult"></collection>  
  </resultMap>  
</mapper>  
``` 

### @InsertProvider 
我们可以使用 `@InsertProvider` 注解创建动态的 INSERT 语句，如下所示： 
``` 
public class TutorDynaSqlProvider {  
    public String insertTutor(final Tutor tutor) {  
        return new SQL()  
        {  
            {  
                INSERT_INTO("TUTORS");  
                if (tutor.getName() != null)  
                {  
                    VALUES("NAME", "#{name}");  
                }  
                if (tutor.getEmail() != null)  
                {  
                    VALUES("EMAIL", "#{email}");  
                }  
            }  
        }.toString();  
    }  
}   
    
public interface TutorMapper {  
    @InsertProvider(type = TutorDynaSqlProvider.class,  
                    method = "insertTutor")  
    @Options(useGeneratedKeys = true, keyProperty = "tutorId")  
    int insertTutor(Tutor tutor);  
} 
```  

### @UpdateProvider 
我们可以通过 `@UpdateProvider` 注解创建 UPDATE 语句，如下所示： 
``` 
public class TutorDynaSqlProvider {  
    public String updateTutor(final Tutor tutor) {  
        return new SQL()  
        {  
            {  
                UPDATE("TUTORS");  
                if (tutor.getName() != null)  
                {  
                    SET("NAME = #{name}");  
                }  
                if (tutor.getEmail() != null)  
                {  
                    SET("EMAIL = #{email}");  
                }  
                WHERE("TUTOR_ID = #{tutorId}");  
            }  
        }.toString();  
    }  
}   
   
public interface TutorMapper {  
    @UpdateProvider(type = TutorDynaSqlProvider.class,  
                    method = "updateTutor")  
    int updateTutor(Tutor tutor);  
}
``` 

### @DeleteProvider 
我们可以使用 `@DeleteProvider` 注解创建动态地 DELETE 语句,如下所示： 
``` 
public class TutorDynaSqlProvider {  
    public String deleteTutor(long tutorId) {  
        return new SQL()  
        {  
            {  
                DELETE_FROM("TUTORS");  
                WHERE("TUTOR_ID = #{tutorId}");  
            }  
        } .toString();  
    }  
}  
   
public interface TutorMapper {  
    @DeleteProvider(type = TutorDynaSqlProvider.class,  
                    method = "deleteTutor")  
    int deleteTutor(long tutorId);  
}  
```  


