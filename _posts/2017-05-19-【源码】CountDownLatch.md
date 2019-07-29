---
layout: post
title:  "【源码】CountDownLatch"
date:   2017-05-19 00:09:01 +0800
categories:  

---

### java.util.concurrent.CountDownLatch

```java
package java.util.concurrent;
//从这儿就可以看出CountDownLatch就是通过AQS来实现的。
import java.util.concurrent.locks.AbstractQueuedSynchronizer;


public class CountDownLatch {
    //静态内部类，继承了AQS
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }

        protected int tryAcquireShared(int acquires) {
          //当state == 0 时表示计数器计数个数为0，可以解除wait阻塞。
          // 在AQS的doAcquireSharedInterruptibly方法或者doAcquireSharedNanos方法中回调
            return (getState() == 0) ? 1 : -1;
        }

        protected boolean tryReleaseShared(int releases) {//计数器减一
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }

    private final Sync sync;

    public CountDownLatch(int count) {//设置计数器计数个数
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }

    public void await() throws InterruptedException {//等待其他线程完成
        sync.acquireSharedInterruptibly(1);
    }

    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {//等待，时间到了打断
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

    public void countDown() {//计数器减一
        sync.releaseShared(1);
    }

    public long getCount() {//计数器剩余计数个数
        return sync.getCount();
    }

    public String toString() {
        return super.toString() + "[Count = " + sync.getCount() + "]";
    }
}

```

### 总结

源码篇幅较小，注释也很明白了。

> Java源码为jdk1.8.0_171