---
layout: post
title: Hibernate 三种数据状态
category : [Hibernate]
tagline: "Supporting tagline"
tags : [Hibernate]
---
{% include JB/setup %}
# Hibernate 三种数据状态

<!--break--> 

## Hibernate 中的三种状态
瞬时状态：刚创建的对象还没有被 Session 持久化、缓存中不存在这个对象的数据并且数据库中没有这个对象对应的数据为瞬时状态这个时候是没有 OID。　　　

持久状态：对象经过 Session 持久化操作，缓存中存在这个对象的数据为持久状态并且数据库中存在这个对象对应的数据为持久状态这个时候有 OID。

游离状态：当 Session 关闭，缓存中不存在这个对象数据而数据库中有这个对象的数据并且有 OID 为游离状态。

注：OID 为了在系统中能够找到所需对象，我们需要为每一个对象分配一个唯一的表示号。在关系数据库中我们称之为关键字，而在对象术语中，则叫做对象标识

(Object identifier-OID).通常OID在内部都使用一个或多个大整数表示，而在应用程序中则提供一个完整的类为其他类提供获取、操作。

Hibernate 数据状态图：  
![Hibernate 数据状态图](/images/2018-05-16-hibernate-cache-status.png) 

需要注意的是：
当对象的临时状态将变为持久化状态。当对象在持久化状态时，它一直位于 Session 的缓存中，对它的任何操作在事务提交时都将同步到数据库，因此，对一个已经持久的对象调用 save() 或 update() 方法是没有意义的。

```
Student stu = new Strudnet();

stu.setCarId(“200234567”);

stu.setId(“100”);

// 打开 Session, 开启事务

// 将 stu 对象持久化操作
session.save(stu);

stu.setCardId(“20076548”);

// 再次对 stu 对象进行持久化操作
session.save(stu); // 无效

session.update(stu); // 无效

// 提交事务，关闭 Session
``` 

## Hibernate 三种状态之间的转化 
> Hibernate 向一级缓存放入数据时，同时复制一份数据放入到 Hibernate 快照中，
当使用 commit() 方法提交事务时，同时会清理 Session 的一级缓存，
这时会使用 OID 判断一级缓存中的对象和快照中的对象是否一致，如果两个对象中的属性发生变化，
则执行 update 语句，将缓存的内容同步到数据库，并更新快照；如果一致，则不执行 update 语句。 

### 1. 
``` 
session = HibernateUtil.openSession();
session.beginTransaction();
User user = new User();
user.setUsername("aaa");
user.setPassword("aaa");
user.setBorn(new Date());
/*
 *  以上 user 就是一个 Transient(瞬时状态),此时 user 并没有被 session 进行托管，即在 session 的
 *  缓存中还不存在 user 这个对象，当执行完 save 方法后，此时 user 被 session 托管，并且数据库中存在了该对象
 *  user 就变成了一个 Persistent(持久化对象)
 */
session.save(user);
session.getTransaction().commit();
```
执行 sql：
``` 
Hibernate: insert into t_user (born, password, username) values (?, ?, ?)
``` 

### 2. 
``` 
session = HibernateUtil.openSession();
session.beginTransaction();
User user = new User();
user.setUsername("aaa");
user.setPassword("aaa");
user.setBorn(new Date());
// 以上 u 就是 Transient（瞬时状态），表示没有被 session 管理并且数据库中没有该记录
// 执行 save 之后，被 session 所管理，而且，数据库中已经存在，此时就是 Persistent 状态
session.save(user);
// 此时 u 是持久化状态，已经被 session 所管理，当在提交时，会把 session 中的对象和目前的对象进行比较
// 如果两个对象中的值不一致就会继续发出相应的 sql 语句
user.setPassword("bbb");
// 此时会发出 2 条 sql，一条用户做插入，一条用来做更新
session.getTransaction().commit();
```


> 参考：[深入hibernate的三种状态](https://www.cnblogs.com/xiaoluo501395377/p/3380270.html)