---
layout: post
title:  "【自撸】J.U.C（ReentrantLock）"
date:   2014-05-19 00:02:08 +0800
categories: JUC AQS CAS ReentrantLock
---

#### 可重入锁

话不多说，先看栗子：

```java
package com.yaochow.demo;

import org.junit.Test;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockTest {

    @Test
    public void testReentrantLock() throws InterruptedException {
        ReentrantLockDemo reentrantLockDemo = new ReentrantLockDemo();
        new Thread(() -> reentrantLockDemo.minusOne("bird", 3)).start();
        new Thread(() -> reentrantLockDemo.minusOne("monkey", 2)).start();
        new Thread(() -> new ReentrantLockDemo().minusOne("lion", 1)).start();

        Thread.sleep(1000);
    }
}

class ReentrantLockDemo {
    private static int num = 3;//定义类变量
    private static Lock lock = new ReentrantLock();//锁

    public void minusOne(String name, int want) {

        lock.lock();
        System.out.println(String.format("%s got lock", name));

        try {
            if (num - 1 < 0 || want == 0) return;
            num--;
            System.out.println(String.format("%s execute, num: %d", name, num));

            minusOne(name, --want);
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(String.format("%s unlock", name));
            lock.unlock();
        }

    }
}
```

运行结果：

```
bird got lock
bird execute, num: 2
bird got lock
bird execute, num: 1
bird got lock
bird execute, num: 0
bird got lock
bird unlock
bird unlock
bird unlock
bird unlock
monkey got lock
monkey unlock
lion got lock
lion unlock

```

可以看到，当bird拿到锁后当前线程可以再次获取锁，而monkey和lion需要等锁拥有者释放掉锁后才能够获取到锁，这就是可重入的概念。

直接上源码吧

```java
package java.util.concurrent.locks;
import java.util.concurrent.TimeUnit;
import java.util.Collection;

/**
 *
 * @since 1.5
 * @author Doug Lea  //对大神的敬畏
 */
public class ReentrantLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = 7373984872572414699L;
    private final Sync sync;//定义了一个Sync对象

    //锁的父类，lock在子类中实现
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        abstract void lock();
		//尝试获取非公平锁
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();//获取当前锁的状态
            if (c == 0) {//状态0代表锁处于空闲状态，通过CAS修改状态为1（获取锁）.
                if (compareAndSetState(0, acquires)) {
                  //设置获取到排他锁是当前线程
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
          	//如果当前线程是已经拿到锁的线程，那么可以重入，将状态+1.
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;//尝试失败
        }

      	//释放锁
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {//因为可重入，所以要检查状态是否为0。
                free = true;
              	//在0的时候清掉当前排他锁状态
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

      	//当前线程是不是占有排他锁
        protected final boolean isHeldExclusively() {
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

      	//获取ConditionObject对象
        final ConditionObject newCondition() {
            return new ConditionObject();
        }

        // Methods relayed from outer class
		//当前线程是不是排他锁拥有者
        final Thread getOwner() {
            return getState() == 0 ? null : getExclusiveOwnerThread();
        }
				
        final int getHoldCount() {
            return isHeldExclusively() ? getState() : 0;
        }

        final boolean isLocked() {
            return getState() != 0;
        }

        /**
         * Reconstitutes the instance from a stream (that is, deserializes it).
         */
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }

    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
          //AbstractQueuedSynchronizer中的方法，原子操作更改锁的状态。
            if (compareAndSetState(0, 1))
              //设置占用排他锁的是当前线程
                setExclusiveOwnerThread(Thread.currentThread());
            else
              //没有抢到锁后进入acquire方法，此方法是在AQS中实现的，方法代码放在了下面。
                acquire(1);
        }

      	//尝试获取锁
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }

    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
          	//公平锁不会马上去申请锁的权限，而是调用acquire()方法排队
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
              	//判断是否排到了自己，是的话获取锁
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
          	//这部分和非公平锁一样
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }

    //默认非公平锁
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

    //获取锁
    public void lock() {
        sync.lock();
    }

    //中断锁的获取
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    //尝试获取非公平锁
    public boolean tryLock() {
        return sync.nonfairTryAcquire(1);
    }

    
    //在时间内尝试获取锁，在达到时间时未获得，停止获取锁。
    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }

   //释放锁
    public void unlock() {
        sync.release(1);
    }

    
    public Condition newCondition() {
        return sync.newCondition();
    }

    
    public int getHoldCount() {
        return sync.getHoldCount();
    }

    
    public boolean isHeldByCurrentThread() {
        return sync.isHeldExclusively();
    }

    
    public boolean isLocked() {
        return sync.isLocked();
    }

    
    public final boolean isFair() {
        return sync instanceof FairSync;
    }

   
    protected Thread getOwner() {
        return sync.getOwner();
    }

    
    public final boolean hasQueuedThreads() {
        return sync.hasQueuedThreads();
    }

    
    public final boolean hasQueuedThread(Thread thread) {
        return sync.isQueued(thread);
    }

    
    public final int getQueueLength() {
        return sync.getQueueLength();
    }

    
    protected Collection<Thread> getQueuedThreads() {
        return sync.getQueuedThreads();
    }

    
    public boolean hasWaiters(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.hasWaiters((AbstractQueuedSynchronizer.ConditionObject)condition);
    }

    
    public int getWaitQueueLength(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.getWaitQueueLength((AbstractQueuedSynchronizer.ConditionObject)condition);
    }

    ted Collection<Thread> getWaitingThreads(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.getWaitingThreads((AbstractQueuedSynchronizer.ConditionObject)condition);
    }

    
    public String toString() {
        Thread o = sync.getOwner();
        return super.toString() + ((o == null) ?
                                   "[Unlocked]" :
                                   "[Locked by thread " + o.getName() + "]");
    }
}

```



涉及到的AQS方法

```
    public final void acquire(int arg) {
      	//tryAcquire方法是子类重写的，公平锁和非公平锁实现不一样，见上面的代码
        if (!tryAcquire(arg) &&
            //新生成一个Node进行排队
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
          selfInterrupt();
    }

		//在队列尾部添加一个Node
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }

    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }

		//自旋获取锁
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
		//检查排队进度
    public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```


以上就是ReentrantLock的源码，根据源码可以了解到，ReentrantLock是如何重入的。
再一个就是公平锁和非公平锁，公平锁在新线程获取锁的时候会直接进行排队；而非公平锁会直接获取锁，失败后才会进行排队。


J.U.C中最核心的就是AQS和CAS，基本上都是围绕着这两部分。



以上源码均来自JDK1.8.0_171
