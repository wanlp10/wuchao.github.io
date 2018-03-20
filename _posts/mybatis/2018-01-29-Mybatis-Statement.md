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
<insert id="batchSaveOrUpdate" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO favorite_group(id, name, user_id) VALUES
    <foreach collection="list" item="item" index="index" separator=",">
        (#{item.id}, #{item.name}, #{item.userId})
    </foreach>
</insert>
```  

最终的语句类似：  

```  
insert into favorite_group(id, name, user_id) values (1, 1, 1), (2, 2, 2), (3, 3, 3)
``` 

批量 insert 或 update：  
> [](http://blog.sina.com.cn/s/blog_13bde781c0102wjpd.html) 

``` 
<insert id="batchSaveOrUpdate" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO favorite_group(id, name, user_id) VALUES
    <foreach collection="list" item="item" index="index" separator=",">
        (#{item.id}, #{item.name}, #{item.userId})
    </foreach>
    ON DUPLICATE KEY UPDATE
    name = CASE id
    <foreach collection="list" item="item" index="index">
        WHEN #{item.id} THEN #{item.name}
    </foreach>
    END
</insert>
```  
 
> 如果有多个字段要改动，`END` 后面加逗号分隔。 

批量 delete： 
``` 
<delete id="batchDelete" parameterType="long">
    DELETE FROM favorite_group where id IN
    <foreach collection="array" item="id" open="(" separator="," close=")">
        #{groupIds}
    </foreach>
</delete>
``` 
最终的语句类似于： 
``` 
delete from favorite_group where id in (1, 2, 3) 
``` 