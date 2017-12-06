---
layout: post
title: Hibernate 持久化对象自动更新
category : [学习问题记录, Hibernate]
tagline: "Supporting tagline"
tags : [Hibernate]
---
{% include JB/setup %}
# Hibernate 持久化对象自动更新
---

> [hibernate持久化对象值改变后自动更新的条件](http://blog.csdn.net/wodatoucai/article/details/17427005) 

持久化对象在一级缓存中的值改变后，与所对应的快照区的数据不同，每当触发Session flush后，会主动更新数据库。
Session flush的三种情况为事务提交、调用查询方法(query)以及手动调用flush，session.close()。

<!--break-->

Spring MVC 提交表单，在 handler 方法的形参表中有一个持久化对象 `@ModelAttribute("currentUser") User user` ，由于表单元素有 name 属性和 User 对象的属性相同，这些相同的属性值也被赋值给了 user 对象，从而使 user 对象被更改了。然后在查询前 user 对象的更改被同步到了数据库中。


