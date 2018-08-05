---
layout: post
title: Spring Data JPA native query with pagination
category : [Spring]
tagline: "Supporting tagline"
tags : [Spring Data JPA, native query, pagination]
---
{% include JB/setup %}
#  Spring Data JPA native query with pagination
---

## Mysql 
> [Spring Data JPA and native queries with pagination](http://www.denismigol.com/posts/44/spring-data-jpa-native-queries-pagination) 
> [Spring Data and Native Query with pagination](https://stackoverflow.com/questions/38349930/spring-data-and-native-query-with-pagination) 

``` 
public interface UserRepository extends JpaRepository<User, Long> {
    @Query(value = "select * from users order by id desc \n#pageable\n",
            countQuery = "select count(*) from users",
            nativeQuery = true)
    Page<User> findAllRandom(Pageable pageable);
}
```  
> 如果 pageable 对象中有排序，这里可以去掉 `order by` 排序。 
> `countQuery` 可以省略。 

## H2 
H2 数据库中要将 `\n#pageable\n` 改成 `\n-- #pageable\n` 形式。 

## Oracle 
> [Spring Data Jpa本地查询（带分页方式）](http://blog.csdn.net/tyyytcj/article/details/78152524)  

Oracle 数据库中要将 `\n#pageable\n` 改成 `?#{#pageable}` 形式。

<!--break-->    