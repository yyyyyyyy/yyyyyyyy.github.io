---
layout: post
title:  "【源码】Collection& AbstractCollection"
date:   2016-05-19 00:00:00 +0800
categories: 
comments: true
---

### java.util.Collection
```java
package java.util;

import java.util.function.Predicate;
import java.util.stream.Stream;
import java.util.stream.StreamSupport;



public interface Collection<E> extends Iterable<E> {
    // Query Operations
    int size();//集合的长度
    boolean isEmpty();//集合是否为空
    boolean contains(Object o);//集合是否包含传入的元素
    Iterator<E> iterator();//集合对应的迭代器
    Object[] toArray();//集合转换为数组
    <T> T[] toArray(T[] a);//集合转换为数组（范型）

    // Modification Operations
    boolean add(E e);//在数组中添加此元素
    boolean remove(Object o);//在数组中移除此元素
    
    // Bulk Operations
    boolean containsAll(Collection<?> c);//集合属否包含指定集合的所有元素
    boolean addAll(Collection<? extends E> c);//集合添加指定集合的所有元素
    boolean removeAll(Collection<?> c);//集合移除指定集合的所有元素
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }

    boolean retainAll(Collection<?> c);//集合移除指定集合的元素外的所有元素
    void clear();//清空集合

    // Comparison and hashing
    boolean equals(Object o);
    int hashCode();
    
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }

    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }

    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
}

```

### java.util.AbstractCollection
```java

package java.util;

public abstract class AbstractCollection<E> implements Collection<E> {
    /**
     * Sole constructor.  (For invocation by subclass constructors, typically
     * implicit.)
     */
    protected AbstractCollection() {
    }

    // Query Operations

    public abstract Iterator<E> iterator();
    public abstract int size();

    //实现了是否为空的方法，通过集合的size是否为0来判断，若为0，返回true（空集合）。
    public boolean isEmpty() {
        return size() == 0;
    }

    //实现了是否包含指定元素方法，
    public boolean contains(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return true;//如果传入的元素是null元素，集合中有null则返回true
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;//通过equals方法来判断元素是否相同。
        }
        return false;
    }

    
    public Object[] toArray() {
        // Estimate size of array; be prepared to see more or fewer elements
        Object[] r = new Object[size()];
        Iterator<E> it = iterator();
        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) // fewer elements than expected
                return Arrays.copyOf(r, i);//当元素的数量比集合的size小时，返回元素数量大小的数组。
            r[i] = it.next();
        }
        //当元素和集合大小相同时，直接返回被填充的数组；否则进入finishToArray方法，见下面方法
        return it.hasNext() ? finishToArray(r, it) : r;
    }

    //与上面的方法一样，多加了范型。
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        // Estimate size of array; be prepared to see more or fewer elements
        int size = size();
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);
        Iterator<E> it = iterator();

        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) { // fewer elements than expected
                if (a == r) {
                    r[i] = null; // null-terminate
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

    //集合的最大长度，为什么减8呢？因为数组存储头信息最大不超过8字节。
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    @SuppressWarnings("unchecked")
    private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
        int i = r.length;
        while (it.hasNext()) {
            int cap = r.length;
            if (i == cap) {
                int newCap = cap + (cap >> 1) + 1;//扩容增加原容量的一半再加1
                // overflow-conscious code
                if (newCap - MAX_ARRAY_SIZE > 0)
                    newCap = hugeCapacity(cap + 1);
                r = Arrays.copyOf(r, newCap);
            }
            r[i++] = (T)it.next();//继续填充
        }
        // trim if overallocated
        return (i == r.length) ? r : Arrays.copyOf(r, i);
    }

    //返回符合当前大小的最小容量，超出则抛出OOM
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError
                ("Required array size too large");
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    // Modification Operations
    //由子类实现添加元素方法
    public boolean add(E e) {
        throw new UnsupportedOperationException();
    }

    //移除元素
    public boolean remove(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    it.remove();
                    return true;
                }
            }
        }
        return false;
    }


    // Bulk Operations

    public boolean containsAll(Collection<?> c) {
        for (Object e : c)
            if (!contains(e))
                return false;
        return true;
    }

    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<?> it = iterator();
        while (it.hasNext()) {
            if (c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            if (!c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    public void clear() {
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            it.next();
            it.remove();
        }
    }


    //  String conversion

    public String toString() {
        Iterator<E> it = iterator();
        if (! it.hasNext())
            return "[]";

        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (;;) {
            E e = it.next();
            sb.append(e == this ? "(this Collection)" : e);
            if (! it.hasNext())
                return sb.append(']').toString();
            sb.append(',').append(' ');
        }
    }

}

```


### 栗子
```java
package com.yaochow.demo;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collection;

public class CollectionTest {


    public static void main(String[] args) {
        Collection collection = new ArrayList();
        collection.add("1");
        collection.add(1);
        System.out.println(Arrays.toString(collection.toArray()));
        collection.removeIf(item -> item instanceof String);
        System.out.println(Arrays.toString(collection.toArray()));
        boolean b = collection.stream().allMatch(item -> item instanceof Integer);
        System.out.println(b);
        collection.add(1);
        collection.stream().distinct().forEach(item -> System.out.println("distinct: " + item));

    }
}

```
执行结果：
```
[1, 1]
[1]
true
distinct: 1
```

### 总结
1. Collection中定义了集合的基本功能
2. AbstractCollection实现了一些集合的基本方法
3. jdk1.8增加了一些新特性，比如stream、removeIf等。spliterator在【List源码】中有栗子。

源码中大部分都标了注释，有些特别简单的就没有加； jdk1.8新增的方法写在了栗子中。

>* 以上源码来自JDK1.8.0_171





