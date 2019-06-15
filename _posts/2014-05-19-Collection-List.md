---
layout: post
title:  "Collection(List)"
date:   2014-05-19 00:02:00 +0800
categories: ArrayList LinkedList Vector
---

Java List源码笔记。


## Collection
```java
/* Collection继承了Iterable接口。Collection类中抽象定义了集合的基本操作。*/
public interface Collection<E> extends Iterable<E> {
    //Collection源码比较简单，可以自行观看。
}
```

## Iterator
```java
/* Iterator是集合的迭代器，集合可以通过Iterator去遍历集合中的元素。*/
public interface Iterable<T> {
    //...
}
```

## ListIterator
```java
/* 队列迭代器。*/
public interface ListIterator<E> extends Iterator<E> {
    //...
}
```

## AbstractCollection
```java
/* AbstractCollection是一个抽象类，实现了Collection的大部分接口。通过继承AbstractCollection即实现了大部分集合类接口。*/
public abstract class AbstractCollection<E> implements Collection<E> {
    //...
}
```

## List
```java
/* List继承自Collection，是集合中的一类--有序队列。List中每个元素都对应着一个索引，允许有重复元素。
  相比Collection增加了一些与索引相关的抽象方法定义。*/
public interface List<E> extends Collection<E>{
    //...
}
```

## AbstractList
```java
/* AbstractList继承了AbstractCollection抽象类，实现了List接口。主要是实现了List接口中索引相关方法。*/
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    //..
}
```

### ArrayList
```java
/* ArrayList继承了AbstractList抽象了，实现了List（集合）、RandomAccess（随机访问）、Cloneable（克隆）、Serializable（序列化）等接口。
* 非线程安全。
* */
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
    /*
    * ArrayList是通过Object[]来实现的。
    * transient修饰的属性序列化需要通过特殊的方式，在这里不细说了，用下面的readObject&writeObject。
    * */
    transient Object[] elementData;
    
    private static final int DEFAULT_CAPACITY = 10;//默认容量为10。
    
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);//每次新增元素都会检查是否超出容量
        elementData[size++] = e;
        return true; 
    }
        
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);//容量每次增加一半
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    
    private void writeObject(java.io.ObjectOutputStream s)
            throws java.io.IOException{
        int expectedModCount = modCount;
        s.defaultWriteObject();
        s.writeInt(size);
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
    
    private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;
        s.defaultReadObject();
        s.readInt(); // ignored
        if (size > 0) {
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            ensureCapacityInternal(size);
            Object[] a = elementData;
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
    
    /*ArrayList的subList方法只是获取了ArrayList的视图，对subList的操作也会影响到ArrayList。*/
    private class SubList extends AbstractList<E> implements RandomAccess {
        public E get(int index) {
            rangeCheck(index);
            checkForComodification();
            return ArrayList.this.elementData(offset + index);
        }
    }
    //...
}
```

### LinkedList
```java
/*
* 非线程安全。
* LinkedList中方法较多，可以自己查看各个方法的实现具体有什么不同。
* 比如poll（unlinkFirst）、pop(removeFirst)，若first为null，pop会抛异常。
* */
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient Node<E> first;//transient在上面ArrayList说过了。
    
    transient Node<E> last;
    
    /*LinkedList是通过双向链表来实现的，Node是LinkedList的一个静态内部类*/
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
    
    Node<E> node(int index) {
        // LinkedList的get方法使用了2分查找
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
}
```

### Vector
```java
/* 线程安全，与ArrayList相比，大部分方法都加入了synchronized*/
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    protected Object[] elementData;//与ArrayList一样也是通过Object[]实现的，主要属性没有被trasient修饰。
    
    protected int capacityIncrement;//扩容时的步长
    
    public Vector() {
        this(10);//默认容量是10
    }
    
    private void grow(int minCapacity) {
        //Vector在扩容时，若定义的步长则增加步长，若没有定义步长则扩容为原来的2倍。
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
}
```

## 总结
    ArrayList 是一个数组队列，相当于动态数组。它由数组实现，随机访问效率高，随机插入、随机删除效率低，非线程安全。
    LinkedList 是一个双向链表。它也可以被当作堆栈、队列或双端队列进行操作。LinkedList随机访问效率低，但随机插入、随机删除效率高，非线程安全。
    Vector 是矢量队列，和ArrayList一样，它也是一个动态数组，由数组实现，不过是线程安全的。

以上源码均来自JDK1.8.0_171