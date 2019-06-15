---
layout: post
title:  "Collection(Set)"
date:   2014-05-19 00:02:01 +0800
categories: HashSet TreeSet
comments: true
---


Java set源码笔记。

## Set

```java
/*Set继承于Collection，是一个不予许有重复元素的集合，定义了集合的基础方法*/
public interface Set<E> extends Collection<E> {
    //...
}
```

## AbstractSet

```java
/*AbstractSet继承于AbstractCollection，实现了Set接口*/
public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {
    //...
}
```

## HashSet

```java
/*HashSet继承于AbstractSet，实现了Set接口、Cloneable（克隆）、Serializable（序列化）。内部是通过HashMap的Key来实现的，非线程安全*/
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    private transient HashMap<E,Object> map;//transient在Collection - List中讲过，需要通过writeObject&readObject来序列化

    private static final Object PRESENT = new Object();
    /*HashSet是通过HashMap的Key结构实现的*/
    public HashSet() {
        map = new HashMap<>();
    }
    
    public boolean add(E e) {
    	/*HashSet的add方法是将元素作为map的key，以new Object()为value，如果已经有此元素返回false*/
        return map.put(e, PRESENT)==null;
    }
    
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out any hidden serialization magic
        s.defaultWriteObject();

        // Write out HashMap capacity and load factor
        s.writeInt(map.capacity());
        s.writeFloat(map.loadFactor());

        // Write out size
        s.writeInt(map.size());

        // Write out all elements in the proper order.
        for (E e : map.keySet())
            s.writeObject(e);
    }

    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in any hidden serialization magic
        s.defaultReadObject();

        // Read capacity and verify non-negative.
        int capacity = s.readInt();
        if (capacity < 0) {
            throw new InvalidObjectException("Illegal capacity: " +
                                             capacity);
        }

        // Read load factor and verify positive and non NaN.
        float loadFactor = s.readFloat();
        if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
            throw new InvalidObjectException("Illegal load factor: " +
                                             loadFactor);
        }

        // Read size and verify non-negative.
        int size = s.readInt();
        if (size < 0) {
            throw new InvalidObjectException("Illegal size: " +
                                             size);
        }
        capacity = (int) Math.min(size * Math.min(1 / loadFactor, 4.0f),
                HashMap.MAXIMUM_CAPACITY);

        SharedSecrets.getJavaOISAccess()
                     .checkArray(s, Map.Entry[].class, HashMap.tableSizeFor(capacity));

        // Create backing HashMap
        map = (((HashSet<?>)this) instanceof LinkedHashSet ?
               new LinkedHashMap<E,Object>(capacity, loadFactor) :
               new HashMap<E,Object>(capacity, loadFactor));

        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            @SuppressWarnings("unchecked")
                E e = (E) s.readObject();
            map.put(e, PRESENT);
        }
    }
 //...
}
```

## TreeSet

```java
/*TreeSet继承于AbstractSet，实现了NavigableSet。内部默认通过TreeMap的key实现。非线程安全*/
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{
    private transient NavigableMap<E,Object> m;//transient在Collection - List中讲过，需要通过writeObject&readObject来序列化
    
    private static final Object PRESENT = new Object();
    
    public TreeSet() {
    	/*默认使用TreeMap实现*/
        this(new TreeMap<E,Object>());
    }
    
    public boolean add(E e) {
        return m.put(e, PRESENT)==null;
    }
    
    private void writeObject(java.io.ObjectOutputStream s)
    	throws java.io.IOException {
    	//...
    }
    
    private void readObject(java.io.ObjectInputStream s)
    	throws java.io.IOException, ClassNotFoundException {
    	//...
    }
}
```

## 总结

	学习Set最好先学习Map。



以上源码均来自JDK1.8.0_171 