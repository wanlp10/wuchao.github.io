---
layout: post
title: Mybatis - 使用XML配置SQL映射器 
category : [JavaEE]
tagline: "Supporting tagline"
tags : [Mybatis]
---
{% include JB/setup %}
# Mybatis - 使用XML配置SQL映射器 
---

> 引用：   
> [Java Persistence with MyBatis 3(中文版) 第三章 使用XML配置SQL映射器](http://blog.csdn.net/luanlouis/article/details/35594455) 

<!--break-->

## 映射器配置文件和映射器接口 
在com.mybatis3.mappers包中的StudentMapper.xml 配置文件内，有个 id 为”findStudentById” 的 SQL 语句： 
``` 
<?xml version="1.0" encoding="utf-8"?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="com.mybatis3.mappers.StudentMapper">  
  <select id="findStudentById" parameterType="long" resultType="Student">  
    select stud_id as studId, name, email, dob   
    from Students where stud_id=#{studId}  
  </select>  
</mapper>  
``` 

我们可以通过下列代码调用findStudentById映射的SQL语句: 
``` 
public Student findStudentById(long studId) {  
    SqlSession sqlSession = MyBatisUtil.getSqlSession();  
    try {  
        Student student =  
            sqlSession.selectOne("com.mybatis3.mappers.StudentMapper.  
                                 findStudentById", studId);  
        return student;  
    } finally {  
        sqlSession.close();  
    }  
} 
``` 

我们可以通过字符串（字符串形式为：映射器配置文件所在的包名namespace + 在文件内定义的语句id，如上，即包名com.mybatis3.mappers.StudentMapper 和语句idfindStudentById 组成）调用映射的SQL语句，但是这种方式容易出错。你需要检查映射器配置文件中的定义，以保证你的输入参数类型和结果返回类型是有效的。
MyBatis通过使用映射器Mapper接口提供了更好的调用映射语句的方法。一旦我们通过映射器配置文件配置了映射语句，我们可以创建一个完全对应的一个映射器接口，接口名跟配置文件名相同，接口所在包名也跟配置文件所在包名完全一样(如StudentMapper.xml所在的包名是com.mybatis3.mappers，对应的接口名就是com.mybatis3.mappers.StudentMapper.java）。映射器接口中的方法签名也跟映射器配置文件中完全对应：方法名为配置文件中id值；方法参数类型为parameterType对应值；方法返回值类型为returnType对应值。 

对于上述的 StudentMapper.xml 文件，我们可以创建一个映射器接口StudentMapper.java 如下： 
``` 
package com.mybatis3.mappers;  
public interface StudentMapper {  
    Student findStudentById(long id);  
} 
``` 

在StudentMapper.xml 映射器配置文件中，其名空间namespace 应该跟StudentMapper接口的完全限定名保持一致。另外，StudentMapper.xml中语句id，parameterType，returnType应该分别和StudentMapper接口中的方法名，参数类型，返回值相对应。 

使用映射器接口我们可以以类型安全的形式调用调用映射语句： 
``` 
public Student findStudentById(long studId) {
   SqlSession sqlSession = MyBatisUtil.getSqlSession();  
   try {  
       StudentMapper studentMapper =  
           sqlSession.getMapper(StudentMapper.class);  
       return studentMapper.findStudentById(studId);  
   } finally {  
       sqlSession.close();  
   }  
}    
``` 

## 映射语句 
MyBatis提供了多种元素来配置不同类型的语句，如SELECT，INSERT，UPDATE，DELETE。接下来让我们看看如何具体配置映射语句。 

### INSERT 语句 
一个INSERT SQL语句可以在<insert>元素在映射器XML配置文件中配置，如下所示： 
``` 
<insert id="insertStudent" parameterType="Student">  
    INSERT INTO STUDENTS(STUD_ID,NAME,EMAIL, PHONE)  
        VALUES(#{studId},#{name},#{email},#{phone})  
</insert>  
``` 

这里我们使用一个ID insertStudent，可以在名空间com.mybatis3.mappers.StudentMapper.insertStudent中唯一标识。parameterType属性应该是一个完全限定类名或者是一个类型别名（alias）。 

我们可以如下调用这个语句： 
``` 
int count = sqlSession.insert("com.mybatis3.mappers.StudentMapper.insertStudent", student);  
``` 

sqlSession.insert()方法返回执行INSERT语句后所影响的行数。

如果不使用名空间（namespace）和语句id来调用映射语句，你可以通过创建一个映射器Mapper接口，并以类型安全的方式调用方法，如下所示: 
``` 
package com.mybatis3.mappers;  
public interface StudentMapper {  
    int insertStudent(Student student);  
}  
``` 

然后可以如下调用 insertStudent 映射语句： 
``` 
StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);  
int count = mapper.insertStudent(student);   
```  

#### 自动生成主键 
在上述的INSERT语句中，我们为可以自动生成（auto-generated）主键的列STUD_ID插入值。我们可以使用useGeneratedKeys 和 keyProperty属性让数据库生成 auto_increment列的值，并将生成的值设置到其中一个输入对象属性内，如下所示： 
``` 
<insert id="insertStudent" parameterType="Student" useGeneratedKeys="true"  
         keyProperty="studId">  
    INSERT INTO STUDENTS(NAME, EMAIL, PHONE)   
    VALUES(#{name},#{email},#{phone})  
</insert> 
``` 

这里STUD_ID列值将会被MySQL数据库自动生成，并且生成的值会被设置到student对象的studId属性上。 

``` 
StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);  
mapper.insertStudent(student); 
```  

现在可以如下获取插入的 STUDENT 记录的 STUD_ID 的值： 
``` 
long studentId = student.getStudId();  
``` 

有些数据库如Oracle并不支持 AUTO_INCREMENT 列，其使用序列（SEQUENCE）来生成主键值。

假设我们有一个名为 STUD_ID_SEQ 的序列来生成SUTD_ID主键值。使用如下代码来生成主键： 
``` 
<insert id="insertStudent" parameterType="Student">  
    <selectKey keyProperty="studId" resultType="long" order="BEFORE">  
        SELECT ELEARNING.STUD_ID_SEQ.NEXTVAL FROM DUAL  
    </selectKey>  
    INSERT INTO STUDENTS(STUD_ID,NAME,EMAIL, PHONE)  
        VALUES(#{studId},#{name},#{email},#{phone})  
</insert>  
```  

这里我们使用了<selectKey>子元素来生成主键值，并将值保存到Student对象的studId属性上。 属性order=“before”表示MyBatis将取得序列的下一个值作为主键值，并且在执行INSERTSQL语句之前将值设置到studId属性上。

我们也可以在获取序列的下一个值时，使用触发器（trigger）来设置主键值，并且在执行INSERT SQL语句之前将值设置到主键列上。如果你采取这样的方式，则对应的INSERT映射语句如下所示： 
``` 
<insert id="insertStudent" parameterType="Student">  
    INSERT INTO STUDENTS(NAME,EMAIL, PHONE)  
        VALUES(#{name},#{email},#{phone})  
    <selectKey keyProperty="studId" resultType="long" order="AFTER">  
        SELECT ELEARNING.STUD_ID_SEQ.CURRVAL FROM DUAL  
    </selectKey>  
</insert>  
``` 

### UPDATE 语句 
一个UPDATE SQL语句可以在<update>元素在映射器XML配置文件中配置，如下所示： 
``` 
<update id="updateStudent" parameterType="Student">  
    UPDATE STUDENTS SET NAME=#{name}, EMAIL=#{email}, PHONE=#{phone}  
    WHERE STUD_ID=#{studId}  
</update>  
``` 

我们可以如下调用此语句： 
``` 
int noOfRowsUpdated = 
    sqlSession.update("com.mybatis3.mappers.StudentMapper.updateStudent", student);  
``` 

sqlSession.update() 方法返回执行 UPDATE 语句之后影响的行数。

如果不使用名空间（namespace）和语句id来调用映射语句，你可以通过创建一个映射器Mapper接口，并以类型安全的方式调用方法，如下所示： 
``` 
package com.mybatis3.mappers;  
public interface StudentMapper {  
    int updateStudent(Student student);  
} 
``` 

然后可以使用映射器Mapper接口来调用 updateStudent 语句： 
``` 
StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);  
int noOfRowsUpdated = mapper.updateStudent(student); 
``` 

### DELETE 语句 
一个 UPDATE SQL 语句可以在<update>元素在映射器 XML 配置文件中配置，如下所示： 
``` 
<delete id="deleteStudent" parameterType="long">  
   DELETE FROM STUDENTS WHERE STUD_ID=#{studId}  
</delete>   
``` 

我们可以如下调用此语句： 
``` 
long studId = 1;  
int noOfRowsDeleted =  
    sqlSession.delete("com.mybatis3.mappers.StudentMapper.deleteStudent", studId);  
``` 

sqlSession.delete()方法返回delete语句执行后影响的行数。 

如果不使用名空间（namespace）和语句id来调用映射语句，你可以通过创建一个映射器Mapper接口，并以类型安全的方式调用方法，如下所示：
``` 
package com.mybatis3.mappers;  
public interface StudentMapper {  
    int deleteStudent(long studId);  
}
``` 

然后以使用映射器Mapper接口来调用 updateStudent 语句： 
``` 
StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);  
int noOfRowsDeleted = mapper.deleteStudent(studId);  
``` 

### SELECT 语句  
MyBatis真正强大的功能，在于映射SELECT查询结果到JavaBeans 方面的极大灵活性。

让我们看看一个简单的select查询是如何（在MyBatis中）配置的，如下所示： 
``` 
<select id="findStudentById" parameterType="long"   
resultType="Student">  
    SELECT STUD_ID, NAME, EMAIL, PHONE   
        FROM STUDENTS   
    WHERE STUD_ID=#{studId}  
</select> 
``` 

我们可以如下调用此语句： 
``` 
long studId =1;  
Student student = 
    sqlSession.selectOne("com.mybatis3.mappers.StudentMapper.findStudentById", studId); 
``` 

如果不使用名空间（namespace）和语句id来调用映射语句，你可以通过创建一个映射器Mapper接口，并以类型安全的方式调用方法，如下所示：
``` 
package com.mybatis3.mappers;  
public interface StudentMapper {  
    Student findStudentById(long studId);  
} 
``` 

然后可以使用映射器 Mapper 接口来调用 updateStudent 语句，如下所示： 
``` 
StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);  
Student student = mapper.findStudentById(studId);  
``` 

如果你检查Student对象的属性值，你会发现studId属性值并没有被stud_id列值填充。这是因为MyBatis自动对JavaBean中和列名匹配的属性进行填充。这就是为什么name ,email,和phone属性被填充，而studId属性没有被填充。

为了解决这一问题，我们可以为列名起一个可以与JavaBean中属性名匹配的别名，如下所示： 
``` 
<select id="findStudentById" parameterType="long"   
resultType="Student">  
    SELECT STUD_ID AS studId, NAME,EMAIL, PHONE   
        FROM STUDENTS   
    WHERE STUD_ID=#{studId}  
</select> 
``` 

现在，Student 这个Bean对象中的值将会恰当地被stud_id,name,email,phone列填充了。

现在，让我们看一下如何执行返回多条结果的SELECT语句查询，如下所示： 
``` 
<select id="findAllStudents" resultType="Student">  
    SELECT STUD_ID AS studId, NAME,EMAIL, PHONE   
    FROM STUDENTS  
</select> 
``` 

我们可以如下调用此语句：  
``` 
List<Student> students =   
sqlSession.selectList("com.mybatis3.mappers.StudentMapper.findAllStudents"); 
``` 

映射器Mapper接口StudentMapper可以如下定义： 
``` 
package com.mybatis3.mappers;  
public interface StudentMapper {  
    List<Student> findAllStudents();  
} 
``` 

然后可以如下调用 findAllStudents 语句： 
``` 
StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);  
List<Student> students = mapper.findAllStudents();  
```

如果你注意到上述的SELECT映射定义，你可以看到，我们为所有的映射语句中的stud_id起了别名。

我们可以使用ResultMap，来避免上述的到处重复别名。我们稍后会继续讨论。

除了java.util.List，你也可以是由其他类型的集合类，如Set,Map，以及SortedSet。MyBatis根据集合的类型，会采用适当的集合实现，如下所示： 
- 对于List，Collection，Iterable类型，MyBatis将返回java.util.ArrayList
- 对于Map类型，MyBatis将返回java.util.HashMap
- 对于Set类型，MyBatis将返回 java.util.HashSet
- 对于SortedSet类型，MyBatis将返回java.util.TreeSet 

## 结果集映射 ResultMap 
ResultMap被用来 将SQL SELECT语句的结果集映射到 JavaBeans的属性中。我们可以定义结果集映射ResultMap并且在一些SELECT语句上引用resultMap。MyBatis的结果集映射ResultMap特性非常强大，你可以使用它将简单的SELECT语句映射到复杂的一对一和一对多关系的SELECT语句上。

### 简单 ResultMap 
一个映射了查询结果和 Student JavaBean 的简单的 resultMap 定义如下： 
``` 
<resultMap id="StudentResult" type="com.mybatis3.domain.Student">  
  <id property="studId" column="stud_id" />  
  <result property="name" column="name" />  
  <result property="email" column="email" />  
  <result property="phone" column="phone" />  
</resultMap>  
  
<select id="findAllStudents" resultMap="StudentResult">  
    SELECT * FROM STUDENTS  
</select>  
  
<select id="findStudentById" parameterType="long" resultMap="StudentResult">  
    SELECT * FROM STUDENTS WHERE STUD_ID=#{studId}  
</select>   
``` 

resultMap的id值StudentResult应该在此名空间内是唯一的。并且type属性应该是完全限定类名或者是返回类型的别名。

<result>子元素被用来将一个resultset列映射到JavaBean的一个属性中。

<id>元素和<result>元素功能相同，不过它被用来映射到唯一标识属性，用来区分和比较对象（一般和主键列相对应）。

在<select>语句中，我们使用了resultMap属性，而不是resultType来引用StudentResult映射。当<select>语句中配置了resutlMap属性，MyBatis会使用此数据库列名与对象属性映射关系来填充JavaBean中的属性。 

> resultType 和 resultMap 二者只能用其一，不能同时使用。 
> 
> resultType 指定一个实体类型，mybatis会自动将查询的结果填充到该实体对象中。 
> 
> resultMap 可以自定义将查询的结果填充到对象的属性，对象的关联对象和关联集合上。

让我们来看另外一个<select>映射语句定义的例子，怎样将查询结果填充到HashMap中。如下所示： 
``` 
<select id="findStudentById" parameterType="long" resultType="map">  
    SELECT * FROM STUDENTS WHERE STUD_ID=#{studId}  
</select>  
``` 

在上述的<select>语句中，我们将resultType配置成map，即java.util.HashMap的别名。在这种情况下，结果集的列名将会作为Map中的key值，而列值将作为Map的value值。 
``` 
HashMap<String,Object> studentMap = 
    sqlSession.selectOne("com.mybatis3.mappers.StudentMapper.findStudentById", studId);  
System.out.println("stud_id :"+studentMap.get("stud_id"));  
System.out.println("name :"+studentMap.get("name"));  
System.out.println("email :"+studentMap.get("email"));  
System.out.println("phone :"+studentMap.get("phone"));  
``` 

让我们再看一个 使用resultType=”map”,返回多行结果的例子： 
``` 
<select id="findAllStudents" resultType="map">  
    SELECT STUD_ID, NAME, EMAIL, PHONE FROM STUDENTS  
</select>  
``` 

由于resultType=”map”和语句返回多行，则最终返回的数据类型应该是List<HashMap<String,Object>>，如下所示： 
``` 
List<HashMap<String, Object>> studentMapList =  
    sqlSession.selectList("com.mybatis3.mappers.StudentMapper.findAllStudents");  
for(HashMap<String, Object> studentMap : studentMapList) {  
    System.out.println("studId :" + studentMap.get("stud_id"));  
    System.out.println("name :" + studentMap.get("name"));  
    System.out.println("email :" + studentMap.get("email"));  
    System.out.println("phone :" + studentMap.get("phone"));  
}  
``` 

### 拓展 ResultMap 
我们可以从另外一个<resultMap>，拓展出一个新的<resultMap>，这样，原先的属性映射可以继承过来。 
``` 
<resultMap type="Student" id="StudentResult">  
  <id property="studId" column="stud_id" />  
  <result property="name" column="name" />  
  <result property="email" column="email" />  
  <result property="phone" column="phone" />  
</resultMap>  
<resultMap type="Student" id="StudentWithAddressResult" extends="StudentResult">  
  <result property="address.addrId" column="addr_id" />  
  <result property="address.street" column="street" />  
  <result property="address.city" column="city" />  
  <result property="address.state" column="state" />  
  <result property="address.zip" column="zip" />  
  <result property="address.country" column="country" />  
</resultMap>  
```

id为StudentWithAddressResult的resultMap拓展了id为StudentResult的resultMap。 

如果你只想映射Student数据，你可以使用id为StudentResult的resultMap,如下所示： 
``` 
<select id="findStudentById" parameterType="long"   
resultMap="StudentResult">  
    SELECT * FROM STUDENTS WHERE STUD_ID=#{studId}  
</select>  
``` 

如果你想将映射Student数据和Address数据，你可以使用id为StudentWithAddressResult的resultMap： 
``` 
<select id="selectStudentWithAddress" parameterType="long"   
resultMap="StudentWithAddressResult">  
SELECT STUD_ID, NAME, EMAIL, PHONE, A.ADDR_ID, STREET, CITY,    
        STATE, ZIP, COUNTRY  
    FROM STUDENTS S LEFT OUTER JOIN ADDRESSES A ON   
            S.ADDR_ID=A.ADDR_ID  
    WHERE STUD_ID=#{studId}  
</select>  
``` 

## 一对一映射 
在我们的域模型样例中，每一个学生都有一个与之关联的地址信息。表STUDENTS有一个ADDR_ID列，是ADDRESSES表的外键。

Student 和 Address 的 JavaBean 以及映射器 Mapper XML 文件定义如下所示：  
``` 
public class Address {  
    private long addrId;  
    private String street;  
    private String city;  
    private String state;  
    private String zip;  
    private String country;  
    //setters & getters  
}  
public class Student {  
    private long studId;  
    private String name;  
    private String email;  
    private PhoneNumber phone;  
    private Address address;  
    //setters & getters  
} 
```

``` 
<resultMap type="Student" id="StudentWithAddressResult">  
  <id property="studId" column="stud_id" />  
  <result property="name" column="name" />  
  <result property="email" column="email" />  
  <result property="phone" column="phone" />  
  <result property="address.addrId" column="addr_id" />  
  <result property="address.street" column="street" />  
  <result property="address.city" column="city" />  
  <result property="address.state" column="state" />  
  <result property="address.zip" column="zip" />  
  <result property="address.country" column="country" />  
</resultMap>  
  
<select id="selectStudentWithAddress" parameterType="long"   
resultMap="StudentWithAddressResult">  
    SELECT STUD_ID, NAME, EMAIL, A.ADDR_ID, STREET, CITY, STATE,   
        ZIP, COUNTRY  
    FROM STUDENTS S LEFT OUTER JOIN ADDRESSES A ON   
        S.ADDR_ID=A.ADDR_ID  
    WHERE STUD_ID=#{studId}  
</select>  
``` 

我们可以使用圆点记法为内嵌的对象的属性赋值。在上述的resultMap中，Student的address属性使用了圆点记法被赋上了address对应列的值。同样地，我们可以访问任意深度的内嵌对象的属性。我们可以如下访问内嵌对象属性： 
``` 
//接口定义  
public interface StudentMapper{  
    Student selectStudentWithAddress(long studId);  
}  
  
  
//使用  
long studId = 1;  
StudentMapper studentMapper =  
    sqlSession.getMapper(StudentMapper.class);  
Student student = studentMapper.selectStudentWithAddress(studId);  
System.out.println("Student :" + student);  
System.out.println("Address :" + student.getAddress()); 
``` 

上述样例展示了一对一关联映射的一种方法。然而，使用这种方式映射，如果address结果需要在其他的SELECT映射语句中映射成Address对象，我们需要为每一个语句重复这种映射关系。MyBatis提供了更好地实现一对一关联映射的方法：嵌套结果ResultMap和嵌套select查询语句。接下来，我们将讨论这两种方式。 

### 使用嵌套结果 ResultMap 实现一对一关系映射 
我们可以使用一个嵌套结果ResultMap方式来获取Student及其Address信息，代码如下： 
``` 

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
  
<select id="findStudentWithAddress" parameterType="long"   
resultMap="StudentWithAddressResult">  
    SELECT STUD_ID, NAME, EMAIL, A.ADDR_ID, STREET, CITY, STATE,   
    ZIP, COUNTRY  
    FROM STUDENTS S LEFT OUTER JOIN ADDRESSES A ON   
    S.ADDR_ID=A.ADDR_ID  
    WHERE STUD_ID=#{studId}  
</select>  
``` 

元素<association>被用来导入“有一个”(has-one)类型的关联。在上述的例子中，我们使用了<association>元素引用了另外的在同一个XML文件中定义的<resultMap>。
我们也可以使用<association定义内联的resultMap，代码如下所示： 
``` 
<resultMap type="Student" id="StudentWithAddressResult">  
  <id property="studId" column="stud_id" />  
  <result property="name" column="name" />  
  <result property="email" column="email" />  
  <association property="address" javaType="Address">  
    <id property="addrId" column="addr_id" />  
    <result property="street" column="street" />  
    <result property="city" column="city" />  
    <result property="state" column="state" />  
    <result property="zip" column="zip" />  
    <result property="country" column="country" />  
  </association>  
</resultMap>  
``` 

使用嵌套结果ResultMap方式，关联的数据可以通过简单的查询语句（如果需要的话，需要与joins 连接操作配合）进行加载。 

### 使用嵌套查询实现一对一关系映射 
我们可以通过使用嵌套select查询来获取Student及其Address信息，代码如下： 
``` 
<resultMap type="Address" id="AddressResult">  
  <id property="addrId" column="addr_id" />  
  <result property="street" column="street" />  
  <result property="city" column="city" />  
  <result property="state" column="state" />  
  <result property="zip" column="zip" />  
  <result property="country" column="country" />  
</resultMap>  
  
<select id="findAddressById" parameterType="long"   
resultMap="AddressResult">  
    SELECT * FROM ADDRESSES WHERE ADDR_ID=#{id}  
</select>  
  
<resultMap type="Student" id="StudentWithAddressResult">  
  <id property="studId" column="stud_id" />  
  <result property="name" column="name" />  
  <result property="email" column="email" />  
  <association property="address" column="addr_id" select="findAddressById" />  
</resultMap>  
  
<select id="findStudentWithAddress" parameterType="long"   
resultMap="StudentWithAddressResult">  
    SELECT * FROM STUDENTS WHERE STUD_ID=#{Id}  
</select> 
``` 

在此方式中，<association>元素的 select属性被设置成了id为 findAddressById的语句。这里，两个分开的SQL语句将会在数据库中执行，第一个调用findStudentById加载student信息，而第二个调用findAddressById来加载address信息。

Addr_id列的值将会被作为输入参数传递给selectAddressById语句。

我们可以如下调用findStudentWithAddress映射语句： 
``` 
StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);  
Student student = mapper.selectStudentWithAddress(studId);  
System.out.println(student);  
System.out.println(student.getAddress());   
```  

## 一对多映射 
在我们的域模型样例中，一个讲师可以教授一个或者多个课程。这意味着讲师和课程之间存在一对多的映射关系。
我们可以使用<collection>元素将 一对多类型的结果 映射到 一个对象集合上。 

Course和Tutor的JavaBean定义如下： 
``` 
public class Course {  
    private long courseId;  
    private String name;  
    private String description;  
    private Date startDate;  
    private Date endDate;  
    private long tutorId;  
    //setters & getters  
}  
public class Tutor {  
    private long tutorId;  
    private String name;  
    private String email;  
    private Address address;  
    private List<Course> courses;  
    //setters & getters  
}  
```  

现在让我们看看如何获取讲师信息以及其所教授的课程列表信息。

<collection>元素被用来将多行课程结果映射成一个课程Course对象的一个集合。和一对一映射一样，我们可以使用嵌套结果ResultMap和嵌套Select语句两种方式映射实现一对多映射。 

### 使用内嵌结果 ResultMap 实现一对多映射 
我们可以使用嵌套结果resultMap方式获得讲师及其课程信息，代码如下： 
``` 
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
  <collection property="courses" resultMap="CourseResult" />  
</resultMap>  
  
<select id="findTutorById" parameterType="long"   
resultMap="TutorResult">  
SELECT T.TUTOR_ID, T.NAME AS TUTOR_NAME, EMAIL, C.COURSE_ID,   
C.NAME, DESCRIPTION, START_DATE, END_DATE  
FROM TUTORS T LEFT OUTER JOIN ADDRESSES A ON T.ADDR_ID=A.ADDR_ID  
LEFT OUTER JOIN COURSES C ON T.TUTOR_ID=C.TUTOR_ID  
WHERE T.TUTOR_ID=#{tutorId}  
</select>  
``` 

这里我们使用了一个简单的使用了JOINS连接的Select语句获取讲师及其所教课程信息。<collection>元素的resultMap属性设置成了CourseResult，CourseResult包含了Course对象属性与表列名之间的映射。 

### 使用嵌套 Select 语句实现一对多映射 
我们可以使用嵌套Select语句方式获得讲师及其课程信息，代码如下： 
``` 
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
  <collection property="courses" column="tutor_id" select="findCoursesByTutor" />  
</resultMap>  
  
<select id="findTutorById" parameterType="long" resultMap="TutorResult">  
    SELECT T.TUTOR_ID, T.NAME AS TUTOR_NAME, EMAIL   
    FROM TUTORS T WHERE T.TUTOR_ID=#{tutorId}  
</select>  
<select id="findCoursesByTutor" parameterType="long" resultMap="CourseResult">  
    SELECT * FROM COURSES WHERE TUTOR_ID=#{tutorId}  
</select> 
``` 

在这种方式中，<aossication>元素的select属性被设置为id为findCourseByTutor的语句，用来触发单独的SQL查询加载课程信息。tutor_id这一列值将会作为输入参数传递给findCouresByTutor语句。   
``` 
//接口定义
public interface TutorMapper {  
    Tutor findTutorById(long tutorId);  
}  


//使用
TutorMapper mapper = sqlSession.getMapper(TutorMapper.class);  
Tutor tutor = mapper.findTutorById(tutorId);  
System.out.println(tutor);  
List<Course> courses = tutor.getCourses();  
for (Course course : courses) {  
    System.out.println(course);  
} 
``` 

> 嵌套 Select 语句查询会导致 N + 1 问题。首先，主查询将会执行（1次），对于主查询返回的每一行，另外一个查询将会被执行（主查询 N 行，则此查询 N 次）。  

## 动态 SQL 
有时候，静态的SQL语句并不能满足应用程序的需求。我们可以根据一些条件，来动态地构建SQL语句。  

MyBatis通过使用<if>,<choose>,<where>,<foreach>,<trim>元素提供了对构造动态SQL语句的高级别支持。 

### If 条件 
<if>元素被用来有条件地嵌入SQL片段，如果测试条件被赋值为true，则相应地SQL片段将会被添加到SQL语句中。

假定我们有一个课程搜索界面，设置了 讲师（Tutor）下拉列表框，课程名称（CourseName）文本输入框，开始时间（StartDate）输入框，结束时间（EndDate）输入框，作为搜索条件。假定课讲师下拉列表是必须选的，其他的都是可选的。

当用户点击 搜索按钮时，我们需要显示符合以下条件的成列表： 
- 特定讲师的课程
- 课程名 包含输入的课程名称关键字的课程；如果课程名称输入为空，则取所有课程
- 在开始时间和结束时间段内的课程 

我们可以对应的映射语句，如下所示： 
``` 
<resultMap type="Course" id="CourseResult">  
  <id column="course_id" property="courseId" />  
  <result column="name" property="name" />  
  <result column="description" property="description" />  
  <result column="start_date" property="startDate" />  
  <result column="end_date" property="endDate" />  
</resultMap>  
  
<select id="searchCourses" parameterType="hashmap" resultMap="CourseResult"></select>  
    SELECT * FROM COURSES  
        WHERE TUTOR_ID= #{tutorId}  
    <if test="courseName != null">  
    AND NAME LIKE #{courseName}  
    </if>  
    <if test="startDate != null">  
    AND START_DATE >= #{startDate}  
    </if>  
    <if test="endDate != null">  
    AND END_DATE <= #{endDate}  
    </if>  
</select>  
``` 

``` 
public interface CourseMapper {  
    List<Course> searchCourses(Map<String, Object> map);  
}  

public void searchCourses() {  
    Map<String, Object> map = new HashMap<String, Object>();  
    map.put("tutorId", 1);  
    map.put("courseName", "%java%");  
    map.put("startDate", new Date());  
    CourseMapper mapper = sqlSession.getMapper(CourseMapper.class);  
    List<Course> courses = mapper.searchCourses(map);  
    for (Course course : courses) {  
        System.out.println(course);  
    }  
}  
``` 

此处将生成查询语句 SELECT * FROM COURSES WHERE TUTOR_ID= ? AND NAME like? AND START_DATE >= ?。 

> Mybatis 是使用 ONGL (Object Graph Navigation Language) 表达式来构建动态 SQL 语句。 

### choose，when 和 otherwise 条件 
有时候，查询功能是以查询类别为基础的。首先，用户需要选择是否希望通过选择讲师，课程名称，开始时间，或结束时间作为查询条件类别来进行查询，然后根据选择的查询类别，输入相应的参数。在这样的情景中，我们需要只使用其中一种查询类别。

MyBatis 提供了<choose>元素支持此类型的SQL预处理。

现在让我们书写一个适用此情景的SQL映射语句。如果没有选择查询类别，则查询开始时间在今天之后的课程，代码如下： 
``` 
<select id="searchCourses" parameterType="hashmap" resultMap="CourseResult">  
    SELECT * FROM COURSES  
    <choose>  
        <when test="searchBy == 'Tutor'">  
            WHERE TUTOR_ID= #{tutorId}  
        </when>  
        <when test="searchBy == 'CourseName'">  
            WHERE name like #{courseName}  
        </when>  
        <otherwise>  
            WHERE TUTOR start_date >= now()  
        </otherwise>  
    </choose>  
</select>  
``` 

MyBatis计算<choose>测试条件的值，且使用第一个值为TRUE的子句。如果没有条件为true，则使用<otherwise>内的子句。 

### where 条件 
有时候，所有的查询条件（criteria）应该是可选的。在需要使用至少一种查询条件的情况下，我们应该使用WHERE子句。并且， 如果有多个条件，我们需要在条件中添加AND或OR。MyBatis提供了<where>元素支持这种类型的动态SQL语句。

在我们查询课程界面，我们假设所有的查询条件是可选的。进而，当需要提供一个或多个查询条件时，应该改使用WHERE子句。 

``` 
<select id="searchCourses" parameterType="hashmap"   
resultMap="CourseResult">  
    SELECT * FROM COURSES  
    <where>   
        <if test=" tutorId != null ">  
            TUTOR_ID= #{tutorId}  
        </if>  
        <if test="courseName != null">  
            AND name like #{courseName}  
        </if>  
        <if test="startDate != null">  
            AND start_date >= #{startDate}  
        </if>  
        <if test="endDate != null">  
            AND end_date <= #{endDate}  
        </if>  
    </where>  
</select> 
``` 

<where>元素只有在其内部标签有返回内容时才会在动态语句上插入WHERE条件语句。并且，如果WHERE子句以AND或者OR打头，则打头的AND或OR将会被移除。

如果tutor_id参数值为null，并且courseName参数值不为null，则<where>标签会将AND name like#{courseName} 中的AND移除掉，生成的SQL WHERE子句为：where name like#{courseName}。 

### trim 条件 
<trim>元素和<where>元素类似，但是<trim>提供了在添加前缀/后缀 或者移除前缀/后缀方面提供更大的灵活性。 

``` 
<select id="searchCourses" parameterType="hashmap"   
resultMap="CourseResult">  
SELECT * FROM COURSES  
<trim prefix="WHERE" prefixOverrides="AND | OR">  
<if test=" tutorId != null ">  
TUTOR_ID= #{tutorId}  
</if>  
<if test="courseName != null">  
AND name like #{courseName}  
</if>  
</trim>  
</select> 
``` 

这里如果任意一个<if>条件为true，<trim>元素会插入WHERE,并且移除紧跟WHERE后面的AND或OR。 

### foreach 循环 
另外一个强大的动态SQL语句构造标签即是<foreach>。它可以迭代遍历一个数组或者列表，构造AND/OR条件或一个IN子句。

假设我们想找到tutor_id为1，3，6的讲师所教授的课程，我们可以传递一个tutor_id组成的列表给映射语句，然后通过<foreach>遍历此列表构造动态SQL。 
``` 
<select id="searchCoursesByTutors" parameterType="map"   
resultMap="CourseResult">  
SELECT * FROM COURSES  
<if test="tutorIds != null">  
<where>  
<foreach item="tutorId" collection="tutorIds">  
OR tutor_id=#{tutorId}  
</foreach>  
</where>   
</if>   
</select> 
```  

``` 
public interface CourseMapper {  
    List<Course> searchCoursesByTutors(Map<String, Object> map);  
}  

public void searchCoursesByTutors() {  
    Map<String, Object> map = new HashMap<String, Object>();  
    List<Long> tutorIds = new ArrayList<Long>();  
    tutorIds.add(1);  
    tutorIds.add(3);  
    tutorIds.add(6);  
    map.put("tutorIds", tutorIds);  
    CourseMapper mapper =  
        sqlSession.getMapper(CourseMapper.class);  
    List<Course> courses = mapper.searchCoursesByTutors(map);  
    for (Course course : courses) {  
        System.out.println(course);  
    }  
} 
```  

现在让我们来看一下怎样使用<foreach>生成 IN 子句： 
``` 
<select id="searchCoursesByTutors" parameterType="map"   
resultMap="CourseResult">  
    SELECT * FROM COURSES  
    <if test="tutorIds != null">  
        <where>  
        tutor_id IN  
            <foreach item="tutorId" collection="tutorIds"   
            open="(" separator="," close=")">  
            #{tutorId}  
            </foreach>  
        </where>  
    </if>  
</select>  
```

### set 条件 
<set>元素和<where>元素类似，如果其内部条件判断有任何内容返回时，他会插入SET SQL片段。 
``` 
<update id="updateStudent" parameterType="Student">  
    update students   
    <set>  
    <if test="name != null">name=#{name},</if>  
    <if test="email != null">email=#{email},</if>  
    <if test="phone != null">phone=#{phone},</if>  
    </set>  
    where stud_id=#{id}  
</update>  
``` 

这里，如果<if>条件返回了任何文本内容，<set>将会插入set关键字和其文本内容，并且会剔除将末尾的 “，”。 
在上述的例子中，如果 phone!=null,<set>将会让会移除  phone=#{phone}后的逗号“,”，生成 set phone=#{phone}。 

## Mybatis 补充说明 
除了简化数据库编程外，MyBatis还提供了各种功能，这些对实现一些常用任务非常有用，比如按页加载表数据，存取CLOB/BLOB类型的数据，处理枚举类型值，等等。 

### 处理枚举类型 
MyBatis支持开箱方式持久化enum 类型属性。假设STUDENTS表中有一列gender（性别）类型为varchar，存储”MALE”或者“FEMALE”两种值。并且，Student对象有一个enum 类型的gender属性，如下所示： 
``` 
public enum Gender {  
    FEMALE,  
    MALE  
} 
``` 

默认情况下，MyBatis使用EnumTypeHandler来处理enum类型的Java属性，并且将其存储为enum值的名称。你不需要为此做任何额外的配置。你可以可以向使用基本数据类型属性一样使用enum类型属性，代码如下： 
``` 
public class Student {  
    private Integer id;  
    private String name;  
    private String email;  
    private PhoneNumber phone;  
    private Address address;  
    private Gender gender;  
    //setters and getters  
} 
```  

``` 
<insert id="insertStudent" parameterType="Student"   
useGeneratedKeys="true" keyProperty="id">  
insert into students(name,email,addr_id, phone,gender)  
values(#{name},#{email},#{address.addrId},#{phone},#{gender})  
</insert>  
``` 

当你执行insertStudent语句的时候，MyBatis会取Gender枚举（FEMALE/MALE）的名称，然后将其存储到GENDER列中。
如果你希望存储原enum的顺序位置，而不是enum名，你需要明确地配置它。

如果你想存储FEMALE为0，MALE为1到gender列中，你需要在mybatis-config.xml文件中配置EnumOrdinalTypeHandler: 
``` 
<typeHandler   
handler="org.apache.ibatis.type.EnumOrdinalTypeHandler"  
javaType="com.mybatis3.domain.Gender"/>  
``` 

> 使用顺序位置为值存储到数据库时要当心。顺序值是根据 enum 中的生命顺序赋值的。如果你改变了 Gender enum 的声明顺序，则数据库存储的数据和顺序值就不匹配了。  

### 处理 CLOB/BLOB 类型数据 
MyBatis 提供了内建的对 CLOB/BLOB 类型列的映射处理支持。

假设我们有如下的表结构来存储学生和讲师的照片和简介信息： 
```  
CREATE TABLE USER_PICS   
(  
ID INT(11) NOT NULL AUTO_INCREMENT,  
NAME VARCHAR(50) DEFAULT NULL,  
PIC BLOB,  
BIO LONGTEXT,  
PRIMARY KEY (ID)  
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=LATIN1;   
``` 

这里，照片可以是PNG,JPG或其他格式的。简介信息可以是学生或者讲师的漫长的人生经历。默认情况下，MyBatis将CLOB类型的列映射到java.lang.String类型上、而把BLOB列映射到byte[]类型上。 

``` 
public class UserPic {  
    private long id;  
    private String name;  
    private byte[] pic;  
    private String bio;  
    //setters & getters  
}  
``` 

创建UserPicMapper.xml文件，配置映射语句，代码如下： 
``` 
<insert id="insertUserPic" parameterType="UserPic">  
    INSERT INTO USER_PICS(NAME, PIC,BIO)  
    VALUES(#{name},#{pic},#{bio})  
</insert>  
<select id="getUserPic" parameterType="long" resultType="UserPic">  
    SELECT * FROM USER_PICS WHERE ID=#{id}  
</select> 
```  

下列的insertUserPic()展示了如何将数据插入到CLOB/BLOB类型的列上：
``` 
public void insertUserPic() {  
    byte[] pic = null;  
    try {  
        File file = new File("C:\\Images\\UserImg.jpg");  
        InputStream is = new FileInputStream(file);  
        pic = new byte[is.available()];  
        is.read(pic);  
        is.close();  
    } catch (FileNotFoundException e) {  
        e.printStackTrace();  
    } catch (IOException e) {  
        e.printStackTrace();  
    }  
    
    String name = "UserName";  
    String bio = "put some lenghty bio here";  
    UserPic userPic = new UserPic(0, name, pic , bio);  
    SqlSession sqlSession = MyBatisUtil.openSession();  
    
    try {  
        UserPicMapper mapper =  
            sqlSession.getMapper(UserPicMapper.class);  
        mapper.insertUserPic(userPic);  
        sqlSession.commit();  
    } finally {  
        sqlSession.close();  
    }  
} 
``` 

下面的getUserPic()方法展示了怎样将CLOB类型数据读取到String类型，BLOB类型数据读取成byte[]属性： 
``` 
public void getUserPic() {  
    UserPic userPic = null;  
    SqlSession sqlSession = MyBatisUtil.openSession();  
    try {  
        UserPicMapper mapper =  
            sqlSession.getMapper(UserPicMapper.class);  
        userPic = mapper.getUserPic(1);  
    } finally {  
        sqlSession.close();  
    }  
    
    byte[] pic = userPic.getPic();  
    try {  
        OutputStream os = 
            new FileOutputStream(new File("C:\\Images\\UserImage_FromDB.jpg"));  
        os.write(pic);  
        os.close();  
    } catch (FileNotFoundException e) {  
        e.printStackTrace();  
    } catch (IOException e) {  
        e.printStackTrace();  
    }  
} 
``` 

### 传入多个输入参数 
MyBatis中的映射语句有一个parameterType属性来制定输入参数的类型。如果我们想给映射语句传入多个参数的话，我们可以将所有的输入参数放到HashMap中，将HashMap传递给映射语句。

MyBatis 还提供了另外一种传递多个输入参数给映射语句的方法。假设我们想通过给定的name和email信息查找学生信息，定义查询接口如下： 
``` 
Public interface StudentMapper {  
    List<Student> findAllStudentsByNameEmail(String name, String email);  
} 
```

MyBatis 支持将多个输入参数传递给映射语句，并以#{param}的语法形式引用它们： 
``` 
<select id="findAllStudentsByNameEmail" resultMap="StudentResult">  
    select stud_id, name,email, phone from Students  
        where name=#{param1} and email=#{param2}  
</select>  
```

这里#{param1}引用第一个参数name，而#{param2}引用了第二个参数email。 

``` 
StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);  
studentMapper.findAllStudentsByNameEmail(name, email);  
```

### 多行结果集映射成 Map 
如果你有一个映射语句返回多行记录，并且你想以HashMap的形式存储记录的值，使用记录列名作为key值，而记录对应值或为value值。我们可以使用sqlSession.selectMap(),如下所示： 
``` 
<select id=" findAllStudents" resultMap="StudentResult">  
    select * from Students  
</select>  
``` 

查询： 
``` 
Map<Long, Student> studentMap =   
sqlSession.selectMap("com.mybatis3.mappers.StudentMapper.findAllStudents", "studId");   
```  

这里studentMap将会将studId作为key值，而Student对象作为value值。 

### 使用 RowBounds 对结果集进行分页 
有时候，我们会需要跟海量的数据打交道，比如一个有数百万条数据级别的表。由于计算机内存的现实我们不可能一次性加载这么多数据，我们可以获取到数据的一部分。特别是在Web应用程序中，分页机制被用来以一页一页的形式展示海量的数据。

MyBatis可以使用RowBounds逐页加载表数据。RowBounds对象可以使用offset和limit参数来构建。参数offset表示开始位置，而limit表示要取的记录的数目。

假设如果你想每页加载并显示25条学生的记录，你可以使用如下的代码： 
``` 
<select id="findAllStudents" resultMap="StudentResult">  
    select * from Students  
</select> 
``` 

然后，你可以如下加载第一页数据（前25条）： 
``` 
int offset = 0 , limit = 25;  
RowBounds rowBounds = new RowBounds(offset, limit);  
List<Student> = studentMapper.getStudents(rowBounds);  
``` 

若要展示第二页，使用offset=25,limit=25;第三页，则为offset=50，limit=25。 

### 使用 ResultSetHandler 自定义处理结果集 resultSet 
MyBatis在将查询结果集映射到JavaBean方面提供了很大的选择性。但是，有时候我们会遇到由于特定的目的，需要我们自己处理SQL查询结果的情况。MyBatis提供了ResultHandler插件形式允许我们以任何自己喜欢的方式处理结果集ResultSet。

假设我们想从学生的stud_id被用作key，而name被用作value的HashMap中获取到student信息。 

> mybatis-3.2.2并不支持使用resultMap配置将查询的结果集映射成一个属性为key，而另外属性为value的HashMap。 
> sqlSession.selectMap()则可以返回以给定列为key，记录对象为value的map。
> 我们不能将其配置成其中一个属性作为key，而另外的属性作为value。 
 
对于sqlSession.select()方法，我们可以传递给它一个ResultHandler的实现，它会被调用来处理ResultSet的每一条记录。 
``` 
public interface ResultHandler {   
    void handleResult(ResultContext context);  
} 
``` 

现在我们来看一下怎么使用ResultHandler来处理结果集ResultSet，并返回自定义化的结果。 
``` 
public Map<Long, String> getStudentIdNameMap() {  

    final Map<Long, String> map = new HashMap<Long, String>();  
    SqlSession sqlSession = MyBatisUtil.openSession();  
    
    try {  
        sqlSession.select("com.mybatis3.mappers.StudentMapper.findAllStudents"
            , new ResultHandler() {  
                @Override  
                public void handleResult(ResultContext context) {  
                    Student student = (Student) context.getResultObject();  
                    map.put(student.getStudId(), student.getName());  
                }  
            }  
    } finally {  
        sqlSession.close();  
    }  
    
    return map;  
}  
``` 

在上述的代码中，我们提供了匿名内部ResultHandler实现类，在handleResult()方法中，我们使用context.getResultObject()获取当前的result对象，即Student对象，因为我们定义了findAllStudent映射语句的resultMap=”studentResult“。对查询返回的每一行都会调用handleResult()方法，并且我们从Student对象中取出studId和name，将其放到map中。 

### 缓存 
将从数据库中加载的数据缓存到内存中，是很多应用程序为了提高性能而采取的一贯做法。MyBatis对通过映射的SELECT语句加载的查询结果提供了内建的缓存支持。默认情况下，启用一级缓存；即，如果你使用同一个SqlSession接口对象调用了相同的SELECT语句，则直接会从缓存中返回结果，而不是再查询一次数据库。

我们可以在 SQL 映射器 XML 配置文件中使用 `<cache />` 元素添加全局二级缓存。

当你加入了 `<cache />` 元素，将会出现以下情况： 
- 所有的在映射语句文件定义的 `<select>` 语句的查询结果都会被缓存
- 所有的在映射语句文件定义的 `<insert>` , `<update>` 和 `<delete>` 语句将会刷新缓存
- 缓存根据最近最少被使用（Least Recently Used，LRU）算法管理
- 缓存不会被任何形式的基于时间表的刷新（没有刷新时间间隔），即不支持定时刷新机制
- 缓存将存储1024个查询方法返回的列表或者对象的引用
- 缓存会被当作一个读/写缓存。这是指检索出的对象不会被共享，并且可以被调用者安全地修改，不会有其他潜在的调用者或者线程的潜在修改干扰，即，缓存是线程安全的。 

你也可以通过复写默认属性来自定义缓存的行为，如下所示： 
``` 
<cache eviction="FIFO" flushInterval="60000" size="512"   
readOnly="true"/>  
``` 

以下是对上述属性的描述： 
- eviction:此处定义缓存的移除机制。默认值是LRU，其可能的值有：LRU（least recently used,最近最少使用），FIFO(first in first out,先进先出)，SOFT(soft reference,软引用)，WEAK（weak reference,弱引用）。
- flushInterval:定义缓存刷新间隔，以毫秒计。默认情况下不设置。所以不使用刷新间隔时，缓存cache只有在调用语句的时候刷新。
- size:表示缓存cache中能容纳的最大元素数。默认值是1024，你也可以设置成任意的正整数。
- readOnly:一个只读的缓存cache会对所有的调用者返回被缓存对象的同一个实例（实际返回的是被返回对象的一份引用）。一个读/写缓存cache将会返回被返回对象的一分拷贝（通过序列化）。默认情况下设置为false。可能的值有false和true。

一个缓存的配置和缓存实例被绑定到映射器配置文件所在的名空间（namespace）上，所以在相同名空间内的所有语句被绑定到一个cache中。

默认的映射语句的cache配置如下： 
``` 
<select ... flushCache="false" useCache="true"/>  
<insert ... flushCache="true"/>  
<update ... flushCache="true"/>  
<delete ... flushCache="true"/> 
``` 

你可以为任意特定的映射语句复写默认的cache行为；例如，对一个select语句不使用缓存，可以设置useCache=“false”。

除了内建的缓存支持，MyBatis也提供了与第三方缓存类库如 `Ehcache` ， `OSCache` ， `Hazelcast` 的集成支持。你可以在 [MyBatis官方网站](https://code.google.com/p/mybatis/wiki/Caches) 上找到关于继承第三方缓存类库的更多信息。  














