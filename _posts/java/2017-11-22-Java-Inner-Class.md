---
layout: post
title: Java inner class
category : [Java]
tagline: "Supporting tagline"
tags : [Java, 内部类]
---
{% include JB/setup %}
# Java anonymous inner class
--- 

### 内部类
> [Java内部类详解](https://www.cnblogs.com/dolphin0520/p/3811445.html) 
>
> [Java内部类(成员内部类、静态内部类、局部内部类、匿名内部类)小结](http://blog.csdn.net/cws1214/article/details/52055980)
> 
> [四种java内部类总结](http://lvwenwen.iteye.com/blog/1906683)

创建内部类的典型的方式是在一个方法体的里面创建，局部内部类不能有访问说明符，因为它不是外部类的一部分，但是它可以访问当前代码块内的常量, 以及此外部类的所有成员。使用内部类的优点是内部类可以随意使用外部类的成员变量（包括私有）而不用生成外部类的对象。

<!--break-->

### 匿名内部类 
匿名内部类特点： 

1：匿名内部类没有构造方法。      
它无法被继续引用以生成实例，因而不需要构造方法。 
在生成匿名内部类的时候，与其一个对应的实例随即产生。

2：一个匿名内部类一定是在 new 的后面，用其隐含实现一个接口或实现一个类。 

3：匿名内部类不可以是public,protected,private,包权限或static的。
             
4：匿名内部类不能定义任何静态的成员、方法和类。  
参考： [浅谈Java的匿名类](https://www.cnblogs.com/caipc/p/5930236.html)     

5：内部类只能访问外部类的 final 变量。   
参考： [java 为什么匿名内部类和局部内部类只能访问final变量](http://blog.csdn.net/qq_16121273/article/details/51120520)