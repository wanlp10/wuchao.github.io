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

stu.setCardId("20076548");

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
session.save(user);
user.setPassword("bbb");
session.getTransaction().commit();
``` 
执行 sql：
``` 
Hibernate: insert into t_user (born, password, username) values (?, ?, ?)
Hibernate: update t_user set born=?, password=?, username=? where id=?
```  

### 3. 
``` 
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
session = HibernateUtil.openSession();
session.beginTransaction();
User u = new User();
u.setBorn(new Date());
u.setUsername("zhangsan");
u.setPassword("zhangsan");
session.save(u);
u.setPassword("222");
// 该条语句没有意义
session.save(u);
u.setPassword("zhangsan111");
// 没有意义
session.update(u);
u.setBorn(sdf.parse("1988-12-22"));
// 没有意义
session.update(u);
session.getTransaction().commit();
``` 
执行 sql：
``` 
Hibernate: insert into t_user (born, password, username) values (?, ?, ?)
Hibernate: update t_user set born=?, password=?, username=? where id=?
``` 

### 4. 
``` 
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
session = HibernateUtil.openSession();
session.beginTransaction();
User u = new User();
u.setBorn(sdf.parse("1976-2-3"));
u.setUsername("zhangsan2");
u.setPassword("zhangsan2");
session.save(u);
/*
 * 以下三条语句没有任何意义
 */
session.save(u);
session.update(u);
session.update(u);
u.setUsername("zhangsan3");
session.getTransaction().commit();
``` 
执行 sql：
``` 
Hibernate: insert into t_user (born, password, username) values (?, ?, ?)
Hibernate: update t_user set born=?, password=?, username=? where id=?
``` 

### 5. 
``` 
session = HibernateUtil.openSession();
session.beginTransaction();
User u = (User)session.load(User.class, 4);
u.setUsername("bbb");
session.getTransaction().commit();
``` 
执行 sql：
``` 
Hibernate: select user0_.id as id0_0_, user0_.born as born0_0_, user0_.password as password0_0_, user0_.username as username0_0_ from t_user user0_ where user0_.id=?
Hibernate: update t_user set born=?, password=?, username=? where id=?
```

### 6. 
``` 
session = HibernateUtil.openSession();
session.beginTransaction();
User u = (User)session.load(User.class, 4);
u.setUsername("123");
session.clear();
session.getTransaction().commit();
``` 
执行 sql：
``` 
Hibernate: select user0_.id as id0_0_, user0_.born as born0_0_, user0_.password as password0_0_, user0_.username as username0_0_ from t_user user0_ where user0_.id=?
```

### 7. 
``` 
session = HibernateUtil.openSession();
session.beginTransaction();
User u = new User();
u.setId(4);
u.setPassword("hahahaha");
session.save(u);
session.getTransaction().commit();
``` 
执行 sql：
``` 
Hibernate: insert into t_user (born, password, username) values (?, ?, ?)
``` 

### 8. 
``` 
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
session = HibernateUtil.openSession();
session.beginTransaction();
User u = new User();
u.setId(5);
session.update(u);
u.setBorn(sdf.parse("1998-12-22"));
u.setPassword("world");
u.setUsername("world");
session.update(u);
session.getTransaction().commit();
``` 
执行 sql：
``` 
Hibernate: update t_user set born=?, password=?, username=? where id=? 
``` 

### 9. 
``` 
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
session = HibernateUtil.openSession();
session.beginTransaction();
User u = new User();
u.setId(5);
session.update(u);
u.setBorn(sdf.parse("1998-12-22"));
u.setPassword("lisi");
u.setUsername("lisi");
u.setId(333);
session.getTransaction().commit();
``` 
执行 sql：
``` 
org.hibernate.HibernateException: identifier of an instance of com.xiaoluo.bean.User was altered from 5 to 333
``` 

### 10. 
``` 
session = HibernateUtil.openSession();
session.beginTransaction();
User u = new User();
u.setId(5);
session.delete(u);
u.setPassword("wangwu");
session.getTransaction().commit();
``` 
执行 sql：
``` 
Hibernate: delete from t_user where id=?
``` 

### 11. 
``` 
session = HibernateUtil.openSession();
session.beginTransaction();
User u = new User();
u.setId(4);
u.setPassword("zhaoliu");
session.saveOrUpdate(u);
session.getTransaction().commit();
``` 
执行 sql：
``` 
Hibernate: delete from t_user where id=?
``` 
这里我们来看看 saveOrUpdate这个方法，这个方法其实是一个"偷懒"的方法，如果对象是一个离线对象，那么在执行这个方法后，其实是调用了update方法，如果对象是一个瞬时对象，则会调用save方法，记住：如果对象设置了ID值，例如u.setId(4)，那么该对象会被假设当作一个离线对象，此时就会执行update操作。
``` 
Hibernate: update t_user set born=?, password=?, username=? where id=?
```
如果此时我将u.setId(4)这句话注释掉，那么此时u就是一个瞬时的对象，那么此时就会执行save操作，就会发送一条insert语句
``` 
Hibernate: insert into t_user (born, password, username) values (?, ?, ?)
```

### 12. 
``` 
session = HibernateUtil.openSession();
session.beginTransaction();
User u1 = (User)session.load(User.class, 3);
System.out.println(u1.getUsername());
User u2 = new User();
u2.setId(3);
u2.setPassword("123456789");
session.saveOrUpdate(u2);
``` 
我们再来看一下这个例子，此时我们的u1已经是持久化的对象了，保存在session缓存中，u2通过调用saveOrUpdate方法后也变成了一个持久化的对象，此时也会保存在session缓存中，这个时候session缓存中就存在了一个持久化对象有两个引用拷贝了，这个时候hibernate就会报错 
``` 
org.hibernate.NonUniqueObjectException: a different object with the same identifier value was already associated with the session: [com.xiaoluo.bean.User#3]
``` 
一个session中不能存在对一个持久化对象的双重copy的，要解决这个方法，我们这里又要介绍session的另一个方法  merge方法，这个方法的作用就是解决一个持久化对象两分拷贝的问题，这个方法会将两个对象合并在一起成为一个对象。
``` 
session = HibernateUtil.openSession();
session.beginTransaction();
User u1 = (User)session.load(User.class, 3);
System.out.println(u1.getUsername());
User u2 = new User();
u2.setId(3);
u2.setPassword("123456789");
session.merge(u2);
session.getTransaction().commit();
``` 
我们看到通过调用了merge方法以后，此时会将session中的两个持久化对象合并为一个对象，但是merge方法不建议被使用
``` 
Hibernate: select user0_.id as id0_0_, user0_.born as born0_0_, user0_.password as password0_0_, user0_.username as username0_0_ from t_user user0_ where user0_.id=?
zhangsan
Hibernate: update t_user set born=?, password=?, username=? where id=?
``` 

总结：  

①.对于刚创建的一个对象，如果session中和数据库中都不存在该对象，那么该对象就是瞬时对象(Transient)。

②.瞬时对象调用save方法，或者离线对象调用update方法可以使该对象变成持久化对象，如果对象是持久化对象时，那么对该对象的任何修改，都会在提交事务时才会与之进行比较，如果不同，则发送一条update语句，否则就不会发送语句。

③.离线对象就是，数据库存在该对象，但是该对象又没有被session所托管。


> 参考： 
> 
> [深入hibernate的三种状态](https://www.cnblogs.com/xiaoluo501395377/p/3380270.html)