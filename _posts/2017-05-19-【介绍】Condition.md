---
layout: post
title:  "【介绍】Condition"
date:   2017-05-19 00:08:00 +0800
categories:  
---

Condition 可以对锁进行更精确的控制。

Condition内部方法与Object的wait(), notify(), notifyAll()相似。

区别是Object的方法是在synchronized内使用，而Condition是在Lock中使用。

先看看栗子：

```java
package com.yaochow.demo;


import org.junit.Test;

public class ObjectTest {

    @Test
    public void test() throws InterruptedException {

        Thread demo = new ThreadDemo("yaochow");
        synchronized (demo) {
            demo.start();
            System.out.println("wait yaochow finish");
            demo.wait();
            System.out.println("main go on");
        }
    }
}

class ThreadDemo extends Thread {

    public ThreadDemo(String name) {
        super(name);
    }

    @Override
    public void run() {
        synchronized (this) {
            System.out.println(Thread.currentThread().getName() + " finished.");
            notify();
        }
    }

}
```

输出：

```
wait yaochow finish
yaochow finished.
main go on

```

可以看到，在synchronized中，通过wait(), notify()可以更精确的控制线程的使用。

下面看看Condition的栗子：

```java
package com.yaochow.demo;

import org.junit.Test;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ConditionTest {

    private static Lock lock = new ReentrantLock();
    private static Condition condition = lock.newCondition();

    @Test
    public void test() {

        try {
            Thread demo = new ConditionDemo("yaochow", lock, condition);
            lock.lock();
            demo.start();
            System.out.println("wait yaochow finish");
            condition.await();
            System.out.println("go on");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}


class ConditionDemo extends Thread {

    private Lock lock;
    private Condition condition;

    public ConditionDemo(String name, Lock lock, Condition condition) {
        super(name);
        this.lock = lock;
        this.condition = condition;
    }

    @Override
    public void run() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + " finished.");
            condition.signal();
        } finally {
            lock.unlock();
        }
    }
}
```

输出：

```
wait yaochow finish
yaochow finished.
go on
```

看到了上面两个栗子，那Condition和Object里的方法比较，好在哪里呢？

一个Lock可以创建多个Condition，这样可以更精细的控制线程的操作。



源码下次在撸。

