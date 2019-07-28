---
layout: post
title:  "【介绍】Executors"
date:   2017-05-19 00:03:00 +0800
categories: 
---

### 直接上栗子
#### FixedThreadPool

```java
package com.Multithreading;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class FixedThreadPoolTest {
    public static void main(String[] args) {
        ExecutorService newFixedThreadPool = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 10; i++) {
            newFixedThreadPool.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName() + " : " + System.currentTimeMillis());
                    try {
                        Thread.sleep(1000);//睡眠1000ms
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
    }
}

```

执行结果：

```
pool-1-thread-1 : 1538407294756
pool-1-thread-2 : 1538407294757
pool-1-thread-3 : 1538407294757
pool-1-thread-4 : 1538407294758
pool-1-thread-5 : 1538407294760
pool-1-thread-2 : 1538407295758
pool-1-thread-4 : 1538407295759
pool-1-thread-3 : 1538407295760
pool-1-thread-1 : 1538407295760
pool-1-thread-5 : 1538407295764

Process finished with exit code 0
```

从结果中可以看到，线程池中有5个线程在跑。每个线程执行任务的时间间隔是1000ms多一些。

#### WorkStealingPool

```java
package com.Multithreading;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Executors;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;

public class WorkStealingPoolTest {
    public static void main(String[] args) {
        ForkJoinPool newWorkStealingPool = (ForkJoinPool) Executors.newWorkStealingPool();
        ProductListGenerator generator = new ProductListGenerator();
        List<Product> products = generator.generate(100);
        Task task = new Task(products, 0, products.size(), 0.20);
        newWorkStealingPool.execute(task);

        do {
            System.out.printf("Main:Thread Count: %d\n", newWorkStealingPool.getActiveThreadCount());
            System.out.printf("Main:Thread Steal: %d\n", newWorkStealingPool.getStealCount());
            System.out.printf("Main:Parallelism: %d\n", newWorkStealingPool.getParallelism());
        } while (!task.isDone());
        newWorkStealingPool.shutdown();
        if (task.isCompletedNormally())
            System.out.println("Main:The process has completed normally.");
        for (Product product : products) 
            if (product.getPrice() != 12) 
                System.out.printf("Product %s: %f\n", product.getName(), product.getPrice());
               
        System.out.println("Main: End of the program.");
    }

    private static class Task extends RecursiveAction {
        private static final long serialVersionUID = 1L;
        private List<Product> products;
        private int first;
        private int last;
        private double increment;

        public Task(List<Product> products, int first, int last, double increment) {
            this.products = products;
            this.first = first;
            this.last = last;
            this.increment = increment;
        }

        @Override
        protected void compute() {
            if (last - first < 10) {
                updatePrices();
            } else {
                int middle = (last + first) / 2;
                System.out.printf("%s: Pending tasks:%s\n", Thread.currentThread().getName(), getQueuedTaskCount());
                Task t1 = new Task(products, first, middle + 1, increment);
                Task t2 = new Task(products, middle + 1, last, increment);
                invokeAll(t1, t2);
            }
        }

        private void updatePrices() {
            for (int i = first; i < last; i++) {
                Product product = products.get(i);
                product.setPrice(product.getPrice() * (1 + increment));
            }
        }
    }

    private static class Product {
        private String name;
        private double price;
        private String getName() {return name;}
        private void setName(String name) {this.name = name;}
        private double getPrice() {return price;}
        private void setPrice(double price) {this.price = price;}
    }

    private static class ProductListGenerator {
        private List<Product> generate(int size) {
            List<Product> ret = new ArrayList<>();
            for (int i = 0; i < size; i++) {
                Product product = new Product();
                product.setName("Product" + i);
                product.setPrice(10);
                ret.add(product);
            }
            return ret;
        }
    }
}

```

执行结果：

```
Main:Thread Count: 1
ForkJoinPool-1-worker-1: Pending tasks:0
ForkJoinPool-1-worker-1: Pending tasks:1
Main:Thread Steal: 0
Main:Parallelism: 4
Main:Thread Count: 4
ForkJoinPool-1-worker-2: Pending tasks:0
ForkJoinPool-1-worker-1: Pending tasks:1
ForkJoinPool-1-worker-0: Pending tasks:0
ForkJoinPool-1-worker-2: Pending tasks:1
Main:Thread Steal: 0
Main:Parallelism: 4
ForkJoinPool-1-worker-3: Pending tasks:0
Main:Thread Count: 4
ForkJoinPool-1-worker-2: Pending tasks:1
ForkJoinPool-1-worker-0: Pending tasks:1
ForkJoinPool-1-worker-1: Pending tasks:1
ForkJoinPool-1-worker-0: Pending tasks:0
ForkJoinPool-1-worker-2: Pending tasks:0
Main:Thread Steal: 0
...
Main:Thread Count: 3
ForkJoinPool-1-worker-3: Pending tasks:1
...
Main:Thread Count: 2
ForkJoinPool-1-worker-0: Pending tasks:0
ForkJoinPool-1-worker-1: Pending tasks:0
Main:Thread Steal: 2
Main:Parallelism: 4
Main:The process has completed normally.
Main: End of the program.

Process finished with exit code 0
```

WorkStealingPool使用的是ForkJoinPool，默认线程数是支持JVM的线程数量，从打印的结果中可以看到执行的线程数量是动态的，根据运行情况尽可能满足需求。

#### SingleThreadExecutor

```java
package com.Multithreading;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SingleThreadExecutorTest {
    public static void main(String[] args) {
        ExecutorService newSingleThreadExecutor = Executors.newSingleThreadExecutor();
        for (int i : new int[]{1, 2, 3, 4, 5}) {
            newSingleThreadExecutor.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName() + " : " + System.currentTimeMillis());
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        newSingleThreadExecutor.shutdown();
    }
}

```

执行结果：

```
pool-1-thread-1 : 1538470592027
pool-1-thread-1 : 1538470593032
pool-1-thread-1 : 1538470594034
pool-1-thread-1 : 1538470595035
pool-1-thread-1 : 1538470596040

Process finished with exit code 0
```

从结果中可以看到，线程池中只有1个线程在跑。每个线程执行任务的时间间隔是1000ms多一些。

#### newCachedThreadPool

```java
package com.Multithreading;

import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class CachedThreadPoolTest {
    public static void main(String[] args) throws InterruptedException {
        ThreadPoolExecutor newCachedThreadPool = (ThreadPoolExecutor) Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++)
            newCachedThreadPool.execute(new RunnableTest());

        TimeUnit.SECONDS.sleep(31);
        System.out.println("pool size : " + newCachedThreadPool.getPoolSize());
        newCachedThreadPool.execute(new RunnableTest());
        TimeUnit.SECONDS.sleep(31);
        System.out.println("pool size : " + newCachedThreadPool.getPoolSize());
        newCachedThreadPool.shutdown();
    }

    private static class RunnableTest implements Runnable {

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " : " + System.currentTimeMillis());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```

执行结果：

```
pool-1-thread-1 : 1538473539047
pool-1-thread-2 : 1538473539047
pool-1-thread-3 : 1538473539048
pool-1-thread-4 : 1538473539048
pool-1-thread-5 : 1538473539049
pool size : 5
pool-1-thread-2 : 1538473570050
pool size : 1

Process finished with exit code 0
```

从结果中可以看到，缓存线程池中第一次发起5个线程,此时poolsize是5，31s后发起一个线程，再31s后获取poolsize是1，默认60s回收空闲线程。

#### ScheduledThreadPool

```java
package com.Multithreading;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class ScheduledThreadPoolTest {
    private static AtomicInteger ac = new AtomicInteger(0);
    public static void main(String[] args) {
        ScheduledExecutorService newScheduledThreadPool = Executors.newScheduledThreadPool(2);
        newScheduledThreadPool.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + " : " + System.currentTimeMillis());
                ac.getAndIncrement();
            }
        }, 1000, 1000, TimeUnit.MILLISECONDS);
        newScheduledThreadPool.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + " : " + System.currentTimeMillis());
            }
        }, 1000, 1000, TimeUnit.MILLISECONDS);
        while (ac.get() < 5) {}
        newScheduledThreadPool.shutdown();
    }
}

```

执行结果：

```
pool-1-thread-1 : 1538474649680
pool-1-thread-2 : 1538474649680
pool-1-thread-1 : 1538474650679
pool-1-thread-2 : 1538474650681
pool-1-thread-1 : 1538474651676
pool-1-thread-2 : 1538474651683
pool-1-thread-1 : 1538474652678
pool-1-thread-2 : 1538474652684
pool-1-thread-1 : 1538474653680

Process finished with exit code 0
```

Schedule与Timer比，可以同时处理多个定时任务。