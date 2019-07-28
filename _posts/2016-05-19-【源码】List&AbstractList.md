---
layout: post
title:  "【源码】List& AbstractList"
date:   2016-05-19 00:01:00 +0800
categories: 
comments: true

---

### java.util.List

```java
package java.util;

import java.util.function.UnaryOperator;



public interface List<E> extends Collection<E> {
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

    // Bulk Modification Operations批量修改操作
    boolean containsAll(Collection<?> c);//集合属否包含指定集合的所有元素
    boolean addAll(Collection<? extends E> c);//集合添加指定集合的所有元素
    boolean addAll(int index, Collection<? extends E> c);//集合添加指定集合的所有元素，从下标index开始
    boolean removeAll(Collection<?> c);//集合移除指定集合的所有元素
    boolean retainAll(Collection<?> c);//集合移除指定集合的元素外的所有元素
    //这个看下面的栗子
    default void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final ListIterator<E> li = this.listIterator();
        while (li.hasNext()) {
            li.set(operator.apply(li.next()));
        }
    }

    //排序
    @SuppressWarnings({"unchecked", "rawtypes"})
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }

    void clear();//清空集合


    // Comparison and hashing
    boolean equals(Object o);
    int hashCode();

    // Positional Access Operations
    E get(int index);//获取指定下标index对应的元素
    E set(int index, E element);//在指定下标index位置上插入指定元素element，并返回一个元素值
    void add(int index, E element);//在指定下标index位置上插入指定元素element
    E remove(int index);//移除指定下标index对应的元素


    // Search Operations
    int indexOf(Object o);//获取指定元素的下标
    int lastIndexOf(Object o);//获取指定元素最后一个下标


    // List Iterators
    ListIterator<E> listIterator();
    ListIterator<E> listIterator(int index);

    // View
    List<E> subList(int fromIndex, int toIndex);

    //看下面的栗子
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.ORDERED);
    }
}

```

### java.util.AbstractList

```java
package java.util;

public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    /**
     * Sole constructor.  (For invocation by subclass constructors, typically
     * implicit.)
     */
    protected AbstractList() {
    }

    public boolean add(E e) {
        add(size(), e);//size()正好是集合最后一个下标+1，在尾部添加新元素。
        return true;
    }

    abstract public E get(int index);//子类实现

    public E set(int index, E element) {//子类实现
        throw new UnsupportedOperationException();
    }

    public void add(int index, E element) {//子类实现
        throw new UnsupportedOperationException();
    }

    public E remove(int index) {//子类实现
        throw new UnsupportedOperationException();
    }


    // Search Operations

  	//返回集合中等于元素o的第一个下标
    public int indexOf(Object o) {
        ListIterator<E> it = listIterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return it.previousIndex();
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return it.previousIndex();
        }
        return -1;
    }

 		//返回集合中等于元素o的最后一个下标
    public int lastIndexOf(Object o) {
        ListIterator<E> it = listIterator(size());
        if (o==null) {
            while (it.hasPrevious())
                if (it.previous()==null)
                    return it.nextIndex();
        } else {
            while (it.hasPrevious())
                if (o.equals(it.previous()))
                    return it.nextIndex();
        }
        return -1;
    }


    // Bulk Operations

    public void clear() {
        removeRange(0, size());
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        boolean modified = false;
        for (E e : c) {
            add(index++, e);//从index下标开始添加集合元素
            modified = true;
        }
        return modified;
    }


    // Iterators

    public Iterator<E> iterator() {
        return new Itr();//返回Itr迭代器
    }

    public ListIterator<E> listIterator() {
        return listIterator(0);
    }

    public ListIterator<E> listIterator(final int index) {
        rangeCheckForAdd(index);

        return new ListItr(index);//返回ListItr迭代器
    }

    private class Itr implements Iterator<E> {
        
        int cursor = 0;//指针

        /**
         * Index of element returned by most recent call to next or
         * previous.  Reset to -1 if this element is deleted by a call
         * to remove.
         */
        int lastRet = -1;

        /**
         * The modCount value that the iterator believes that the backing
         * List should have.  If this expectation is violated, the iterator
         * has detected concurrent modification.
         */
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size();
        }

        public E next() {
            checkForComodification();
            try {
                int i = cursor;
                E next = get(i);
                lastRet = i;
                cursor = i + 1;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            cursor = index;
        }

        public boolean hasPrevious() {
            return cursor != 0;
        }

        public E previous() {
            checkForComodification();
            try {
                int i = cursor - 1;
                E previous = get(i);
                lastRet = cursor = i;
                return previous;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor-1;
        }

        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.set(lastRet, e);
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                AbstractList.this.add(i, e);
                lastRet = -1;
                cursor = i + 1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }

    public List<E> subList(int fromIndex, int toIndex) {
        return (this instanceof RandomAccess ?
                new RandomAccessSubList<>(this, fromIndex, toIndex) :
                new SubList<>(this, fromIndex, toIndex));
    }

    // Comparison and hashing

    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof List))
            return false;

        ListIterator<E> e1 = listIterator();
        ListIterator<?> e2 = ((List<?>) o).listIterator();
        while (e1.hasNext() && e2.hasNext()) {
            E o1 = e1.next();
            Object o2 = e2.next();
            if (!(o1==null ? o2==null : o1.equals(o2)))
                return false;
        }
        return !(e1.hasNext() || e2.hasNext());
    }

    public int hashCode() {
        int hashCode = 1;
        for (E e : this)
            hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
        return hashCode;
    }

    protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
    }

    protected transient int modCount = 0;

    private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size())
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size();
    }
}

//视图
class SubList<E> extends AbstractList<E> {
    private final AbstractList<E> l;
    private final int offset;
    private int size;

    SubList(AbstractList<E> list, int fromIndex, int toIndex) {
        if (fromIndex < 0)
            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > list.size())
            throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        if (fromIndex > toIndex)
            throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                               ") > toIndex(" + toIndex + ")");
        l = list;//没有生成一个新集合，而是对原集合进行操作。
        offset = fromIndex;
        size = toIndex - fromIndex;
        this.modCount = l.modCount;
    }

    public E set(int index, E element) {
        rangeCheck(index);
        checkForComodification();
        return l.set(index+offset, element);
    }

    public E get(int index) {
        rangeCheck(index);
        checkForComodification();
        return l.get(index+offset);
    }

    public int size() {
        checkForComodification();
        return size;
    }

    public void add(int index, E element) {
        rangeCheckForAdd(index);
        checkForComodification();
        l.add(index+offset, element);
        this.modCount = l.modCount;
        size++;
    }

    public E remove(int index) {
        rangeCheck(index);
        checkForComodification();
        E result = l.remove(index+offset);
        this.modCount = l.modCount;
        size--;
        return result;
    }

    protected void removeRange(int fromIndex, int toIndex) {
        checkForComodification();
        l.removeRange(fromIndex+offset, toIndex+offset);
        this.modCount = l.modCount;
        size -= (toIndex-fromIndex);
    }

    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        int cSize = c.size();
        if (cSize==0)
            return false;

        checkForComodification();
        l.addAll(offset+index, c);
        this.modCount = l.modCount;
        size += cSize;
        return true;
    }

    public Iterator<E> iterator() {
        return listIterator();
    }

    public ListIterator<E> listIterator(final int index) {
        checkForComodification();
        rangeCheckForAdd(index);

        return new ListIterator<E>() {
            private final ListIterator<E> i = l.listIterator(index+offset);

            public boolean hasNext() {
                return nextIndex() < size;
            }

            public E next() {
                if (hasNext())
                    return i.next();
                else
                    throw new NoSuchElementException();
            }

            public boolean hasPrevious() {
                return previousIndex() >= 0;
            }

            public E previous() {
                if (hasPrevious())
                    return i.previous();
                else
                    throw new NoSuchElementException();
            }

            public int nextIndex() {
                return i.nextIndex() - offset;
            }

            public int previousIndex() {
                return i.previousIndex() - offset;
            }

            public void remove() {
                i.remove();
                SubList.this.modCount = l.modCount;
                size--;
            }

            public void set(E e) {
                i.set(e);
            }

            public void add(E e) {
                i.add(e);
                SubList.this.modCount = l.modCount;
                size++;
            }
        };
    }

    public List<E> subList(int fromIndex, int toIndex) {
        return new SubList<>(this, fromIndex, toIndex);
    }

    private void rangeCheck(int index) {
        if (index < 0 || index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    private void checkForComodification() {
        if (this.modCount != l.modCount)
            throw new ConcurrentModificationException();
    }
}

class RandomAccessSubList<E> extends SubList<E> implements RandomAccess {
    RandomAccessSubList(AbstractList<E> list, int fromIndex, int toIndex) {
        super(list, fromIndex, toIndex);
    }

    public List<E> subList(int fromIndex, int toIndex) {
        return new RandomAccessSubList<>(this, fromIndex, toIndex);
    }
}

```

### 栗子

```java
package com.yaochow.demo;

import java.util.*;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;


public class ListTest {

    private static List<Integer> TEST_LIST = new ArrayList<>();
    private static Spliterator<Integer> spliterator;


    public static void main(String[] args) throws InterruptedException {
        initArray();
        spliterator = TEST_LIST.spliterator();
        ExecutorService executorService = Executors.newFixedThreadPool(5);

        final Map<String, List<Integer>> countMap = new HashMap<>();
        final CountDownLatch countDownLatch = new CountDownLatch(5);
        for (int i = 0; i < 5; i++) {
            executorService.execute(() -> {
                countMap.put(Thread.currentThread().getName(), new ArrayList<>());
                Spliterator<Integer> integerSpliterator = spliterator.trySplit();
                integerSpliterator.forEachRemaining(
                        integer -> countMap.get(Thread.currentThread().getName()).add(integer));
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();

        countMap.put(Thread.currentThread().getName(), new ArrayList<>());
        spliterator.forEachRemaining(
                integer -> countMap.get(Thread.currentThread().getName()).add(integer));

        countMap.entrySet().forEach(value -> {
            value.getValue().replaceAll(t -> t + 1);
            System.out.println("thread-name:【" + value.getKey() + "】, execute: 【" + Arrays.toString(value.getValue().toArray()) + "】");
        });
    }

    private static void initArray() {
        for (int i = 0; i < 100; i++)
            TEST_LIST.add(i);
    }
}

```

执行结果：

```
thread-name:【pool-1-thread-1】, execute: 【[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50]】
thread-name:【pool-1-thread-3】, execute: 【[76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87]】
thread-name:【pool-1-thread-2】, execute: 【[51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75]】
thread-name:【pool-1-thread-5】, execute: 【[94, 95, 96]】
thread-name:【pool-1-thread-4】, execute: 【[88, 89, 90, 91, 92, 93]】
thread-name:【main】, execute: 【[97, 98, 99, 100]】
```

* 栗子中先初始化一个集合，从0～99共100个元素。
* 通过线程池启动5个线程，每个线程中会将所执行的数据放到map中。
* 最后打印出map中统计的数据

栗子中使用到了CountDownLatch是为了可以最后完整的打印出结果，CountDownLatch可以使一个线程等待其他线程完成后继续执行。

从打印结果中能看出replaceAll方法将集合中原有元素都加了1 (t-> t+1)。

至于spliterator方法将集合分为了几块，每个线程分别去执行不同的分块，这样可以保证线程安全。从打印结果中可以看出，第一个线程执行了50个，第二个25，第三个12，第四个6，第五个3，剩下的由主线程完成的。
也就是说每次都进行二等分分给新的线程来执行。最后的部分应不去调用trySplit方法，而是直接调用forEachRemaining，保证所有的数据都执行完整。

### 总结

1. List是一个有序集合，继承自java.util.Collection。
2. AbstractList继承AbstractCollection实现List接口。
3. 相比java.util.Collection增加了一些与索引相关的方法定义。
4. List中每个元素都对应着一个索引，允许有重复元素。
5. jdk1.8增加了spliterator等新特性。

源码中大部分都标了注释，有些特别简单的就没有加； jdk1.8新增的方法写在了栗子中。

>* 以上源码来自JDK1.8.0_171