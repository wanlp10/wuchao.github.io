---
layout: post
title: Java 类的 hashCode 方法和 equals 方法
category : [Java]
tagline: "Supporting tagline"
tags : [hashCode, equals]
---
{% include JB/setup %}
# Java 类的 hashCode 方法和 equals 方法
--- 

<!--break-->  

The contract between equals() and hashCode() is:
1) If two objects are equal, then they must have the same hash code.
2) If two objects have the same hash code, they may or may not be equal. 

如果两个对象相同，则它们必须要有相同的 hash code，所以重写 equals 方法时，一定也要重写 hashCode 方法。
否则会出现不可预见的错误，例如下面的代码：

``` 
import java.util.HashMap;
 
public class Apple {
	private String color;
 
	public Apple(String color) {
		this.color = color;
	}
 
	public boolean equals(Object obj) {
		if(obj==null) return false;
		if (!(obj instanceof Apple))
			return false;	
		if (obj == this)
			return true;
		return this.color.equals(((Apple) obj).color);
	}
 
	public static void main(String[] args) {
		Apple a1 = new Apple("green");
		Apple a2 = new Apple("red");
 
		//hashMap stores apple type and its quantity
		HashMap<Apple, Integer> m = new HashMap<Apple, Integer>();
		m.put(a1, 10);
		m.put(a2, 20);
		System.out.println(m.get(new Apple("green")));
	}
}
``` 

打印结果输出 null。

