---
layout: post
title: Java8 Comparator
category : [Java]
tagline: "Supporting tagline"
tags : [Java, Comparator]
---
{% include JB/setup %}
# Java8 Comparator
---

一般来说，Comparable 是为了对某个类的集合进行排序，所以此时一般都是这个需要排序的类本身去实现 Comparable 接口。换句话说，如果某个类实现了 Comparable 接口，那么这个类的数组或者说 List 就可以进行排序了。很多时候我们无法对类进行修改，或者说此类修改的成本太高，但是又希望对其进行排序。那怎么办？这个时候 Comparator 接口就排上了用场。

<!--break-->

例如：
```
public final class Employee  {

    private String name;
    private int salary;

    public Employee(String name, int salary) {
        this.name = name;
        this.salary = salary;
    }

    @Override
    public String toString() {
        return "name is: " + name + ", salary is: " + salary;
    }

    // ...此处省略 get/set
}
```
Employee 类没有实现 Comparable 接口，并且加上了 final 关键字，但是还是需要对其排序，怎么办？

如果在 jdk8 之前，使用匿名内部类的方式：
```
public static void test() {
    List<Employee> list = genList();
    Collections.sort(list, new Comparator<Employee>() {
        @Override
        public int compare(Employee o1, Employee o2) {
            return o1.getSalary() - o2.getSalary();
        }
    });
    System.out.println(list);
}
```

在 jdk8 之后，可以使用 lambda 表达式：
```
public static void test() {
    List<Employee> list = genList();
    Comparator<Employee> comparator = (Employee e1, Employee e2) -> e1.getSalary() - e2.getSalary();
    list.sort(comparator);
    System.out.println(list);
}
```

如果将此方法 run 起来，输出如下：
```
[name is: ccc, salary is: 80, name is: aaa, salary is: 100, name is: bbb, salary is: 150]
```

Comparable 接口中只有一个 compareTo 方法要实现，而 Comparator 有两个方法，但是我们只实现了一个方法，那么另外一个方法呢？
其实很简单，因为另外一个方法是 equals 方法。所有的类都继承了 Object 类，而 Object 类中实现了 equals方法，所以我们这里不实现 equals 方法也无所谓。

### Comparator 中的各种实现方式比较
#### 1. 传统的匿名内部类
```
Collections.sort(list, new Comparator<Employee>() {
    @Override
    public int compare(Employee o1, Employee o2) {
        return o1.getSalary() - o2.getSalary();
    }
});
```

#### 2. lambda 表达式
```
list.sort((Employee e1, Employee e2) -> e1.getSalary() - e2.getSalary());
```

##### 2.1 精简版的 lambda 表达式：
```
list.sort((e1, e2) -> e1.getSalary() - e2.getSalary());
```

##### 2.2 使用 Comparator.comparing 的方式：
我们使用上述lambda表达式的时候，IDE 会提示我们：`can be replaced with comparator.comparing Int`。
```
list.sort(Comparator.comparing(employee -> employee.getSalary()));
```

##### 2.3 使用静态方法的引用
`::` 是 JDK8 里引入 lambda 后的一种用法，表示方法引用，比如静态方法的引用 String::valueOf，比如构造器的引用 ArrayList::new。
```
list.sort(Comparator.comparing(Employee::getSalary));
```

##### 2.4 排序反转（逆序）
```
list.sort(Comparator.comparing(Employee::getSalary).reversed());
```

### 多条件组合排序
#### 1. 传统的匿名内部类
```
list.sort((e1, e2) -> {
    if(e1.getSalary() != e2.getSalary()) {
        return e1.getSalary() - e2.getSalary();
    } else {
        return e1.getName().compareTo(e2.getName());
    }
});
```

#### 2. lambda 表达式

从 JDK8 开始，我们现在可以把多个 Comparator 链在一起（chain together）去建造更复杂的比较逻辑。
```
list.sort(Comparator.comparing(Employee::getSalary).thenComparing(Employee::getName));
```

> [Comparable 与 Comparator 比较](https://blog.csdn.net/bitcarmanlee/article/details/73381705)
> 
> [Java8：Lambda表达式增强版Comparator和排序](http://www.importnew.com/15259.html)
> 
> [java comparator 升序、降序、倒序从源码角度理解](https://blog.csdn.net/u013066244/article/details/78997869)
