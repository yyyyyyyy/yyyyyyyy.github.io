---
layout: post
title:  "【源码】SortedMap& NavigableMap"
date:   2016-05-19 01:01:00 +0800
categories: 
comments: true

---

## java.util.SortedMap

```java
package java.util;

/*定义有序Map*/
public interface SortedMap<K,V> extends Map<K,V> {

    Comparator<? super K> comparator();//比较器
    SortedMap<K,V> subMap(K fromKey, K toKey);//获取范围内的值
    SortedMap<K,V> headMap(K toKey);//获取头Map
    SortedMap<K,V> tailMap(K fromKey);//获取尾部Map
    K firstKey();//获取头key
    K lastKey();//获取尾key
    Set<K> keySet();
    Collection<V> values();
    Set<Map.Entry<K, V>> entrySet();
}

```

## java.util.NavigableMap

```java
package java.util;

/*继承于SortedMap， 有序Map， 提供了多种有序特性方法*/
public interface NavigableMap<K,V> extends SortedMap<K,V> {

    Map.Entry<K,V> lowerEntry(K key);//小于
    K lowerKey(K key);//小于
    Map.Entry<K,V> floorEntry(K key);//小于等于
    K floorKey(K key);//小于等于
    Map.Entry<K,V> ceilingEntry(K key);//大于等于
    K ceilingKey(K key);//大于等于
    Map.Entry<K,V> higherEntry(K key);//大于
    K higherKey(K key);//大于
    Map.Entry<K,V> firstEntry();//头Entry
    Map.Entry<K,V> lastEntry();//尾Entry
    Map.Entry<K,V> pollFirstEntry();//获取并删除头Entry
    Map.Entry<K,V> pollLastEntry();//获取并删除尾部Entry
    NavigableMap<K,V> descendingMap();//返回逆向排序的Map
    NavigableSet<K> navigableKeySet();
    NavigableSet<K> descendingKeySet();
    NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive,
                             K toKey,   boolean toInclusive);
    NavigableMap<K,V> headMap(K toKey, boolean inclusive);
    NavigableMap<K,V> tailMap(K fromKey, boolean inclusive);
    SortedMap<K,V> subMap(K fromKey, K toKey);
    SortedMap<K,V> headMap(K toKey);
    SortedMap<K,V> tailMap(K fromKey);
}

```



### 总结

有序Map通过Comparator来实现， 提供了排序后的特性方法。



>* 以上源码均来自JDK1.8.0_171 