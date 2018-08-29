---
layout: post
title: Java Utility Class
category : [Java]
tagline: "Supporting tagline"
tags : [Java]
---
{% include JB/setup %}
# Java 工具类的定义
---

### XxxUtil.java 和 其他例如 java.lang.Math.java这样的类
1. 类定义为 final(不希望被继承).
2. 构造方法定义为 private 权限(不希望被实例化使用),并且在构造方法里抛出异常(防止通过反射调用).
3. 只定义静态(static)字段(希望通过 "类名.字段" 方式访问).
4. 方法全部定义为 static(希望通过 "类名.方法" 方式访问). 

<!--break-->

``` 
public final class Math {  
   /**  
    * Don't let anyone instantiate this class.  
    */  
   private Math() {}  
}  
```

### XxxUtils.java 
1. 类定义为 abstract.
2. 只定义静态(static)字段(希望通过 "类名.字段" 方式访问).
3. 方法全部定义为 static(希望通过 "类名.方法" 方式访问). 
``` 
public abstract class StringUtils {

	private static final String FOLDER_SEPARATOR = "/";

	private static final String WINDOWS_FOLDER_SEPARATOR = "\\";

	private static final String TOP_PATH = "..";

	private static final String CURRENT_PATH = ".";

	private static final char EXTENSION_SEPARATOR = '.';

	public static boolean isEmpty(Object str) {
		return (str == null || "".equals(str));
	}
	
    //...
}
```