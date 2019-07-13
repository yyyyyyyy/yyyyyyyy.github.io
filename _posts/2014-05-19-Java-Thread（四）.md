---
layout: post
title:  "【自撸】J.U.C（Executors）"
date:   2014-05-19 00:02:06 +0800
categories: JUC 线程 Executors ThreadPools
---

Java Executors、ThreadPools源码笔记。

## 一、java.util.concurrent.Executors

```java
package java.util.concurrent;
import java.util.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.security.AccessControlContext;
import java.security.AccessController;
import java.security.PrivilegedAction;
import java.security.PrivilegedExceptionAction;
import java.security.PrivilegedActionException;
import java.security.AccessControlException;
import sun.security.util.SecurityConstants;

public class Executors {

    /*返回固定大小的线程池。Executors.newFixedThreadPool(5);这里指定线程池中线程的数量固定为“5”*/
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

    /*返回一个总是能够满足并行运行的线程池，可以动态增加或减少线程数量，并通过多个队列减少竞争。*/
    public static ExecutorService newWorkStealingPool(int parallelism) {
        return new ForkJoinPool
            (parallelism,
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);//可以看到workStealingPool使用的是ForkJoinPool
    }

    public static ExecutorService newWorkStealingPool() {
        return new ForkJoinPool
            (Runtime.getRuntime().availableProcessors(),
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }

    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }

    /*返回一个线程池，线程池中只保持一个线程在运行*/
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }

    /*返回一个缓存线程池,缓存的线程默认空闲时间超过60s会被回收。*/
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

    public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
    }

    /*返回一个定时线程池，可以以固定频率执行，也可以以成功执行到下次开始时间延时执行。*/
    public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
        return new DelegatedScheduledExecutorService
            (new ScheduledThreadPoolExecutor(1));
    }

    public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory) {
        return new DelegatedScheduledExecutorService
            (new ScheduledThreadPoolExecutor(1, threadFactory));
    }

    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }

    public static ScheduledExecutorService newScheduledThreadPool(
            int corePoolSize, ThreadFactory threadFactory) {
        return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
    }

    /*返回一个ExecutorService的委托对象*/
    public static ExecutorService unconfigurableExecutorService(ExecutorService executor) {
        if (executor == null)
            throw new NullPointerException();
        return new DelegatedExecutorService(executor);
    }

    /*返回一个ScheduledExecutorService的委托对象*/
    public static ScheduledExecutorService unconfigurableScheduledExecutorService(ScheduledExecutorService executor) {
        if (executor == null)
            throw new NullPointerException();
        return new DelegatedScheduledExecutorService(executor);
    }

    /*返回默认的线程工厂*/
    public static ThreadFactory defaultThreadFactory() {
        return new DefaultThreadFactory();
    }

    /*返回一个与当前线程拥有相同权限的线程工厂*/
    public static ThreadFactory privilegedThreadFactory() {
        return new PrivilegedThreadFactory();
    }

    /*返回一个拥有固定返回值的Callable对象*/
    public static <T> Callable<T> callable(Runnable task, T result) {
        if (task == null)
            throw new NullPointerException();
        return new RunnableAdapter<T>(task, result);
    }

    /*返回一个返回null值的Callable对象*/
    public static Callable<Object> callable(Runnable task) {
        if (task == null)
            throw new NullPointerException();
        return new RunnableAdapter<Object>(task, null);
    }

    /*当运行完成后会执行action并返回action结果*/
    public static Callable<Object> callable(final PrivilegedAction<?> action) {
        if (action == null)
            throw new NullPointerException();
        return new Callable<Object>() {
            public Object call() { return action.run(); }};
    }

    /*同上，带有抛出异常*/
    public static Callable<Object> callable(final PrivilegedExceptionAction<?> action) {
        if (action == null)
            throw new NullPointerException();
        return new Callable<Object>() {
            public Object call() throws Exception { return action.run(); }};
    }

    /*运行的callable与当前线程具有相同权限*/
    public static <T> Callable<T> privilegedCallable(Callable<T> callable) {
        if (callable == null)
            throw new NullPointerException();
        return new PrivilegedCallable<T>(callable);
    }

    /*运行的callable与当前线程具有相同权限，并使用当前的类加载器*/
    public static <T> Callable<T> privilegedCallableUsingCurrentClassLoader(Callable<T> callable) {
        if (callable == null)
            throw new NullPointerException();
        return new PrivilegedCallableUsingCurrentClassLoader<T>(callable);
    }

    static final class RunnableAdapter<T> implements Callable<T> {
        final Runnable task;
        final T result;
        RunnableAdapter(Runnable task, T result) {
            this.task = task;
            this.result = result;
        }
        public T call() {
            task.run();
            return result;
        }
    }

    static final class PrivilegedCallable<T> implements Callable<T> {
        private final Callable<T> task;
        private final AccessControlContext acc;

        PrivilegedCallable(Callable<T> task) {
            this.task = task;
            this.acc = AccessController.getContext();
        }

        public T call() throws Exception {
            try {
                return AccessController.doPrivileged(
                    new PrivilegedExceptionAction<T>() {
                        public T run() throws Exception {
                            return task.call();
                        }
                    }, acc);
            } catch (PrivilegedActionException e) {
                throw e.getException();
            }
        }
    }

    static final class PrivilegedCallableUsingCurrentClassLoader<T> implements Callable<T> {
        private final Callable<T> task;
        private final AccessControlContext acc;
        private final ClassLoader ccl;

        PrivilegedCallableUsingCurrentClassLoader(Callable<T> task) {
            SecurityManager sm = System.getSecurityManager();
            if (sm != null) {
                // Calls to getContextClassLoader from this class
                // never trigger a security check, but we check
                // whether our callers have this permission anyways.
                sm.checkPermission(SecurityConstants.GET_CLASSLOADER_PERMISSION);

                // Whether setContextClassLoader turns out to be necessary
                // or not, we fail fast if permission is not available.
                sm.checkPermission(new RuntimePermission("setContextClassLoader"));
            }
            this.task = task;
            this.acc = AccessController.getContext();
            this.ccl = Thread.currentThread().getContextClassLoader();
        }

        public T call() throws Exception {
            try {
                return AccessController.doPrivileged(
                    new PrivilegedExceptionAction<T>() {
                        public T run() throws Exception {
                            Thread t = Thread.currentThread();
                            ClassLoader cl = t.getContextClassLoader();
                            if (ccl == cl) {
                                return task.call();
                            } else {
                                t.setContextClassLoader(ccl);
                                try {
                                    return task.call();
                                } finally {
                                    t.setContextClassLoader(cl);
                                }
                            }
                        }
                    }, acc);
            } catch (PrivilegedActionException e) {
                throw e.getException();
            }
        }
    }

    static class DefaultThreadFactory implements ThreadFactory {
        private static final AtomicInteger poolNumber = new AtomicInteger(1);
        private final ThreadGroup group;
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;

        DefaultThreadFactory() {
            SecurityManager s = System.getSecurityManager();
            group = (s != null) ? s.getThreadGroup() :
                                  Thread.currentThread().getThreadGroup();
            namePrefix = "pool-" +
                          poolNumber.getAndIncrement() +
                         "-thread-";
        }

        public Thread newThread(Runnable r) {
            Thread t = new Thread(group, r,
                                  namePrefix + threadNumber.getAndIncrement(),
                                  0);
            if (t.isDaemon())
                t.setDaemon(false);
            if (t.getPriority() != Thread.NORM_PRIORITY)
                t.setPriority(Thread.NORM_PRIORITY);
            return t;
        }
    }

    static class PrivilegedThreadFactory extends DefaultThreadFactory {
        private final AccessControlContext acc;
        private final ClassLoader ccl;

        PrivilegedThreadFactory() {
            super();
            SecurityManager sm = System.getSecurityManager();
            if (sm != null) {
                // Calls to getContextClassLoader from this class
                // never trigger a security check, but we check
                // whether our callers have this permission anyways.
                sm.checkPermission(SecurityConstants.GET_CLASSLOADER_PERMISSION);

                // Fail fast
                sm.checkPermission(new RuntimePermission("setContextClassLoader"));
            }
            this.acc = AccessController.getContext();
            this.ccl = Thread.currentThread().getContextClassLoader();
        }

        public Thread newThread(final Runnable r) {
            return super.newThread(new Runnable() {
                public void run() {
                    AccessController.doPrivileged(new PrivilegedAction<Void>() {
                        public Void run() {
                            Thread.currentThread().setContextClassLoader(ccl);
                            r.run();
                            return null;
                        }
                    }, acc);
                }
            });
        }
    }

    static class DelegatedExecutorService extends AbstractExecutorService {
        private final ExecutorService e;
        DelegatedExecutorService(ExecutorService executor) { e = executor; }
        public void execute(Runnable command) { e.execute(command); }
        public void shutdown() { e.shutdown(); }
        public List<Runnable> shutdownNow() { return e.shutdownNow(); }
        public boolean isShutdown() { return e.isShutdown(); }
        public boolean isTerminated() { return e.isTerminated(); }
        public boolean awaitTermination(long timeout, TimeUnit unit)
            throws InterruptedException {
            return e.awaitTermination(timeout, unit);
        }
        public Future<?> submit(Runnable task) {
            return e.submit(task);
        }
        public <T> Future<T> submit(Callable<T> task) {
            return e.submit(task);
        }
        public <T> Future<T> submit(Runnable task, T result) {
            return e.submit(task, result);
        }
        public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
            throws InterruptedException {
            return e.invokeAll(tasks);
        }
        public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                             long timeout, TimeUnit unit)
            throws InterruptedException {
            return e.invokeAll(tasks, timeout, unit);
        }
        public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
            throws InterruptedException, ExecutionException {
            return e.invokeAny(tasks);
        }
        public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                               long timeout, TimeUnit unit)
            throws InterruptedException, ExecutionException, TimeoutException {
            return e.invokeAny(tasks, timeout, unit);
        }
    }

    static class FinalizableDelegatedExecutorService
        extends DelegatedExecutorService {
        FinalizableDelegatedExecutorService(ExecutorService executor) {
            super(executor);
        }
        protected void finalize() {
            super.shutdown();
        }
    }

    /**
     * A wrapper class that exposes only the ScheduledExecutorService
     * methods of a ScheduledExecutorService implementation.
     */
    static class DelegatedScheduledExecutorService
            extends DelegatedExecutorService
            implements ScheduledExecutorService {
        private final ScheduledExecutorService e;
        DelegatedScheduledExecutorService(ScheduledExecutorService executor) {
            super(executor);
            e = executor;
        }
        public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit) {
            return e.schedule(command, delay, unit);
        }
        public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit) {
            return e.schedule(callable, delay, unit);
        }
        public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit) {
            return e.scheduleAtFixedRate(command, initialDelay, period, unit);
        }
        public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit) {
            return e.scheduleWithFixedDelay(command, initialDelay, delay, unit);
        }
    }

    /** Cannot instantiate. */
    private Executors() {}
}

```

Executors提供了5种创建线程池的静态工厂方法，包括固定大小的线程池、可以动态扩展的线程池、单线程线程池、缓存线程池、定时线程池。

### 1.1 FixedThreadPool

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

### 1.2 WorkStealingPool

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

###  1.3 SingleThreadExecutor

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

### 1.4 newCachedThreadPool

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

### 1.5 ScheduledThreadPool

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

## 二、java.util.concurrent.ThreadPoolExecutor#execute

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
	private volatile ThreadFactory threadFactory; //线程工厂，线程池的线程是通过threadFactory创建的，默认使用DefaultThreadFactory。
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    
    /*创建了线程池后，此时只是初始化了线程池内部属性，并没有真正的创建线程*/
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
    //线程池执行任务方法，当调用addWorker方法时会通过new Worker()新建线程。
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
        //当运行的线程数小于设置的线程池线程数时，直接新建一个worker，在worker中会通过ThreadFactory新建线程。
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        //当running >= corePoolSize && 线程池处于运行状态，尝试将command添加到队列中。
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            //二次检查
            if (! isRunning(recheck) && remove(command))
                reject(command);//抛出异常
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);//添加一个没有任务的worker
        }
        else if (!addWorker(command, false))//直接添加worker
            reject(command);//失败-->抛出异常
    }
    
     private boolean addWorker(Runnable firstTask, boolean core) {
        //...
        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
            w = new Worker(firstTask);//创建worker
            final Thread t = w.thread;
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                //...
                if (workerAdded) {
                    t.start();//调用线程start()方法，进入就绪状态。
                    workerStarted = true;
                }
            }
        } finally {
            if (! workerStarted)
                addWorkerFailed(w);
        }
        return workerStarted;
    }
    
    public ThreadFactory getThreadFactory() {
        return threadFactory;
    }
    
    //ThreadPoolExecutor内部类，管理线程。
    private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
    {
        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);//创建线程
        }
    }
}
```







## 三、总结

Executors是一个线程池静态工厂，提供五种特性不同的线程池创建。



> Java源码为jdk1.8.0_171

