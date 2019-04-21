---
title: Java 核心技术-性能基础篇
date: 2019-04-14 17:00:00
categories:
- Java
tags:
- Java
---
> 性能调优工具、方法、基础

# 后台服务运行"变慢"，诊断思路？

## 诊断问题

- 服务是突然变慢还是长时间运行后变慢？类似问题是否重复出现？

- “慢” 的定义是什么？可以理解为系统对其他方面的请求反应延时变长吗？

- 对于分布式系统和大型单体应用有着不同的诊断思路？

## 对症下药

对于分布式系统的诊断主要还是借助于**系统日志、性能监控系统**，Java 诊断工具 JFR（Java Flight Recorder），监控应用是否出现某种类型异常。如果没有异常，还可以查看系统级别资源，监控 CPU 、内存等；

监控 Java 服务自身，例如 GC 日志里面是否观察到 Full GC 等恶劣情况出现，或者是否 Minor GC 在变长等；利用 jstat 工具，获取内存使用的统计信息；利用 jstack 工具检查死锁情况；

## 性能分析方法论

- 自上而下，从应用顶层，逐步深入到具体的不同模块，找到可能的问题和解决办法。

- 自下而上，从硬件底层，类似 CPU 判断 Cache-Miss 之类的问题和调优机会，出发点是指令级别优化。（专业的性能工程师）

### 自上而下的分析思路和应用工具

#### 系统性能分析

系统性能分析中 CPU、内存和 IO 是主要关注项

Linux 系统中先用 top命令查看 CPU 负载情况。

![Java核心技术-性能基础篇\top](Java核心技术-性能基础篇\top.png)

可以看到 load average（平均负载）三个值（1分钟，5分钟，15分钟）非常低，暂时没有升高迹象。如果数值非常高，并且短期平均值高于长期平均值，则表明负载很重；

利用 top 命令获取相应 pid，“H” 表示 thread 模式，可以配合 grep 命令精准定位。

`top h`

然后转成 16 进制

printf "%x" your_pid

利用 jstack 获取线程栈，对比相应的 ID

更加通用的诊断方向利用 vmstat 之类，查看上下文切换数量，比如指定时间间隔为 1 收集 10 次

`vmstat -1 -10`

iostat 工具诊断磁盘健康情况。

#### JVM 性能分析

- JMC、JConsole 等工具进行运行时监控。

- 利用各种工具，在运行时进行堆转储分析，或者获取各种角度的统计数据（如 jstat -gcutil 分析 gc 、内存分带）

- GC 日志等手段，诊断 Full GC、Minor GC，或者引用堆积等。

# Lambda 真的能让Java程序变慢吗？

> 本篇重点探讨 Java 开发者，如何从代码级别判断应用的性能表现，重点理解[基准测试]([https://baike.baidu.com/item/%E5%9F%BA%E5%87%86%E6%B5%8B%E8%AF%95/5876292?fr=aladdin](https://baike.baidu.com/item/%E5%9F%BA%E5%87%86%E6%B5%8B%E8%AF%95/5876292?fr=aladdin)（Benchmark）

**使用基准测试判断应用表现的性能必须明确定义自身的范围和目标，否则有可能产生误导的结果。**

## Lambda/Stream 局限性

Lambda / Stream 提供了强大的函数式编程能力。

- Lambda / Stream 提供了与传统方式接近对等的性能，对于性能敏感的场景，就不能不关注它了。例如：初始化的开销，Lambda 并不算是语法糖，而是一种新的工作机制，在首次调用时，JVM 需要为其构建 [CallSite](https://docs.oracle.com/javase/8/docs/api/java/lang/invoke/CallSite.html) 实例。这意味着 Java 应用启动过程引入很多 Lambda 语句，会导致启动过程变慢。其实现特点决定了 JVM 对它的优化可能与传统方式存在差异。

- 增加了程序诊断等方面的复杂性，程序栈复杂很多，Fluent 风格本身不算是对调试非常友好的结构，并且在检查异常的处理方面也存在局限性等。

## 如何利用主流框架构建简单的基准测试

### 什么时候开发微基准测试？

当需要对一个大型软件中的某一小部分的性能评估时，就可以开发微基准测试。

### 微基准测试框架选型？

[JMH](http://openjdk.java.net/projects/code-tools/jmh/) 由 Hotspot JVM 团队专家开发，除了支持完整的基准测试过程，包括预热、运行、统计和报告等，还支持 Java 和其他 JVM 语言。它针对 Hotspot JVM 提供了各种特性，以保证基准测试的正确性，整体准确性优于其他框架。

Maven 工程

![Java核心技术-性能基础篇\JMH](Java核心技术-性能基础篇\JMH.png)

```powershell
$ mvn archetype:generate \
          -DinteractiveMode=false \
          -DarchetypeGroupId=org.openjdk.jmh \
          -DarchetypeArtifactId=jmh-java-benchmark-archetype \
          -DgroupId=org.sample \
          -DartifactId=test \
          -Dversion=1.0
```

利用注解（Annotation）定义具体的测试方法，以及为基准测试详细配置。

示例：利用@Benchmark 标识基准测试方法，@BenchmarkMode 标识基准测试模式（如吞吐量、平局时间）等模式。

```java
@Benchmark
@BenchmarkMode(Mode.Throughput)
public void testMethod() {
   // Put your benchmark code here.
}
```

当实现具体的测试后，利用 Maven 命令构建

```powershell
mvn clean install
```

运行基准测试

```powershell
java -jar target/benchmarks.jar
```

## 如何保证基准测试的正确性。

[微基准测试中的典型问题](https://www.ibm.com/developerworks/java/library/j-jtp02225/)

- 保证代码经过了足够并且合适的预热。默认情况，在 server 模式下，JIT 会在一段代码执行 10000 次后，将其编译为本地代码，client 模式则是 1500 次以后。我们需要排除代码执行初期的噪音，保证真正采样到的统计数据符合其稳定运行状态。

```java
-XX:+PrintCompilation

我这里建议考虑另外加上一个参数，否则 JVM 将默认开启后台编译，也就是在其他线程进行，可能导致输出的信息有些混淆。
-Xbatch
```

- 防止 JVM 进行无效代码消除。例如下面的代码片段中，由于我们并没有使用计算结果 mul，那么 JVM 就可能直接判断无效代码，根本就不执行它

```java
public void testMethod() {
   int left = 10;
   int right = 100;
   int mul = left * right;
}
// 如果你发现代码统计数据发生了数量级程度上的提高，需要警惕是否出现了无效代码消除的问题。
// 解决办法也很直接，尽量保证方法有返回值，而不是 void 方法，或者使用 JMH 提供的

public void testMethod(Blackhole blackhole) {
   // …
   blackhole.consume(mul);
}
```

- 防止发生常量折叠。JVM 如果发现计算过程是依赖于常量或者事实上的常量，就可能会直接计算其结果，所以基准测试并不能真实反映代码执行的性能。JMH 提供了 State 机制来解决这个问题，将本地变量修改为 State 对象信息，请参考下面示例

```java
@State(Scope.Thread)
public static class MyState {
   public int left = 10;
   public int right = 100;
}

public void testMethod(MyState state, Blackhole blackhole) {
   int left = state.left;
   int right = state.right;
   int mul = left * right;
   blackhole.consume(mul);
}
```

- 另外 JMH 还会对 State 对象进行额外的处理，以尽量消除伪共享（[false-sharing](https://blogs.oracle.com/dave/java-contended-annotation-to-help-reduce-false-sharing)）的影响，标记 @State，JMH 会自动进行补齐。

- 如果你希望确定方法内联（Inlining）对性能的影响，可以考虑打开下面的选项。

```powershell
-XX:+PrintInlining
```

## 盲区

### 什么是常量折叠？

常量折叠是指 Java 在编译阶段做的一个优化，也就是说在编译阶段就把表达式计算好，不需要在运行时计算。例如：

```java
//折叠前
int mul = 5 * 5; 
//折叠后
int mul = 25;
```

# JVM 优化 Java 代码都做了些什么？

JVM 对代码执行的优化分为：

- 运行时（runtime）优化，主要是解释执行和动态编译通用的一些机制，比如锁机制（偏斜锁）、内存分配机制（TLAB），除此以外还有一些专门用于优化解释执行效率的，比如说模板解释器、内联缓存（inline cache，用于优化虚方法调用的动态绑定）。

- 即时编译器（JIT）优化，将热点代码以方法为单位转换成机器代码，直接运行在底层硬件之上。采用了多种优化方式，包括静态编译器可以使用的方法内联、逃逸分析、也包括基于程序运行 profile 的投机性优化（可以理解为，有一条 instanceof 指令，在编译之前的执行过程中，测试对象的类一直是同一个，那么即时编译器可以假设编译之后的执行过程还是这个类，并且根据这个类直接返回 instanceof 的结果。如果出现其他类，那就抛弃这段编译后的机器代码，并切换到解释执行）。

Java 代码的生命周期

![Java核心技术-性能基础篇\Java 生命周期](Java核心技术-性能基础篇\Java 生命周期.png)

java 通过字节码屏蔽不同的硬件差异，JVM 负责完成字节码到机器码的转换。

javac 等编译器或者相关 API 等将源码转换成字节码的过程中也会进行少量类似常量折叠之类的优化。

javac 优化与 JVM 内部优化也存在关联，毕竟它负责字节码的生成。例如，Java 9 中的字符串拼接，会被 javac 替换成对 StringConcatFactory 的调用，进而为 JVM 进行字符串拼接优化提供了统一的入口。在实际场景中，还可以通过不同的[策略](http://openjdk.java.net/jeps/280)来干预这个过程。

## JVM 运行时优化

![Java核心技术-性能基础篇\JVM 运行时优化](Java核心技术-性能基础篇\JVM 运行时优化.png)

JVM 会根据统计信息，动态决定什么方法被编译，什么方法解释执行，即使是已经编译过的代码，也可能在不同的运行阶段不再是热点，JVM 有必要将这种代码从 Code Cache 中移除出去，毕竟其大小是有限的。

## 探查JVM 优化具体发生情况？

- 打印编译发生细节 `-XX:+PrintCompilation`

- 输出更多编译细节 `-XX:UnlockDiagnosticVMOptions -XX:+LogCompilation -XX:LogFile=<your_file_path>`

- 打印内联的发生，可利用下面的诊断选项，也需要明确解锁。 `-XX:+PrintInlining`

- 如何知晓 Code Cache 的使用状态呢？利用 JMC、JConsole 、NMT 等工具。

## 应用开发者的调优手段

- 调整热点代码门限值，`-XX:CompileThreshold=N`，server 模式默认 10000 次 ，client 模式默认 1500 次。

- 调整 Code Cache 大小，`-XX:ReservedCodeCacheSize=<SIZE>` 或调整其初始大小`-XX:InitialCodeCacheSize=<SIZE>`

- 调整编译器线程数`-XX:CICompilerCount=N`，或选择适当的编译模式

## 盲点

### 逃逸分析

参考维基百科[Escape_analysis](https://en.wikipedia.org/wiki/Escape_analysis)

### 循环开展

参考维基百科[Loop_unrolling](https://en.wikipedia.org/wiki/Loop_unrolling)

# MySQL 支持事务隔离级别，以及悲观锁和乐观锁的原理和应用场景？

## 隔离级别

MySQL InnoDB 引擎是基于 MVCC（Multi Versioning Concurrency Control） 和 锁的复合实现。按照隔离程度从低到高，MySQL 事务隔离级别分为四个不同层次。

- 读未提交（Read uncommitted）允许脏读，也就是事务B能够看到事务A尚未提交的修改。

- 读已提交（Read committed），事务能够看到的数据都是其他事务已经提交的修改，保证不会看到任何中间性状态，允许不可重复读和幻想度。

- 可重复读（Repeatable reads），保证同一个事务中多次读取的数据是一致的，MySql InnoDB 默认的隔离级别。

- 串行化（Serializable），并发事务之间是串行化的，意味着读取需要获取共享读锁，更行需要获取排他写锁，如果 SQL 使用 WHERE 语句，还要获取区间锁（MySQL 以 GAP 锁形式实现，可重复读级别中默认也会使用）

## 悲观锁和客观锁

主要的区别体现在操作共享数据时：“悲观锁”认为数据出现冲突可能性更大，“乐观锁”则认为大部分情况不会出现冲突，进而决定是否采用排他性措施。

反映到 MySQL 数据库应用中：

悲观锁一般是利用类是 SELECT ...... FOR UPDATE 这样的语句，对数据加锁，避免其他事务意外修改数据。

乐观锁 与 Java 并发包中的 AtomicFieldUpdate 类是，也是利用 CAS 机制，并不会对数据加锁，而是通过对比数据的时间戳或者版本号，来实现乐观锁需要的版本判断。

# Spring 生命周期和作用域？

## Bean 的生命周期

![Java核心技术-性能基础篇\Spring Bean生命周期](Java核心技术-性能基础篇\Spring Bean生命周期.jpg)

1. Spring 对 bean 进行实例化。

2. Spring 将值和 bean 的引用注入到 bean 对应的属性中；

3. 若 bean 实现了 BeanNameAware接口，Spring 将 Bean 的 ID 传递给 setBeanName() 方法。

4. 若 bean 实现了 BeanFactoryAware 几口，Spring 将调用 setBeanFactory() 方法，将 BeanFactory 容器实例传入。

5. 若 bean 实现了 ApplicationContextAware 接口，Spring 将调用 setApplicationContext() 方法，将 bean 所在的应用上下文的引用传入进来。

6. 若 bean 实现了 BeanPostProcessor 接口，Spring 将调用 postProcessBeforeInitialization() 方法；

7. 若 bean 实现了 InitializingBean 接口，Spring 将调用 afterPropertiesSet() 方法。类似的，若 bean 使用 init-method 声明了初始化方法，该方法也会调用；

8. 若 bean 实现了 BeanPostProcessor 接口，Spring 将调用 postProcessAfterInitialization() 方法；

9. bean 准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到应用上下文被销毁；

10. 若 bean 实现了 DisposableBean 接口，Spring 将调用 destory() 方法。类似的若 bean 使用 destory-method 声明了销毁方法，该方法也会被调用。

## Bean 的作用域

- 单例（Singleton）默认选项，Spring 容器只创建一个实例。

- 原型（Prototype）每次注入或通过 Spring 上下文获取 bean 时候，都会创建一个 bean 实例。

- 会话（Session）Web 应用中使用，为每个会话创建一个 bean  实例。

- 请求（Request）Web 应用中使用，为每个请求创建一个 bean 实例。

- 全局（GlobalSession）用于 Portlet 容器，每个Portlet 有单独的 Session ，GlobalSession 提供一个全局的 HTTP Session.

## 控制反转

控制反转（Inversion of Control）也成为依赖注入（Dependency Injection），主要作用就是解耦合。

## 切面编程

切面编程AOP（Aspect Oriented Programming）,把遍布在应用各处的功能分离出来，让服务模块化

切面提供了取代继承和委托的另一种解决方案。

![Java核心技术-性能基础篇\Spring Advice](Java核心技术-性能基础篇\Spring Advice.png)

### 通知（Advice）

Advice 明确了切面编程中的什么，Spring AOP 提供 5 种类型通知

- Before（前置通知）在目标方法调用之前调用通知功能；

- After（后置通知）在目标方法完成之后调用通知，不关心方法输出是什么；

- After-returning（返回通知）在目标方法成功执行后调用通知；

- After-throwing（异常通知）在目标方法抛出异常后调用通知；

- Around（环绕通知）通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行的自定义行为；

### 连接点（join point）

join point 明确了切面编程中的目标。

### 切点（Ponitcut）

pointcut 明确了切面编程中的目标何处。

### 切面（Aspect）:

切面是 Advice 和 Ponitcut 的结合，明确了它是什么，在何时和何处完成其功能。

### 引入（Introduction）

引入允许我们想现有的类中添加新方法或属性。

### 织入（Weaving）

织入把切面应用到目标对象并创建新的代理对象的过程。

![AOP流程图](Java核心技术-性能基础篇/AOP流程图.jpg)

# 对比Java标准NIO类库，Netty是如何实现更高性能的吗？

- 更加优雅的 Reactor 模式实现、灵活的线程模型、利用 EventLoop 等创新性的机制，可以非常高效地管理成百上千的 Channel。

- 充分利用了 Java 的 Zero-Copy 机制，并且从多种角度，“斤斤计较”般的降低内存分配和回收的开销。例如，使用池化的 Direct Buffer 等技术，在提高 IO 性能的同时，减少了对象的创建和销毁；利用反射等技术直接操纵 SelectionKey，使用数组而不是 Java 容器等。

- 使用更多本地代码。例如，直接利用 JNI 调用 Open SSL 等方式，获得比 Java 内建 SSL 引擎更好的性能。

- 在通信协议、序列化等其他角度的优化。

## Netty

Netty 是一个异步且基于事件 Client/Server 的网络框架，目标是提供一种简单、快速构建网络应用的方式，通知保证高吞吐量、低延时、高可靠性。

Netty 的设计强调 关注点分离（Separation Of Concerns，SOC），通过精巧设计的事件机制，将业务逻辑和无关技术逻辑进行隔离，并通过各种方便的抽象，一定程度上填补了了基础平台和业务开发之间的鸿沟，更有利于在应用开发中普及业界的最佳实践。

**Netty > java.io + java.net**

在基础 NIO 之上，Netty 构建了更加易用、高性能的网络框架，广泛应用于互联网、游戏、电信等各种领域。

# 分布式ID的设计方案？Snowflake是否受冬令时切换影响？

## 分布式 ID 的基本要求

- 全局唯一，区别于单点系统的唯一，全局是要求分布式系统内唯一。

- 有序性，通常都需要保证生成的 ID 是有序递增的。例如，在数据库存储等场景中，有序 ID 便于确定数据位置，往往更加高效。

## 分布式 ID 的设计方案

- 数据库自增序列，简单易用，其扩展性和可靠性存在很大的局限性。

- Snowflake，TWitter 早起的开源的 Snowflake 实现，结构定义如下

  ![Java核心技术-性能基础篇\Snowflake](Java核心技术-性能基础篇\Snowflake.png)

  整体长度为 64（1 + 41 + 10 + 12 = 64）适合 Java 中 long 类型存储。

  头部是 1 位的正负标识位。

  41 位时间戳，通常使用System.currentTimeMills()

  10 位WorkerID，标准定义的是 5 位数据中心 + 5为机器ID，组成机器编号，便于区分不同的集群点。

  12 位序列，是单位毫秒内可生成的序列号数据的理论极限。

- Redis、Zookeeper、MongoDB 等中间件也都有各种唯一 ID 解决方案

Snowflake 算法的 Java 实现，依赖 System.currentTimeMillis() ,它是返回当前时间和 1970 年 1 月 1 号 UTC 时间相差的毫秒数，这个跟夏冬时令没关系，所以不受起影响。
