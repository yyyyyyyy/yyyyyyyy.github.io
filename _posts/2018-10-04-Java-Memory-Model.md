---
layout: post
date:   2018-10-04 13:36:31 +0800
categories: 
---
## 一、基础
### 1.1 并发编程模型的分类
在并发编程中，我们需要处理两个关键问题：线程之间如何通信以及线程之间如何同步。通信是指线程之间以何种机制来交换信息。在命令式编程中，线程之间的通信机制有两种：共享内存和消息传递。

在共享内存的模型里，线程之间共享程序的公共状态，线程之间通过写-读内存中的公共状态来隐式进行通信。在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过明确的发送消息来显示进行通信。

同步是指程序用于控制不同线程之间操作发生相对顺序的机制。在共享内存并发模型里，同步是显式进行的。程序员必须显示指定某个方法或某段代码需要在线程之间互斥执行。在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的。

Java的并发采用的是共享内存模型，Java线程之间的通信总是隐式进行，整个通信过程对程序员完全透明。
### 1.2 Java内存模型的抽象
在Java中，所有实例域、静态域和数组元素存储在堆内存中，堆内存在线程之间共享。局部变量、方法定义参数和异常处理器参数不会在线程之间共享，它们不会有内存可见行问题，也不受内存模型的影响。

Java线程之间的通信由Java内存模型控制，JMM决定一个线程对共享变量的写入合适对另一个线程可见。
从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：本地内存中存储了该线程以读/写共享变量的副本。

### 1.3 重排序
在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。重排序分三种类型：
​    
* 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
* 指令集并行的重排序。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
* 内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

上述的1属于编译器重排序，2和3属于处理器重排序。这些重排序都可能会导致多线程程序出现内存可见性问题。
对于编译器，JMM的编译器重排序规则会禁止特定类型的编译器重排序。
对于处理器重排序，JMM的处理器重排序规则会要求Java编译器在生成指令序列时，插入特定类型的内存屏障指令，通过内存屏障指令来禁止特定类型的处理器重排序。
JMM属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器排序和处理器重排序，为程序员提供一致的内存可见性保证。

### 1.4 处理器重排序与内存屏障指令
现代的处理器使用写缓冲区来临时保存向内存写入的数据。
写缓冲区可以保证指令流水线持续运行，它可以避免由于处理器停顿下来等待向内存写入数据而产生的延迟。同时，通过以批处理的方式刷新写缓冲区，以及合并写缓冲区中对同一内存地址的多次写，可以减少对内存总线的占用。

### 1.5 happens-before
在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系。这里提到的两个操作既可以是在一个线程内，也可以在不同的线程之间。
​    
* 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
* 监视器锁规则：对一个监视器锁的解锁，happens-before于随后对这个监视器锁的加锁。
* volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
* 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。

两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行，happens-before仅仅要求前一个操作对后一个操作可见，且前一个操作按顺序排在第二个操作之前。

## 二、重排序
### 2.1 数据依赖性
如果两个操作访问同一个变量，且这两个操作中还有一个为写操作，此时这两个操作之间就存在数据依赖性。

编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。

这里所说的数据依赖性仅针对单个处理器中执行的指令序列和单个线程中执行的操作。
### 2.2 as-if-serial
as-if-serial语义是指：不管怎么重排序，程序的执行结果不能被改变。编译器、runtime和处理器都必须遵守as-if-serial语义。

### 2.3 程序顺序规则
在计算机中，软件技术和硬件技术有一个共同的目标：在不改变程序执行结果的前提下，尽可能的开发并行度。

## 三、顺序一致性
### 3.1 数据竞争与顺序一致性保证
当程序未正确同步时，就会存在数据竞争。

JMM对正确同步的多线程程序的内存一致性做了如下保证：
如果程序时正确同步的，程序的执行将具有顺序一致性-即程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同。
这里的同步是指广义上的同步，包括对常用同步原语（lock，volatile和final）的正确使用

### 3.2 顺序一致性内存模型
顺序一致性内存模型是一个被计算机科学家理想化了的理论参考模型，它为程序员提供了极强的内存可见性保证。
​    
* 一个线程中的所有操作必须按照程序的顺序来执行。
* （不管是否同步）所有线程都只能看到一个单一的操作执行顺序。在顺序一致性内存模型中，每个操作都必须院子执行且立刻对所有线程可见。

但是，在JMM中没有这个保证。未同步程序在JMM中不但整体的执行顺序是无序的，而且所有线程看到的操作执行顺序也可能不一致。

### 3.3 同步程序的顺序一致性效果
JMM在具体实现上的基本方针：在不改变（正确同步的）程序执行结果的前提下，尽可能的为编译器和处理器的优化打开方便之门。

### 3.4 未同步程序的执行特性
对于未同步或未正确同步的多线程程序，JMM只提供最小安全性：线程执行时读取到的值，要么是之前某个线程写入的值，要么是默认值，JMM保证线程读操作读取到的值不会无中生有。

## 四、volatile
### 4.1 volatile的特性
当我们声明共享变量volatile后，对这个变量的读/写将会很特别。

理解volatile特性的一个好方法是：把对volatile变量的单个读/写，看成是使用同一个锁对这些单个读/写操作做了同步。

简而言之，volatile变量自身具有下列特性：

* 可见性：对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。
* 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。

### 4.2 volatile的写-读建立的happens before关系
从内存语义的角度来说，volatile于锁有相同的效果：volatile写和锁的释放有相同的内存语义；volatile读与锁的获取有相同的内存语义。
### 4.3 volatile写-读的内存语义
* volatile写的内存语义：当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存。
* volatile读的内存语义：当读一个volatile变量是，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。

## 五、锁
### 5.1 锁的释放-获取建立的happens-before关系
### 5.2 锁释放和获取的内存语义
* 当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中。
* 当线程获取锁时，JMM会把该线程对应的本地内存置为无效。从而使得被监视器保护的临界区代码必须要从主内存中去读取共享变量。

### 5.3 concurrent包的实现
由于Java的CAS同时具有volatile读和volatile写的内存语义，因此Java线程之间的通信现在有了下面四种方式：
​    
* A线程写volatile变量，随后B线程读这个volatile变量。
* A线程写volatile变量，随后B线程用CAS更新这个volatile变量。
* A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。
* A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。

Java的CAS会使用现代处理器上提供的高效机器级别原子指令，这些原子指令以原子方式对内存执行读-改-写操作，这是在多处理器中实现同步的关键。
同时，volatile变量的读/写和CAS可以实现线程之间的通信。把这些特性整合在一起，就形成了整个concurrent包得以实现的基石。

AQS，非阻塞数据结构和原子变量类，这些concurrent包中的基础类都是使用这种模式来实现的，而concurrent包中的高层类又是依赖与这些基础类来实现的。

## 六、final
与前面介绍的锁和volatile相比，对final域的读和写更像是普通的变量访问。对于final域，编译器和处理器要遵守两个重排序规则：
​    
* 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
* 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。

### 6.1 写final域的重排序规则
写final域的重排序规则禁止把final域的写重排序到构造函数之外。这个规则的实现包含下面2个方面：
​    
* JMM禁止编译器把final域的写重排序到构造函数之外。
* 编译器会在final域的写之后，构造函数return之前，插入一个StoreStore屏障。这个屏障禁止处理器把final域的写重排序到构造函数之外。

### 6.2 读final域的重排序规则
读final域的重排序规则如下：
* 在一个线程中，初次读对象引用域初次读该对象包含的final域，JMM禁止处理器重排序这两个操作。编译器会在读final域操作的前面插入一个LoadLoad屏障。

### 6.3 如果final域是引用类型
* 在构造函数内对一个final引用的对象的成员域的写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。

### 6.4 final语义在处理器中的实现

## 七、总结
### 处理器内存模型
顺序一致性内存模型是一个理论参考模型，JMM和处理器内存模型在设计时通常会把顺序一致性内存模型作为参照。

## 八、资料
[深入理解Java内存模型](http://www.infoq.com/cn/articles/java-memory-model-1)