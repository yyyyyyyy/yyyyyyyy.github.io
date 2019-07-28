---
layout: post
title:  "【介绍】CountDownLatch& Semaphore"
date:   2017-05-19 00:09:00 +0800
categories:  
---

先看看栗子：

```java
package com.yaochow.demo;

import org.junit.Test;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class TestPlatform {

    public static int clientTotal = 10000;//访问数
    public static int threadTotal = 4;//线程数
    public static int count = 0;

    @Test
    public void test() throws InterruptedException {

        ExecutorService executorService = Executors.newCachedThreadPool();

        final Semaphore semaphore = new Semaphore(threadTotal);
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        for (int i = 0; i < clientTotal; i++) {
            executorService.execute(() -> {
                try {
                    semaphore.acquire();
                    add();
                    semaphore.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        System.out.println(count);
    }

    private static void add() {
        count++;//每访问一次加一
    }
}

```

结果：
```
9997
```

从结果中可以看到，出现了并发问题。

在工作中有时会遇见各类奇怪的问题，在解决问题的时候首先需要让问题复现，再去一点点扒代码定位问题。
各大公司实际上都有提供并发测试环境，不过要部署到测试环境，这样就比较麻烦。
所以试着写一个并发测试的栗子。

这个就是本地并发测试的一个栗子，当线程数越高的时候，越容易复现问题。通过这个栗子也可以了解一下CountDownLatch& Semaphore。

CountDownLatch的作用是让一个或N个线程等待其他线程运行结束之后再继续运行。

Semaphore的作用是保证同一时间最多只有固定的线程在工作。

以上，源码下次在撸。

