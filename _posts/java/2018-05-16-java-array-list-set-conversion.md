---
layout: post
title: Java Array、List、Set 的互相转换
category : [Java]
tagline: "Supporting tagline"
tags : [Java]
---
{% include JB/setup %}
# Java Array、List、Set 的互相转换
---

<!--break-->

 ## Array、List、Set 互转实例
 ### Array、List 互转

 #### Array 转 List
 ```
 String[] s = new String[]{"A", "B", "C", "D", "E"};
 List<String> list = Arrays.asList(s);
 ```
注意这里 list 里面的元素直接是 s 里面的元素（list backed by the specified array），换句话就是说：对 s 的修改，直接影响 list。
```
s[0] = "AA";
System.out.println("list: " + list);
```
输出结果
```
list: [AA, B, C, D, E]
```

#### List 转 Array
```
String[] dest = list.toArray(new String[0]); // new String[0]是指定返回数组的类型
System.out.println("dest: " + Arrays.toString(dest));
```
输出结果
```
dest: [AA, B, C, D, E]
```
注意这里的 dest 里面的元素不是 list 里面的元素，换句话就是说：对 list 中关于元素的修改，不会影响 dest。

```
list.set(0, "Z");
System.out.println("modified list: " + list);
System.out.println("dest: " + Arrays.toString(dest));
```
输出结果
```
modified list: [Z, B, C, D, E]
dest: [AA, B, C, D, E]
```
可以看到 list 虽然被修改了，但是 dest 数组没有没修改。

### List、Set 互转   
因为 List 和 Set 都实现了 Collection 接口，且 addAll(Collection<? extends E> c) 方法，因此可以采用 addAll() 方法将 List 和 Set 互相转换；另外，List 和 Set 也提供了 Collection<? extends E> c 作为参数的构造函数，因此通常采用构造函数的形式完成互相转化。  
```
// List 转 Set
Set<String> set = new HashSet<>(list);
System.out.println("set: " + set);
// Set 转 List
List<String> list_1 = new ArrayList<>(set);
System.out.println("list_1: " + list_1);
```
和 toArray() 一样，被转换的 List / Set 的修改不会对被转化后的 Set / List 造成影响。

#### Array、Set 互转
由上面可完成 Array 和 Set 的互转。
```
// array 转 set
s = new String[]{"A", "B", "C", "D","E"};
set = new HashSet<>(Arrays.asList(s));
System.out.println("set: " + set);
// set 转 array
dest = set.toArray(new String[0]);
System.out.println("dest: " + Arrays.toString(dest));
```

## Arrays.asList() 和 Collection.toArray()
上述列出的互相转换离不开 Arrays.asList() 和 Collection.toArray() 两个重要的方法；
> This method acts as bridge between array-based and collection-based APIs, in combination with Collection.toArray. The returned list is serializable and implements RandomAccess.

### Arrays.asList()
```
@SafeVarargs
@SuppressWarnings("varargs")
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```
这里出现的 ArrayList<> 并不是我们通常使用的 java.util.ArrayList，因为 java.util.ArrayList 没有数组作为参数的构造函数。查看对应的源码发现，其实 Arrays 类的静态内部类。
```
/**
  * @serial include
  */
private static class ArrayList<E> extends AbstractList<E>
    implements RandomAccess, java.io.Serializable {

    private static final long serialVersionUID = -2764017481108945198L;
    private final E[] a;

    ArrayList(E[] array) {
        a = Objects.requireNonNull(array);
    }

    @Override
    public int size() {
        return a.length;
    }

    @Override
    public Object[] toArray() {
        return a.clone();
    }

    @Override
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        int size = size();
        if (a.length < size)
            return Arrays.copyOf(this.a, size,
                                 (Class<? extends T[]>) a.getClass());
        System.arraycopy(this.a, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    @Override
    public E get(int index) {
        return a[index];
    }

    @Override
    public E set(int index, E element) {
        E oldValue = a[index];
        a[index] = element;
        return oldValue;
    }

    @Override
    public int indexOf(Object o) {
        E[] a = this.a;
        if (o == null) {
            for (int i = 0; i < a.length; i++)
                if (a[i] == null)
                    return i;
        } else {
            for (int i = 0; i < a.length; i++)
                if (o.equals(a[i]))
                    return i;
        }
        return -1;
    }

    @Override
    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }

    @Override
    public Spliterator<E> spliterator() {
        return Spliterators.spliterator(a, Spliterator.ORDERED);
    }

    @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        for (E e : a) {
            action.accept(e);
        }
    }

    @Override
    public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        E[] a = this.a;
        for (int i = 0; i < a.length; i++) {
            a[i] = operator.apply(a[i]);
        }
    }

    @Override
    public void sort(Comparator<? super E> c) {
        Arrays.sort(a, c);
    }
}
```

可以看到，这个由 Arrays 类实现的另一个 Arrays$ArrayList，对于 java.util.ArrayList 类来讲，是比较简单粗糙的类。

- 没有扩容机制；

- 无法在指定位置 add(int index, E element)，调用该方法会抛异常；这些不同让这个 ArrayList 看起来实际上就是一个 List-View 的数组。

### Collection.toArray()
虽然 List、Set 的具体实现类都对 Collection.toArray() 方法进行了不同程度的重写，但是大致都差不多。

这里选 AbstractCollection.toArray() 的实现：
```
public <T> T[] toArray(T[] a) {
    // Estimate size of array; be prepared to see more or fewer elements
    int size = size();
    T[] r = a.length >= size ? a :
              (T[])java.lang.reflect.Array
              .newInstance(a.getClass().getComponentType(), size);//如果给定的参数T[] a的长度足够存放当前collection(list or set)的元素，则采用该参数来存放元素；否则则根据参数给定的类型反射生成一个数组；
//因此这里的参数T[] a有俩作用；第一：可能用作存放元素；第二：为返回数组提供类型
    Iterator<E> it = iterator();
    for (int i = 0; i < r.length; i++) {
        if (! it.hasNext()) { // fewer elements than expected 集合的size少于给定的参数数组的长度
            if (a == r) {
                r[i] = null; // null-terminate 最后一个元素被设置为null，表明collection元素结束；
            } else if (a.length < i) {
                return Arrays.copyOf(r, i);
            } else {
                System.arraycopy(r, 0, a, 0, i);
                if (a.length > i) {
                    a[i] = null;
                }
            }
            return a;
        }
        r[i] = (T)it.next();
    }
    // more elements than expected
    return it.hasNext() ? finishToArray(r, it) : r;
}
```

> 参考：[Java Array、List、Set互相转化](https://blog.csdn.net/u014532901/article/details/78820124)
