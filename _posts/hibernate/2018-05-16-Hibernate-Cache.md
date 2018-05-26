---
layout: post
title: Hibernate 缓存机制
category : [Hibernate]
tagline: "Supporting tagline"
tags : [Hibernate, Cache]
---
{% include JB/setup %}
# Hibernate 缓存机制

<!--break--> 

## Hibernate 缓存机制
什么是缓存？

　　缓存是介于应用程序和物理数据源之间，其作用是为了降低应用程序对物理数据源访问的频次，从而提高了应用的运行性能。缓存内的数据是对物理数据源中的数据的复制，应用程序在运行时从缓存读写数据，在特定的时刻或事件会同步缓存和物理数据源的数据。　

缓存有什么好处？

　　缓存的好处是降低了数据库的访问次数，提高应用性能，减少了读写数据的时间。

什么时候适合用缓存？

　　程序中经常用到一些不变的数据内容，从数据库查出来以后不会去经常修改它而又经常要用到的就可以考虑做一个缓存，以后读取就从缓存来读取，而不必每次都去查询数据库。因为硬盘的速度比内存的速度慢的多。从而提高了程序的性能，缓存的出现就会为了解决这个问题。


Hibernate中的缓存包括一级缓存（Session缓存）、二级缓存（SessionFactory缓存）和查询缓存。

### 一级缓存（Session 缓存）
由于 Session 对象的生命周期通常对应一个数据库事务或者一个应用事务，因此它的缓存是事务范围的缓存。Session 级缓存是必需的，不允许而且事实上也无法卸除。在 Session 级缓存中，持久化类的每个实例都具有唯一的 OID。

当应用程序调用 Session 的 save()、update()、savaeOrUpdate()、get() 或 load()，以及调用查询接口的 list()、iterate() 或 filter() 方法时，如果在 Session 缓存中还不存在相应的对象，Hibernate 就会把该对象加入到第一级缓存中。

当清理缓存时，Hibernate 会根据缓存中对象的状态变化来同步更新数据库。

Session 为应用程序提供了两个管理缓存的方法：

- evict(Object obj)：从缓存中清除参数指定的持久化对象。

- clear()：清空缓存中所有持久化对象。

```
public static void testOneLeveCache(){
    Session session = HibernateUtil.getSession();
    // 获取持久化对象 Dept
    Dept d1 = (Dept)session.get(Dept.class, 10);
    // 再次获取持久化对象 Dept
    Dept d2 = (Dept)session.get(Dept.class, 10);    
    session.close();
}
```
通过 Session 的 get() 方法获取到了 Dept 对象默认会将 Dept 对象保存到一级缓存中（Session 缓存） 当第二次获取的时候会先从一级缓存中查询对应的对象（前提是不能清空或关闭 Session，否则一级缓存会清空或销毁），如果一级缓存中存在相应的对象就不会到数据库中查询。所以只执行一次查询的查询代码如下：
```
Hibernate:
    select
        dept0_.DEPTNO as DEPTNO0_0_,
        dept0_.DNAME as DNAME0_0_,
        dept0_.LOC as LOC0_0_
    from
        SCOTT.DEPT dept0_
    where
        dept0_.DEPTNO=?
```

如何清除一级缓存？

　　通过 session.clear(); // 清除所有缓存

　　　　 session.evict(); // 清除指定缓存

```
public static void testOneLeveCache() {
    Session session = HibernateUtil.getSession();
    SessionFactory sf = HibernateUtil.getSessionFactory();
    // 获取持久化对象 Dept
    Dept d1 = (Dept)session.get(Dept.class, 10);
    session.clear(); // 清除所有缓存
    session.evict(d1); // 清除指定缓存
    // 再次获取持久化对象 Dept
    Dept d2 = (Dept)session.get(Dept.class, 10);    
    session.close();
}
```
使用 session.evict(Object obj) 会删除指定的 Bean，所以当你查询被你删除二级缓存的 Bean 时也会执行两条SQL语句。

使用 Session.clear() 清除后会发现执行了两条 SQL 语句：
```
Hibernate:
    select
        dept0_.DEPTNO as DEPTNO0_0_,
        dept0_.DNAME as DNAME0_0_,
        dept0_.LOC as LOC0_0_
    from
        SCOTT.DEPT dept0_
    where
        dept0_.DEPTNO=?
Hibernate:
    select
        dept0_.DEPTNO as DEPTNO0_0_,
        dept0_.DNAME as DNAME0_0_,
        dept0_.LOC as LOC0_0_
    from
        SCOTT.DEPT dept0_
    where
        dept0_.DEPTNO=?
```

### 二级缓存（SessionFactory 缓存）  
由于 SessionFactory 对象的生命周期和应用程序的整个过程对应，因此 Hibernate 二级缓存是进程范围或者集群范围的缓存，有可能出现并发问题，因此需要采用适当的并发访问策略，该策略为被缓存的数据提供了事务隔离级别。

　　save、update、saveOrupdate、load、get、list、query、Criteria方法都会填充二级缓存。

　　get、load、iterate 会从二级缓存中取数据 session.save(user)   

　　如果 user 主键使用 "native" 生成，则不放入二级缓存。

　　第二级缓存是可选的，是一个可配置的插件，默认下 SessionFactory 不会启用这个插件。Hibernate 提供了 `org.hibernate.cache.CacheProvider` 接口，它充当缓存插件与 Hibernate 之间的适配器。

Hibernate的二级缓存策略的一般过程如下：  
  1.) 条件查询的时候，总是发出一条 select * from table_name where …. （选择所有字段）这样的 SQL 语句查询数据库，一次获得所有的数据对象。  

  2.) 把获得的所有数据对象根据 ID 放入到第二级缓存中。  

  3.) 当 Hibernate 根据 ID 访问数据对象的时候，首先从 Session 一级缓存中查；查不到，如果配置了二级缓存，那么从二级缓存中查；查不到，再查询数据库，把结果按照 ID 放入到缓存。   

  4.) 删除、更新、增加数据的时候，同时更新缓存。  

　  Hibernate 的二级缓存策略，是针对于 ID 查询的缓存策略，对于条件查询则毫无作用。为此，Hibernate 提供了针对条件查询的查询缓存（Query Cache）。


#### 配置二级缓存（SessionFactory 缓存）
在 hibernate.cfg.xml 中配置以下代码：  
```
<!-- 开启二级缓存 -->
<property name="hibernate.cache.use_second_level_cache">true</property>
<!-- 为hibernate指定二级缓存的实现类 -->
<property name="hibernate.cache.provider_class">org.hibernate.cache.EhCacheProvider</property>
```  

指明哪些类需要放入二级缓存，需要长期使用到的对象才有必要放入二级缓存，放入二级缓存的方式有两种：
1.在 hibernate.cfg.xml 中配置
```
<class-cache class="entity.PetInfo" usage="read-only" /> // 不允许更新缓存中的对象
<class-cache class="entity.PetInfo" usage="read-write" />  // 允许更新缓存中的对象
```

2.在 Bean.hbm 文件中配置
```
<hibernate-mapping>
    <class name="com.bdqn.entity.Dept" table="DEPT" schema="SCOTT"  >
        <cache usage="read-only"/>//将这个类放入二级缓存
        <id name="deptno" type="java.lang.Integer">
            <column name="DEPTNO" precision="2" scale="0" />
            <generator class="assigned"></generator>
        </id>
        <property name="dname" type="java.lang.String">
            <column name="DNAME" length="14" />
        </property>
    </class>
</hibernate-mapping>
```

在 ehcache.xml 配置文件中可以设置缓存的最大数量、是否永久有效、时间等。
```
<defaultCache
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="120"
        timeToLiveSeconds="120"
        overflowToDisk="true"
        />
```

#### 如何清除二级缓存
需要用 SessionFactory 来管理二级缓存代码如下：
```
// sessionFactory.evict(Entity.class); // 清除所有 Entity
// sessionFactory.evict(Entity.class, id); // 清除指定 Entity

SessionFactory factory=HibernateUtils.getSessionFactory();

// evict() 把 id 为 1 的 Student 对象从二级缓存中清除
factory.evict(Student.class, 1);
// evict() 清除所有二级缓存
factory.evict(Student.class);
```

什么样的数据适合存放到第二级缓存中？      
　　1) 很少被修改的数据   
　　2) 不是很重要的数据，允许出现偶尔并发的数据   
　　3) 不会被并发访问的数据   
　　4) 常量数据

不适合存放到第二级缓存的数据？  
　　1) 经常被修改的数据  
　　2) 绝对不允许出现并发访问的数据，如财务数据，绝对不允许出现并发  
　　3) 与其他应用共享的数据  


### 查询缓存（Query Cache）
hibernate 的查询缓存是主要是针对普通属性结果集的缓存，而对于实体对象的结果集只缓存 id。

在一级缓存，二级缓存和查询缓存都打开的情况下作查询操作时这样的：

查询普通属性，会先到查询缓存中取，如果没有，则查询数据库；查询实体，会先到查询缓存中取 id，如果有，则根据 id 到缓存（一级/二级）中取实体，如果缓存中取不到实体，再查询数据库。

在 hibernate.cfg.xml 配置文件中，开启查询缓存　
```
<!-- 是否开启查询缓存,true开启查询缓存，false关闭查询缓存 -->
<property name="cache.use_query_cache">true</property>
```

开启查询缓存后还需要在程序中进行启用查询缓存
```
public static void testQueryCache() {
    Session session = HibernateUtil.getSession();
    String hql = "from Emp as e";
    Query query = session.createQuery(hql);
    query.setCacheable(true); // 启用查询缓存（二级缓存）
    List<Emp> empList = query.list();
    session.close();
}
```

查询缓存是基于二级缓存机制，如果根据 Bean 的属性查询，可以不开启二级缓存，代码如下：
```
session = HibernateUtils.getSession();
t = session.beginTransaction();
Query query = session.createQuery("select s.name from Student s");
// 启用查询缓存    
query.setCacheable(true);
List<String> names = query.list();
for (Iterator<String> it = names.iterator(); it.hasNext();) {
  String name = it.next();
  System.out.println(name);
}
System.out.println("================================");
query = session.createQuery("select s.name from Student s");
// 启用查询缓存
query.setCacheable(true);
// 没有发出查询语句,因为这里使用的查询缓存
names = query.list();
for (Iterator<String> it = names.iterator(); it.hasNext();) {
  String name = it.next();
  System.out.println(name);
}
t.commit();
```

> 参考：  
> 
> [Java三大框架之——Hibernate中的三种数据持久状态和缓存机制](https://www.cnblogs.com/hcl22/p/6100191.html)
> 
> [Hibernate object states](https://docs.jboss.org/hibernate/orm/3.3/reference/en/html/objectstate.html)