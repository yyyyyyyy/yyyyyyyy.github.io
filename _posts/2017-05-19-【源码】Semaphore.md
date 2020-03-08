---
layout: post
title:  "【源码】Semaphore"
date:   2017-05-19 00:09:02 +0800
categories:  

---

### java.util.concurrent.Semaphore

```java
package java.util.concurrent;
import java.util.Collection;
import java.util.concurrent.locks.AbstractQueuedSynchronizer;


public class Semaphore implements java.io.Serializable {
    private static final long serialVersionUID = -3222578661600680210L;
    /** All mechanics via AbstractQueuedSynchronizer subclass */
    private final Sync sync;

    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 1192457210091910933L;

        Sync(int permits) {//设置资源限制
            setState(permits);
        }

        final int getPermits() {
            return getState();
        }

        final int nonfairTryAcquireShared(int acquires) {
            for (;;) {//通过CAS尝试获取锁，获取失败返回-1。
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }

        protected final boolean tryReleaseShared(int releases) {
            for (;;) {//通过CAS释放锁
                int current = getState();
                int next = current + releases;
                if (next < current) // overflow
                    throw new Error("Maximum permit count exceeded");
                if (compareAndSetState(current, next))
                    return true;
            }
        }

        final void reducePermits(int reductions) {
            for (;;) {//减少资源
                int current = getState();
                int next = current - reductions;
                if (next > current) // underflow
                    throw new Error("Permit count underflow");
                if (compareAndSetState(current, next))
                    return;
            }
        }

        final int drainPermits() {
            for (;;) {//设置为无资源
                int current = getState();
                if (current == 0 || compareAndSetState(current, 0))
                    return current;
            }
        }
    }

    /**
     * NonFair version
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = -2694183684443567898L;

        NonfairSync(int permits) {
            super(permits);
        }

        protected int tryAcquireShared(int acquires) {
          //直接获取锁，AQS回调
            return nonfairTryAcquireShared(acquires);
        }
    }

    /**
     * Fair version
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = 2014338818796000944L;

        FairSync(int permits) {
            super(permits);
        }

        protected int tryAcquireShared(int acquires) {
            for (;;) {//排队获取锁，AQS回调
                if (hasQueuedPredecessors())
                    return -1;
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
    }

    public Semaphore(int permits) {
        sync = new NonfairSync(permits);//默认非公平锁
    }

    public Semaphore(int permits, boolean fair) { 
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }

    public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);//获取锁
    }

    public void acquireUninterruptibly() {
        sync.acquireShared(1);//获取锁，不能被中断
    }

    public boolean tryAcquire() {//通过非公平锁获取锁，不是一定能获取到锁
        return sync.nonfairTryAcquireShared(1) >= 0;
    }

    public boolean tryAcquire(long timeout, TimeUnit unit)
        throws InterruptedException {//时间内获取锁，否则抛出异常
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

    public void release() {//释放资源
        sync.releaseShared(1);
    }

    public void acquire(int permits) throws InterruptedException {//可释放permits个资源
        if (permits < 0) throw new IllegalArgumentException();
        sync.acquireSharedInterruptibly(permits);
    }

    public void acquireUninterruptibly(int permits) {//可释放permits个资源，不被中断
        if (permits < 0) throw new IllegalArgumentException();
        sync.acquireShared(permits);
    }

    public boolean tryAcquire(int permits) {//可释放permits个资源，不是一定成功
        if (permits < 0) throw new IllegalArgumentException();
        return sync.nonfairTryAcquireShared(permits) >= 0;
    }

    public boolean tryAcquire(int permits, long timeout, TimeUnit unit)
        throws InterruptedException {
        if (permits < 0) throw new IllegalArgumentException();
        return sync.tryAcquireSharedNanos(permits, unit.toNanos(timeout));
    }

    public void release(int permits) {
        if (permits < 0) throw new IllegalArgumentException();
        sync.releaseShared(permits);
    }

    public int availablePermits() {
        return sync.getPermits();
    }

    public int drainPermits() {
        return sync.drainPermits();
    }

    protected void reducePermits(int reduction) {
        if (reduction < 0) throw new IllegalArgumentException();
        sync.reducePermits(reduction);
    }

    public boolean isFair() {
        return sync instanceof FairSync;
    }

    public final boolean hasQueuedThreads() {
        return sync.hasQueuedThreads();
    }

    public final int getQueueLength() {
        return sync.getQueueLength();
    }

    protected Collection<Thread> getQueuedThreads() {
        return sync.getQueuedThreads();
    }

    public String toString() {
        return super.toString() + "[Permits = " + sync.getPermits() + "]";
    }
}

```

### 总结

信号量可以限制同一时刻有多少资源可以被使用。也分为公平和非公平锁，具体看源码注释就好。

后面没加注释的和前面差不多，可以自行理解。



> Java源码为jdk1.8.0_171