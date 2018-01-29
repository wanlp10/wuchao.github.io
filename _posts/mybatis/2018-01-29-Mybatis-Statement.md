---
layout: post
title: Mybatis 语句
category : [Mybatis]
tagline: "Supporting tagline"
tags : [Mybatis]
---
{% include JB/setup %}
# Mybatis 语句
---


<!--break-->


### 批量操作：
> [Mybatis 插入和删除批处理操作](http://www.jb51.net/article/99134.htm)  
> 
> [springMVC 接收数组参数，mybatis 接收数组参数，mybatis批量插入/批量删除案例](https://www.cnblogs.com/digdeep/p/4589037.html)
> 
> [SQL中的case when then else end用法](https://www.cnblogs.com/prefect/p/5746624.html) 

批量 insert： 
``` 
<insert id="batchInsert" parameterType="Test">
    INSERT INTO 
    test_table(test_x, test_y, test_z)
    VALUES
    <foreach item="item" index="index" collection="list" open="(" close=")" separator=",">
        #{item}.x, #{item.y}, #{item}.z
    </foreach>
</insert>
``` 
最终的语句类似： 
```  
insert into test_table(x, y, z) values (1, 1, 1), (2, 2, 2), (3, 3, 3)
``` 

批量 insert 或 update： 
``` 
<insert id="batchInsertOrUpdate" parameterType="Test">
    INSERT INTO 
    test_table(test_x, test_y, test_z)
    VALUES
    <foreach item="item" index="index" collection="list" open="(" close=")" separator=",">
        #{item}.x, #{item.y}, #{item}.z
    </foreach>
    ON DUPLICATE KEY UPDATE
    test_x = VALUES(test_x),
    test_y = VALUES(test_y),
    test_z = VALUES(test_z)
    END 
</insert>
```  

> [](http://blog.sina.com.cn/s/blog_13bde781c0102wjpd.html) 
> 如果有多个字段要改动，`END` 后面加逗号分隔。 

批量 delete： 
``` 
<delete id="deleteTestList" parameterType="Test">
    DELETE FROM 
    test_table
    WHERE
    <foreach item="item" index="index" collection="list" open="(" close=")" separator="or">
        test_x = #{item.x} AND test_y = #{item.y} AND test_z = #{item.z}
    </foreach>
</delete>
``` 
最终的语句类似于： 
``` 
delete from test_table where (test_x = 1 AND test_y = 1 AND test_z = 1) or (test_x = 2 AND test_y = 2 AND test_z = 2) or (test_x = 3 AND test_y = 3 AND test_z = 3) 
``` 
> 上面的代码为 x,y,z 为联合主键的情况，普通情况使用 `where id in`。 