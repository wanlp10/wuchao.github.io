---
layout: post
title: Java8 Optional
category : [Java]
tagline: "Supporting tagline"
tags : [Java]
---
{% include JB/setup %}
# Java8 Optional
---

<!--break-->

### 创建 Optional 对象
#### of()
为非 null 的值创建一个 Optional。

of 方法通过工厂方法创建 Optional 类。需要注意的是，创建对象时传入的参数不能为 null。如果传入参数为 null，则抛出 NullPointerException。
```
// 调用工厂方法创建 Optional 实例
Optional<String> name = Optional.of("Java8");

// 传入参数为 null，抛出 NullPointerException.
Optional<String> name = Optional.of(null);
```

#### ofNullable()
为指定的值创建一个 Optional，如果指定的值为 null，则返回一个空的 (empty)Optional。

ofNullable 与 of 方法相似，唯一的区别是可以接受参数为 null 的情况。示例如下：
```
// 下面创建了一个不包含任何值的 Optional 实例
Optional empty = Optional.ofNullable(null);
```

### 常用 API
#### isPresent()
如果存在值就返回 true，否则返回 false。

#### get()
如果 Optional 中有值则将其返回，否则抛出 NoSuchElementException。

#### ifPresent(Consumer<? super T> consumer)
如果 Optional 实例有值则调用 consumer，否则不做处理。
```
name.ifPresent((value) -> {
  System.out.println("The length of the value is: " + value.length());
});
```

#### orElse(T default)
如果有值则将其返回，否则返回指定的默认值。
```
name.getElse("default");
```

#### orElseGet(Supplier<? extends T> other)
orElseGet 与 orElse 方法类似，区别在于得到的默认值。orElse 方法将传入的对象作为默认值，orElseGet 方法可以接受 Supplier 接口的实现用来生成默认值。
```
name.orElseGet(()-> "default value")
```

#### orElseThrow(Supplier<? extends X> exceptionSupplier)
如果有值则将其返回，否则抛出 Supplier r接口创建的异常。
```
try {
  empty.orElseThrow(()->new RuntimeException());
} catch (Throwable ex) {
  // 输出: No value present in the Optional instance
  System.out.println(ex.getMessage());
}
```

### map 函数
当 user.isPresent() 为真, 获得它关联的 orders , 为假则返回一个空集合时, 我们用上面的 orElse , orElseGet 方法都乏力时, 那原本就是 map 函数的责任, 我们可以这样一行：
```
return user.map(u -> u.getOrders()).orElse(Collections.emptyList());

//上面避免了我们类似 Java 8 之前的做法
if(user.isPresent()) {
  return user.get().getOrders();
} else {
  return Collections.emptyList();
}
```

map 是可能无限级联的, 比如再深一层, 获得用户名的大写形式：
```
return user.map(u -> u.getUsername())
           .map(name -> name.toUpperCase())
           .orElse(null);
```

这要搁在以前, 每一级调用的展开都需要放一个 null 值的判断：
```
User user = .....
if(user != null) {
  String name = user.getUsername();
  if(name != null) {
    return name.toUpperCase();
  } else {
    return null;
  }
} else {
  return null;
}
```

针对这方面 Groovy 提供了一种安全的属性/方法访问操作符 `?.`：
```
user?.getUsername()?.toUpperCase();
```

> 参考：
>
> [Java8新特性7--使用Optional解决空指针问题](https://www.jianshu.com/p/00fa8597d0c7)
>
> [【Java】jdk8 Optional 的正确姿势](https://blog.csdn.net/hj7jay/article/details/52459334)
