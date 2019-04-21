---
title: Java 核心技术-进阶篇二
date: 2019-03-24 21:10:00
categories:
- Java
tags:
- Java
- Java并发编程
---

# 什么情况下Java程序会产生死锁？如何定位、修复？

![Java核心技术-进阶篇二\死锁](Java核心技术-进阶篇二\死锁.png)

死锁是应用程序表现的一种状态，通常是两个实体之间相互依赖导致彼此一直处于等待之中。死锁不仅仅会发生在线程之间，资源独占的进程之间的进程也会出现死锁。

多线程环境中的死锁现象：两个或两个以上的多线程之间，由于互相持有对方需要的锁，而永久处于阻塞状态。

## 定位死锁

死锁一般无法通过在线方式解决，只能重启，修复程序本身问题。

利用 jstack 工具获取线程栈，在定位相互之间的依赖关系，进而找到死锁。

```java
public class DeadLockSample extends Thread {
    private String first;
    private String second;

    public DeadLockSample(String name, String first, String second) {
        super(name);
        this.first = first;
        this.second = second;
    }
    @Override
    public void run() {
        synchronized (first) {
            System.out.println(this.getName() + "obtained:" + first);
            try {
                Thread.sleep(1000L);
                synchronized (second) {
                    System.out.println(this.getName() + "obtained:" + second);
                }
            } catch (InterruptedException e) {

            }
        }
    }
    public static void main(String[] args) throws InterruptedException{
        String lockA = "lockA";
        String lockB = "lockB";
        DeadLockSample t1 = new DeadLockSample("Thread1", lockA, lockB);
        DeadLockSample t2 = new DeadLockSample("Thread2", lockB, lockA);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
    }

}
```

如上代码，编译运行后打印结果:

Thread2obtained:lockB

Thread1obtained:lockA

### 方法一：使用 jstack 命令

1. 使 用 jps 命令定位 Java 进程 PID

   ```sh
   1760 DeadLockSample
   1829 Jps
   1608 org.eclipse.equinox.launcher_1.5.300.v20190213-1655.jar
   ```

2. 使用 jstack 定位，jstack 1760

### 方法二：使用 Java API，ThreadMXBean 提供的 findDeadThreads()

```java
 public static void main(String[] args) throws InterruptedException{
        ThreadMXBean mxBean = ManagementFactory.getThreadMXBean();
        Runnable dlCheck = new Runnable() {
            @Override
            public void run() {
                long[] threads = mxBean.findDeadlockedThreads();
                if(threads!=null){
                    ThreadInfo[] threadInfos = mxBean.getThreadInfo(threads);
                    System.out.println("Dectected deadlock threads:");
                    for (ThreadInfo threadInfo : threadInfos){
                        System.out.println(threadInfo.getThreadName());
                    }
                }
            }
        };
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);
        scheduledExecutorService.scheduleAtFixedRate(dlCheck,5L,10L, TimeUnit.SECONDS);
    }
```

## 思考题

有时候并不是阻塞导致的死锁，只是某个线程进入了死循环，导致其他线程一直等待，这种问题如何诊断呢？

某个线程进入死循环导致其他线程一直等待，可以看作是自旋锁的表现。

如何诊断？当死循环引起的其他线程阻塞，会导致 CPU 使用率飙升，在 Linux 系统中使用 top 命令配合 grep Java 之类，找到忙的 PID；再使用 jstack 命令查看线程具体情况；最后定位代码。

# Java并发包提供了哪些并发工具类？

在 java.util.concurrent 及其子包下提供了 Java 并发的各种基础工具类。

- 提供了比 synchronized 更加高级的同步结构，包括 CountDownLatch、CycliBarrier、Semaphore 等，可以实现更加丰富的多线程操作。比如：利用 Semaphore 作为资源调度器，限制同时进行工作的线程数量。

- 线程安全容器，ConcurrentHashMap、有序的 ConcurrentSkipListMap 或者通过类似快照机制实现线程安全的动态数组 CopyOnWriteArrayList 等。

- 并发队列实现，比如各种 BlockedQueue 实现，比较典型的 ArrayBlockingQueue、SynchorousQueue 或针对特殊场景的 PriorityBlockingQueue.

- Eexcutor 框架，可创建不同类型的线程池，调度任务运行。

## 知识扩展

### 同步结构

- CountDownLatch，允许一个或多个线程等待某个操作完成。

- CycliBarrier，辅助型同步结构，允许多个线程等待达到某个屏障。

- Semaphore，Java 版本的信号量实现。

  Semaphore 通过控制一定数量的允许（permit）的方式，来达到限制通用资源访问的目的。生活例子：在车站，当空车就位时，为了避免过度拥挤，调度员指挥人员排队等待乘车，一个批次允许10 个人上车，等这10个人上车后，再放下一批次乘车。

### Semaphore

```java
package concurrent;

import java.util.concurrent.Semaphore;

public class UsualSemaphoreSample {
    public static void main(String[] args){
        System.out.println("Action...Go!");
        Semaphore semaphore = new Semaphore(5);
        for(int i = 0 ; i < 10;i++){
            Thread thread = new Thread(new SemaphoreWorker(semaphore));
            thread.start();
        }
    }
}
class SemaphoreWorker implements Runnable{
    private String name;
    private Semaphore semaphore;

    public SemaphoreWorker(Semaphore semaphore) {
        this.semaphore = semaphore;
    }

    @Override
    public void run() {
        try {
            log("is waiting for a permit!");
            semaphore.acquire();
            log("acquired a permit!");
            log("executed!");
        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            log("released a permit");
            semaphore.release();
        }
    }
    public void log(String msg){
        if(name==null){
            name = Thread.currentThread().getName();
        }
        System.out.println(name+"\t"+msg);
    }
}
```

```java
import java.util.concurrent.Semaphore;

public class AbnormalSemaphoreSample {
    public static void main(String[] args) throws  InterruptedException{
        Semaphore semaphore = new Semaphore(0);
        for(int i = 0; i < 10 ; i++){
            Thread thread = new Thread(new MyWorker(semaphore));
            thread.start();
        }
        System.out.println("Action...Go!");
        semaphore.release(5);
        System.out.println("Wait for permits off");
        while (semaphore.availablePermits()!=0){
            Thread.sleep(100L);
        }
        System.out.println("Action...Go again");
        semaphore.release(5);
    }
}
class MyWorker implements Runnable{
    private Semaphore semaphore;

    public MyWorker(Semaphore semaphore) {
        this.semaphore = semaphore;
    }

    @Override
    public void run() {
        try {
            semaphore.acquire();
            System.out.println("Executed");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}
```

### CountDownLatch & CyclicBarrier

- CountDownLatch 是不可重置的，无法重用；而 CyclicBarrier 没有这种限制，可以重用。

- CountDownLatch 的基本操作组合是 countDown / await。调用 await 的线程阻塞等待 countDown 足够的次数，不论你是在一个线程还是多个线程里 countDown ，只要次数足够即可。CountDownLatch 操作的是事件。

- CyclicBarrier 的基本操作组合，则就是 await，当所有的伙伴（parties）都调用了 await，才会继续进行任务，并自动进行重置。**注意，正常情况下，CyclicBarrier 的重置都是自动发生的**，如果我们调用 reset 方法，但还有线程在等待，就会导致等待线程被打扰，抛出 BrokenBarrierException 异常。**CyclicBarrier 侧重点是线程**，而不是调用事件，它的典型应用场景是用来等待并发线程结束。

```java
package concurrent;

import java.util.concurrent.CountDownLatch;

public class LatchSample {
    public static void main(String[] args) throws InterruptedException{
        CountDownLatch latch = new CountDownLatch(6);
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(new FirstBatchWorker(latch));
            thread.start();
        }
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(new SecondBatchWorker(latch));
            thread.start();
        }
        while (latch.getCount() !=1){
            Thread.sleep(100L);
        }
        System.out.println("Wait for first batch finish");
        latch.countDown();
    }
}
class FirstBatchWorker implements Runnable{
    private CountDownLatch latch;

    public FirstBatchWorker(CountDownLatch latch) {
        this.latch = latch;
    }

    @Override
    public void run() {
        System.out.println("First batch executed!");
        latch.countDown();
    }
}

class SecondBatchWorker implements Runnable{
    private CountDownLatch latch ;
    public SecondBatchWorker(CountDownLatch latch) {
        this.latch = latch;
    }

    @Override
    public void run() {
        try {
            latch.await();
            System.out.println("Second batch executed!");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}
```

```java
package concurrent;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierSample {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, new Runnable() {
            @Override
            public void run() {
                System.out.println("Action...Go again!");
            }
        });
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(new CyclicBarrierWorker(cyclicBarrier));
            thread.start();
        }
    }
}
class CyclicBarrierWorker implements Runnable{
    private CyclicBarrier cyclicBarrier;

    public CyclicBarrierWorker(CyclicBarrier cyclicBarrier) {
        this.cyclicBarrier = cyclicBarrier;
    }

    @Override
    public void run() {
        try{
            for (int i = 0; i < 3; i++) {
                System.out.println("Executed");
                cyclicBarrier.await();
            }
        }catch (BrokenBarrierException e){
            e.printStackTrace();
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}
```

![Java核心技术-进阶篇二\并发包提供的线程安全集合](Java核心技术-进阶篇二\并发包提供的线程安全集合.png)

## 小结

CountDownLatch 和 CyclicBarrier 都能实现线程之间的等待，主要的区别如下：

- CountDownLatch 用于某个线程等待其他线程执行完任务之后，它才执行；不可重用

- CyclicBarrier 用于一组线程相互等待至某个状态，然后这一组线程在同时执行；可重用

# ConcurrentLinkedQueue 和 inkedBlockingQueue有什么区别？

线程安全队列

[Lock-Free]([https://www.cnblogs.com/gaochundong/p/lock_free_programming.html](https://www.cnblogs.com/gaochundong/p/lock_free_programming.html)

- Concurrent 类型基于 lock-free ，多线程访问场景中多有应用，一般可以提高吞吐量。

- LinkdBlockingQueue 内部是基于锁，并提供 BlockingQueue 的等待特性方法。LinkedBlockingQueue 改进了锁操作的粒度，头、尾操作使用不同的锁，通用场景下，吞吐能力相对更好些。

java.util.concurrent 包提供的容器 Queue、List、Set、Map，从命名上可以区分为 Concurrent*、CopyOnWrite 、Blocking 等三类，同样的线程安全容器，可以简单的认为：

- Concurrent 类型没有类是 CopyOnWrite 之类容器相对较重的修改开销。

- Concurrent 提供了比较低的遍历一致性（弱一致性），比如，利用迭代器遍历时，如果容器发生修改，迭代器任然可以继续进行遍历。

- 与弱一致性相对应的，就是同步容器常见的行为 “fail-fast”，也就是监测到容器在遍历过程中发生了修改，则抛出 ConcurrentModificationException ，不再继续遍历。

- 弱一致性的另一个体现是，size 等操作准确性是有限的，未必 100% 正确。

- 读取的性能也具有一定的不确定性。

## 面试考察

- 哪些队列是有界（Bounded），哪些队列是无界的（Unbounded）？

- 如何选择合适的队列实现？

- 常见的线程安全队列如何实现的，并进行了哪些改进提高性能表现？

## 知识扩展

![Java核心技术-进阶篇二\线程安全队列](Java核心技术-进阶篇二\线程安全队列.png)

ConcurrentLinkedDeque 和 LinkedBlockingDeque，Deque 侧重点是支持队列头尾都进行插入和删除，所以提供了特定的方法：

- 尾部插入需要 addLast(e)、offerLast(e)

- 尾部删除需要 removeLast()、pollLast()

大部分 Queue 都实现了 BlockingQueue 接口，在常规队列操作基础上，**Blocking 意味着其提供了特定的等待性操作**，获取时 take 等待元素进入队列，或者插入是 put 等待队列出现空位。

```java
 /**
 * 获取并移除队列头结点，如果必要，其会等待直到队列出现元素
…
 */
E take() throws InterruptedException;

/**
 * 插入元素，如果队列已满，则等待直到队列出现空闲空间
   …
 */
void put(E e) throws InterruptedException;  

```

### BlockingQueue 是否有界（Bounded，Unbouded）?

- ArrayBlockingQueue 典型的有界队列，内部以 final 数组保存数据，数组大小决定了队列的边界，在创建 ArrayBlockingQueue 时，要指定容量大小。

- LinkedBlockingQueue，容易被误解为无边界，但行文和内部代码都是基于有界的逻辑实现的，如果在创建队列时没有指定容量，那容量限制将自动被设置为 Integer.MAX_VALUE，成为了无界队列。

- SynchronousQueue，说是非常奇葩的队列实现，每个操作都要等待插入操作，反之每个插入操作也要等待删除动作。其内部容量是（0）

- PriorityBlockingQueue 无边界的优先队列，严格意义上讲，其大小总归是要受到系统资源的影响。

- DelayedQueue 和 LinkedTransformQueue 同样是无边界队列。对于无边界的队列，有个自然结果，就是 put 操作永远不会发生其他 BlockingQueue 那种等待情况。

### 队列使用场景和典型用例

Queue 被广泛用在生产者-消费者场景，比如利用 BlockingQueue 的实现。

```java
package sample.queue;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class ConsumerProducerBlockingQueue {
    public static final String EXIT_MSG = "Good Bye!";

    public static void main(String[] args) {
        BlockingQueue<String> queue = new ArrayBlockingQueue<>(3);
        Producer producer = new Producer(queue);
        Consumer consumer = new Consumer(queue);
        new Thread(producer).start();
        new Thread(consumer).start();

    }

    static class Producer implements Runnable{
        private BlockingQueue<String> queue;
        public Producer(BlockingQueue<String> queue) {
            this.queue = queue;
        }

        @Override
        public void run() {
            for (int i = 0; i < 20; i++) {
                try {
                    Thread.sleep(5L);
                    String msg = "Message"+i;
                    System.out.println("Producer new item\t"+msg);
                    queue.put(msg);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            try {
                System.out.println("Time to say good by!");
                queue.put(EXIT_MSG);
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }
    static class Consumer implements Runnable{
        private BlockingQueue<String> queue;
        public Consumer(BlockingQueue<String> queue) {
            this.queue = queue;
        }

        @Override
        public void run() {
            try {
                String msg;
                while (!EXIT_MSG.equalsIgnoreCase((msg= queue.take()))){
                    System.out.println("Consumed item\t"+msg);
                    Thread.sleep(10L);
                }
                System.out.println("Got exit message bye!");
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }
}

```

### 日常开发中如何选择？

以 LinkedBlockingQueue 、 ArrayBlockingQueue、SynchronousQueue 为例。

- 队列边界的需求，ArrayBlockingQueue 有明确的容量限制；LinkedBlockingQueue 则取决于创建时是否制定容量；SynchronousQueue 则干脆不能缓存任何元素。

- 空间利用角度，数组结构的 ArrayBlockingQueue 比 LinkedBlockingQueue 紧凑，因为不需要创建所谓的节点，但是其初始分配阶段就需要一段连续的空间，所以初始内存需求更大。

- 通用场景中，LinkedBlockingQueue 吞吐量优于 ArrayBlockingQueue ，因为它实现了更加细粒度的锁操作。

- ArrayBlockingQueue 实现简单，性能更好预测。

- 如果需要实现两个线程间接力行（handoff）的场景，可以选择 CountDownLatch，但是 SynchronousQueue 也是完美符合这种场景的，而且线程协调和数据传输统一起来，代码更加规范。

- 也许是意外，很多时候 SynchronousQueue 的性能表现，大大超过了其他实现，尤其是在队列元素较小的场景。

## 小结

有界：ArrayBlockingQueue、SynchronousQueue、

无界：PriorityBlockingQueue、DelayedQueue、LinkedTransferQueue

可有界可无界的属于 LinkedBlockingQueue，取决于创建时是否制定容量。

BlockingQueue 基本都是基于锁实现。

Concurrent 类型基于 lock-free 实现。

LinkedBlockingQueue 改进了锁操作的粒度，头、尾操作使用了不同的锁，通用场景下，吞吐量相对更好一些。

# Java并发类库提供的线程池有哪几种？有什么特点？

一个线程不能调用两次 start() 方法，也就是说线程不能够重复启动，线程的创建和销毁存在一定的性能开销。

线程池技术用来提供系统资源利用效率，简化线程管理。



通常我们都是利用 Executors 提供的通用线程池创建方法，去创建不同配置的线程池，主要区别在于不同的 ExecutorService 类型或者不同的初始参数。

Executors 目前提供了 5 种不同的线程池创建配置：

- newCachedThreadPool() ,它是一种用来处理大量短时间工作任务的线程池，主要特点：试图缓存线程并重用，当无缓存线程可用时就会创建新的工作线程；若线程闲置 60 秒，则被移除缓存；长时间闲置，这种线程池不会消耗什么资源。内部使用 SynchronousQueue 作为工作队列。

- newFixedThreadPool(int nThreads)，重用指定数据（nThreads） 的线程，其背后使用无界的工作队列，任何时候最多有 nThreads 个工作线程事活动的。如果任务数量大于活动队列数目，将在工作队列中等待空闲线程出现；如果有工作线程退出，将会有新的工作线程被创建，以补足指定的数据 nThreads。

- newSingleThreadExecutor()，将线程数据被限制为 1 ，操作一个无界的工作队列，所以它保证了所有任务的都是被顺序执行，最多会有一个任务处理活动状态，并且不允许使用者改动线程池实例，因此可避免其改变线程数目。

- newSingleThreadScheduledExecutor() 和 newSingleScheduledThreadPool(int corePoolSize)，创建的是个 ScheduledExecutorService，可以进行定时或周期性的工作调度，区别在于单一工作线程还是多个工作线程。

- newWorkStealingPool(int parallelism（Java 8 新增方法）内部会构建 ForkJoinPool 利用 Work-Stealing 算法，并行处理任务，不保证处理顺序。

## Executor 框架主要内容

- 掌握 Executor 框架主要内容，组成与职责，掌握基本开发用例中的使用。

- 对线程池和相关并发工具类型的理解，甚至是源码层面的掌握。

- 实践中有哪些常见问题，基本诊断思路。

- 如何根据自身应用特点合理使用线程池。

![Java核心技术-进阶篇二\Executor 框架基本组成](Java核心技术-进阶篇二\Executor 框架基本组成.png)

- Executor 作为基础接口，其初衷是将任务提交和任务执行细节解耦。

- ExecutorService 不仅提供 service 的管理功能，比如 shutdown 等方法，也提供更加全面的任务机制，如 Future 而不是 void 的 submit 放啊。

- Java 标准类库提供几种基础实现，比如 ThreadPollExecutor、ScheduledThreadPoolExecutor、ForkJoinPool，这些线程池的设计特点在于其高度可调节行和灵活性。

- Executors 则从简化使用的角度，提供各种快捷方便的静态工厂方法。

![Java核心技术-进阶篇二\应用与线程池交互和内部工作过程](Java核心技术-进阶篇二\应用与线程池交互和内部工作过程.png)

- 工作队列，负责存储用户提交的各个任务，这个工作队列，可以是容量为 0 的 SynchronousQueue（使用 newCachedThreadPool），也可以是固定大小线程池（newFixedThreadPool）那样使用 LinkedBlockingQueue。

  `private final BlockingQueue<Runnable> workQueue;`

- 内部的“线程池”，这是指保持工作线程的集合，线程池需要在运行过程中管理线程创建、销毁。例如，对于带缓存的线程池，当任务压力较大时，线程池会创建新的工作线程；当业务压力退去，线程池会在闲置一段时间（默认 60 秒）后结束线程。

  `private final HashSet<Worker> workers = new HashSet<>();`

线程池的工作线程被抽象为静态内部类 Worker ,基于 AQS 实现。

- ThreadFactory 提供上面所需要的创建线程逻辑。

- 如果任务提交时被拒绝，比如线程池已经处于 SHUTDOWN 状态，需要为其提供处理逻辑，Java 标准库提供了类似ThreadPoolExecutor.AbortPolicy 等默认实现，也可以按照实际需求自定义。

### 线程池的基本组成部分（体现在线程池的构造函数中）：

- corePoolSize ，核心线程数（长期驻留线程数，除非设置 allowCoreThreadTimeOut）。对于不同的线程池，该值的区别可能会很大，比如 newFixedThreadPool 会将其设置为 nThreads ，而对于 newCacheThreadPool 则是 0 。

- maximumPoolSize，顾名思义，就是线程不够时能够创建的最大线程数。同样进行对比，对于 newFixedThreadPool，当然就是 nThreads，因为其要求是固定大小，而 newCachedThreadPool 则是 Integer.MAX_VALUE。

- keepAliveTime 和 TimeUnit，这两个参数指定了额外的线程能够闲置多久，显然有些线程池不需要它。

- workQueue，工作队列，必须是 BlockingQueue。

```java
public ThreadPoolExecutor(int corePoolSize,
                      	int maximumPoolSize,
                      	long keepAliveTime,
                      	TimeUnit unit,
                      	BlockingQueue<Runnable> workQueue,
                      	ThreadFactory threadFactory,
                      	RejectedExecutionHandler handler)


```

### 线程池的生命周期，他的状态是如何表征的呢？

![Java核心技术-进阶篇二\线程生命周期](Java核心技术-进阶篇二\线程生命周期.png)

### 线程池大小选择策略

- 如果我们的任务主要是进行计算，那么就意味着 CPU 的处理能力是稀缺的资源，我们能够通过大量增加线程数提高计算能力吗？往往是不能的，如果线程太多，反倒可能导致大量的上下文切换开销。所以，这种情况下，通常建议按照 CPU 核的数目 N 或者 N+1。

- 如果是需要较多等待的任务，例如 I/O 操作比较多，可以参考 Brain Goetz 推荐的计算方法：

  ```
  线程数 = CPU 核数 × 目标 CPU 利用率 ×（1 + 平均等待时间 / 平均工作时间）
  ```

- 上面是仅仅考虑了 CPU 等限制，实际还可能受各种系统资源限制影响，例如我最近就在 Mac OS X 上遇到了大负载时ephemeral 端口受限的情况。当然，我是通过扩大可用端口范围解决的，如果我们不能调整资源的容量，那么就只能限制工作线程的数目了。这里的资源可以是文件句柄、内存等。

## 小结

任需加固和实践来消化这波知识点。



# AtomicInteger底层实现原理是什么？如何在自己的产品代码中应用CAS操作？

并发包的组成，同步结构、线程池等，基于什么原来来设计和实现的？

AtomicInteger 是对 int 类型的一个封装，提供原子性的访问和更新操作，其原子性性操作的实现是基于 CAS（compare-and-swap）

CAS 是一系列操作的集合，获取当前数值，进行一些运算，利用 CAS 指令试图进行更新。如果当前值未改变，代表没有其他线程进行并发修改，则成功更新。否则，可能出现不同的选择，要么进行重试，要么返回一个成功或失败的结果。

AtomicInteger 依赖 Unsafe 提供的一些底层能力，进行底层操作；以 volatile 的 value 字段，记录数值，保证可见性。



对 ReentrantLock、CyclicBarrier 等并发结构底层的实现技术的理解。



[java.util.concurrent.atomic]([https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/atomic/package-summary.html](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/atomic/package-summary.html) 提供了常用的原子性数据类型、甚至是引用、数组等相关原子类型和更新操作工具，是很多线程安全程序的首选。



## 小结

对这一讲的理解难度很大啊先 mark 下

### CAS 是什么？compare-and-swap（比较并替换）

CAS 是 Java 并发中所谓 lock-free 机制的基础。

CAS 需要 3 个操作数：内存地址 V，旧的预期值 A，即将要更新的目标值 B。

CAS 指令执行时，当且仅当内存地址 V  的值与预期值相等时，将内存地址 V 的值修改为 B，否则什么都不做。整个 CAS 操作是个原子操作。

了解就够了：CAS 底层是如何实现的？这依赖于 CPU 提供的特定指令，具体根据体系结构的不同存在明显的区别。

### AQS（AbstractQueueSynchronoizer）

AQS（AbstractQueueSynchronoizer）实现各种同步结构和部分其他组成单元（如线程池中的 Worker） 的基础。

为什么需要 AQS？如何使用它？AQS 实现原理。


