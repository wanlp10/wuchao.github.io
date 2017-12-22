---
layout: post
title: Java Collection
category : [Java]
tagline: "Supporting tagline"
tags : [Java, List, Set, Map, Iterator]
---
{% include JB/setup %}
# Java Collection
--- 

<!--break-->

集合遍历时，如果要删除元素，唯一正确的方法时先将集合转换成迭代器，再使用iterator.remove() 方法，如：

```
Iterator<Integer> it = list.iterator();
while (it.hasNext()) {
    it.remove();
}
```

Java 不允许集合遍历时删除元素，因为集合遍历底层是使用 Iterator,Iterator 是工作在一个独立的线程中，并且拥有一个互斥锁。Iterator 被创建之后会建立一个指向原来对象的单链索引表，当原来的对象数量发生变化时，这个索引表的内容不会同步改变，所以当索引指针往后移动的时候就找不到要迭代的对象，所以按照 fail-fast 原则 Iterator 会马上抛出 java.util.ConcurrentModificationException 异常。所以 Iterator 在工作的时候是不允许被迭代的对象被改变的。

> [关于 List 比较好玩的操作](http://blog.csdn.net/ghsau/article/details/9347357)

## List 

### ArrayList 


### LinkList 


## Map 
### HashMap
对于 HashMap 而言，系统 key-value 当成一个Entry对象进行处理。它采用 Hash 算法来决定集合中元素的存储位置。当系统开始初始化 HashMap 时，系统会创建一个长度为 capacity 的 Entry 数组，这个数组里可以存储元素的位置被称为 “桶（bucket）”，每个 bucket 都有其指定索引，系统可以根据其索引快速访问该 bucket 里存储的元素。
无论何时，HashMap 的每个 “桶” 只存储一个元素（也就是一个 Entry），由于 Entry 对象可以包含一个引用变量（就是 Entry 构造器的的最后一个参数）用于指向下一个 Entry，因此可能出现的情况是：HashMap 的 bucket 中只有一个 Entry，但这个 Entry 指向另一个 Entry ——这就形成了一个 Entry 链。
当程序试图将一个 key-value 对放入 HashMap 中时，程序首先根据该 key 的 hashCode() 返回值决定该 Entry 的存储位置：如果两个 Entry 的 key 的 hashCode() 返回值相同，那它们的存储位置相同。如果这两个 Entry 的 key 通过 equals 比较返回 true，新添加 Entry 的 value 将覆盖集合中原有 Entry 的 value，但 key 不会覆盖。如果这两个 Entry 的 key 通过 equals 比较返回 false，新添加的 Entry 将与集合中原有 Entry 形成 Entry 链，而且新添加的 Entry 位于 Entry 链的头部。


### LinkedHashMap 
> [Java集合之LinkedHashMap](https://www.cnblogs.com/xiaoxi/p/6170590.html) 


## Set 
### HashSet
对于 HashSet 而言，它是基于 HashMap 实现的。它只是封装了一个 HashMap 对象来存储所有的集合元素，所有放入 HashSet 中的集合元素实际上由 HashMap 的 key 来保存，而 HashMap 的 value 则存储了一个 PRESENT，它是一个静态的 Object 对象。
HashSet 的绝大部分方法都是通过调用 HashMap 的方法来实现的，因此 HashSet 和 HashMap 两个集合在实现本质上是相同的。
所以 HashSet 在查找元素时需要调用到其集合元素的 hashcode() 和 equals() 方法。

> HashMap 与 HashSet 摘自：https://www.ibm.com/developerworks/cn/java/j-lo-hash/index.html

