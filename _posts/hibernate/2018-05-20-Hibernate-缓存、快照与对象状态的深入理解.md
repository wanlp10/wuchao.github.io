---
layout: post
title: Hibernate 缓存、快照与对象状态的深入理解
category : [Hibernate]
tagline: "Supporting tagline"
tags : [Hibernate]
---
{% include JB/setup %}
# Hibernate 缓存、快照与对象状态的深入理解

从几种现象，理解缓存与快照的运行机制

<!--break--> 

//@1 缓存与快照机制

hibernate通过缓存和快照机制，实现对修改内容批量提交。 
当查询DB时，会将数据保存到session缓存中，同时在内存中存储一份快照副本。 
当我修改数据时，其实修改的是session缓存中的实体内容，并不立即提交DB执行。 
当我们主动flush或提交事物时，会对比session缓存与快照中的内容是否一致，将不一致的内容更新到DB中。

//@2 思考与疑惑 
以上是对缓存流程的理解，但是通过实验，发现以下几种情况，无法很好的解释：

//@ 2-1 
1）
``` 
Stut s = new Stut();
s.setId("1");
s.setClazz("Oralce1601");
s.setName("123");
se.save(s);//执行新增

s.setName("456");//执行修改

se.getTransaction().commit();//事物提交
``` 
有实体类Stut，对应数据表stut，该实体类有三个简单的属性：id,clazz,name，其中id映射表的主键id。 
这里我们新增了一条Stus记录 s，并save到缓存中；接下来我们修改这条记录；最后commit提交。 
按照上面的逻辑进行分析，新增的记录s，应该只存在于session缓存中，快照中并不存在该条记录，接下来我们修改，也是修改缓存中的该记录。当我们进行提交时，会对比缓存与快照的内容，会发现session中的s是新增的记录，快照中并没有，会执行insert语句，将数据插入到DB中。 
以上是我们的分析过程，但是我们看hibernate打印的日志: 
![](/images/2018-05-20-Hibernate-Cache-Snapshot-Status-Deep-Understanding-01.png)

不仅执行了insert，还执行了update，这就奇怪了，为什么会执行修改呢？不应该直接insert就是修改后的数据吗？？（备注，多次修改，也只有一条update）

//@ 2-2 
2）
``` 
Session se = HiberHelper.getSession();
se.beginTransaction();

Stut s = se.get(Stut.class, "1");

se.delete(s);
System.out.println(se.contains(s));//false

s.setName("888");
//此处，打印日志1；还没到事务提交，就执行了delete语句，但是数据库没有改变
se.save(s);

System.out.println(se.contains(s));//true

//此处，打印日志2，执行了insert语句，提交数据库
se.getTransaction().commit();
``` 
首先读取数据库中id等于1的学生对象s，该对象会被存储在session缓存中，同时也在快照内存中备份。 
接下来我们将它删除，根据之前的分析，此时只会在session缓存删除对象，快照中依然存在该对象。 
接着，设置对象新的name值“888”，重新save对象。 
最后提交事务。 
问题就出现在重新save对象时，此时终端居然打印出了delete的sql语句： 
![](/images/2018-05-20-Hibernate-Cache-Snapshot-Status-Deep-Understanding-02.png)
（红框框起来的部分，之后再分析）

最后提交事务的时候，打印sql日志： 
![](/images/2018-05-20-Hibernate-Cache-Snapshot-Status-Deep-Understanding-03.png)
整个流程理顺下来，就有疑问了： 
1）不是说hibernate在事务提交时统一执行sql语句吗，为什么在我们save对象时，会执行delete的sql语句呢？ 
2）不是说hibernate的sql执行顺序是，insert，update，delete吗，为什么最后只有一个insert？

//@3 解答

后来经过反复思考，还是终于有了答案，其实还是对Hibernate三种对象状态的理解不深刻引起的。 
三种对象状态如下： 
1) 瞬时态：在数据库和session缓存中，都不存在该对象id的记录。我们通过new关键字新建的对象，就是处于该种状态。

2）持久态：在数据库和session缓存中，都有该对象id的记录；我们可以通过save方法，将瞬时态的对象变为持久态。

3）托管状态：托管，就是脱离管理，准确的说是脱离了session的管理，所以处于该状态的对象，在数据库中是有记录的，但是在session缓存中没有。

三种状态之前的切换图（摘自网络）：
![](/images/2018-05-20-Hibernate-Cache-Snapshot-Status-Deep-Understanding-04.png)
从图中，我们可以看出各个状态直接是如何通过方法的调用进行切换的。

第二部分两个问题的出现，就是对这个图的理解不够深刻。由于之前程序运行结果和图中的描述“不一致”，导致一直对这个图不够重视，或者说不能真正的理解，不一致如下： 
1）Stu s = new Stu(); 新创建了对象s，对象处于瞬时状态。（之后设置s的id和其他属性） 
2）session.save(s); 由图中的描述可知，通过save方法，将s由瞬时态转化为持久态。也就是说，数据库和session中均存在该对象id。 
问题就出在这一步，当调用save方法时，查看hibernate打印的日志，并没有发现执行了任何的insert语句。也就是说，s并没有真正的插入到数据库中，也就是只有缓存中有，这就是描述与测试结果不一致的地方。 
同时，当我们执行delete方法，将对象由持久变为瞬时态时，也并没有看到任何delete sql语句的执行，也就是说只是缓存中没有了，数据库中依然有该记录，直到事务提交才打印出sql。 
基于以上的两点问题，导致我对状态的转换一直混沌，不能真正理解它。

那这怎么解释呢？程序的结果和理论的描述，为什么会不统一呢？ 
经过了好久的思考，终于有了一个合理的解释，其实程序运行的也没错，理论描述的也没错。只是在hibernate实现这个状态转换时，基于“快照”进行了流程优化。

首先，从瞬时态，到持久态，当我们调用save方法时，理论上要将这条数据通过执行insert sql语句，实实在在的插入到数据库表中，但是考虑到每次与数据库的“沟通”都会影响效率，hibernate此时做了一个“巧妙”的处理，他将本应该保存到数据库和session中的数据，保存到了快照和session中，相当于用快照来充当数据库（因为快照就是从数据库读取信息的副本啊），同时将这条本该马上执行的insert语句保存起来，等事务提交时一起执行。 
这样，即提升了运行效率，又造成了一种数据保存到数据库中的假象。

那么，我们使用这种方法解释delete，就容易多了，当delete时，本该将session缓存与数据库中的记录都删除，但是为了减少与数据库的交互次数，此时将删除session缓存与快照中的记录，因为快照就是数据库内容的副本啊，同时暂存一条delete语句，等事务提交时一起执行。

理解了这两点，那对状态切换图就有了新的认识，不再疑惑程序运行结果的这些所谓的“不一致”。 
并且突然发现，第二部分的那两个问题也有了很好的解释： 
首先，第一个问题，“为什么会出现insert和update两条语句呢？”，因为当我们save对象时，由于hibernate实现流程的优化，此处会占时保存一条insert语句，同时快照和session中同时存在该对象，当我们修改数据并提交时，hibernate比对session中修改后的对象与快照中的原对象值不一致了，此时会产生一条update语句。所以，最终执行的sql语句是，先执行insert，再执行update。

接着，第二个问题，“为什么会在save调用时，执行delete语句呢？”，当delete方法调用时，删除缓存与快照中的对象，同时暂存一条delete的sql语句，当我们再次保存对象时，将对象同时保存在缓存与快照中，同时暂存一条save语句，这时候就存储了两条待执行的语句，一条是save，一条是delete，并且这两个操作是针对同一个对象id，如果此时不执行delete 的sql，那么当事务提交时，sql语句会按照insert，update，delete的顺序批量执行，那么重新保存的对象会被delete语句删除，而出现逻辑错误。所以，当我们对同一个对象re-save时，之前的delete语句会取出来flush到数据库执行。所以，当我们执行save时，会打印出日志： 
``` 
19:31:00,898 DEBUG SessionImpl:1455 - Flushing to force deletion of re-saved object: [model.Stut#1]
```
“ force deletion of re-saved object ” 


> 参考： 
> 
> [Hibernate 缓存、快照与对象状态的深入理解](https://blog.csdn.net/boboma_dut/article/details/79659061) 
