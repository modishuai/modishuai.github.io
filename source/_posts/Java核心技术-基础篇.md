---
title: Java 核心技术-基础篇
date: 2019-03-10 22:59:22
categories:
- Java
tags:
- Java
---

> 来自极客时间，《Java 核心技术36讲》整理笔记。

# 谈谈你对Java平台的理解？

Java 是众多面向对象编程语言中多其中一种，其显著的两个特点就是**“跨平台”** 和**“垃圾收集”**。

跨平台是指**“书写一次，到处执行”** 与 JVM 的存在密不可分；垃圾收集是指 Java 通过垃圾收集器回收分配内存，大部分情况下，程序员无需关心内存的分配和回收，更多的关注到技术编程上面。

 JRE（Java Runtime Enviroment）:  Java 运行环境，包含 JVM 和 Java 类库和模块等。

 JDK（Java Development Kit）: JDK 是 JRE 的超级，包含很多工具，比如编译器、诊断工具等；

## Java 是解释执行吗？

Java 源代码（*.java）先是通过 javac 编译成为字节码（bytecode）然后在运行时通过 Java 虚拟机（JVM）内嵌的解释器将字节码转换成为对应的机器码。常见的 JVM 多数使用 Oracle JDK 提供的 Hotspot JVM，提供的 JIT（Just-In-Time）动态编译器，JIT 能够在运行时将热点代码编译成机器码，这种情况下部分热点代码就属于**“编译执行”**，而不是解释执行。

Java 分为编译期和运行时：编译期是将 Java 源代码通过 javac 编译成字节码；运行时 JVM 通过类加载器（Class-Loader）加载字节码，解释或者编译执行。Oracle Hotspot JVM 内置了两个不同的 JIT compiler， C1 对应 client 模式，适用于启动速度敏感的桌面应用；C2 对应 server 模式，为长时间运行的服务端应用设计。默认采用分层编译（TieredCompilation）。

Java 虚拟机启动时，可手动指定不同参数对运行模式选择。

 "-Xint" 开启解释执行，关闭编译执行。

“-Xcomp” 表示开启编译执行，关闭解释执行。

参数 “-Xint”，抛弃了可能带来的性能优势（解释器（interpreter） 是逐条读入，逐条解释运行。）“-Xcomp” 会导致 JVM 启动缓慢；

JDK 9 引入新的编译方式即 AOT（Ahead-of-Time Compilation）直接将字节码编译成机器代码，避免了 JIT 预热等方面的开销。

## 小结

Java 特性

平台无关性：依赖 JVM 完成二次编译，将 .class 编译成适合操作系统的机器码。

面向对象：封装、继承、多态、抽象。

垃圾回收（GC机制）：JVM 内存结构，堆、栈、方法区。

语言特性：泛型、lambda、Stream、类型推断

基础类库：集合、IO/NIO、网络、并发、安全

常见垃圾收集器：SerialGC、ParallerGC、CMS、G1

# Exception 和 Error有什么区别？

Exception 和 Error 都是Throwable 的子类，只有 Throwable 类实例才能被抛出（throw）或则捕获（catch），它是异常处理机制的基本组成类型。

Exception 是程序正常运行中，可以预料的意外情况，可以进行捕获并处理。Error 是指在正常运行中，不太可能出现的情况，一旦出现就不可恢复，不能被捕获比如 OOM（OutOfMemoryError）

Exception 分**检查（checked）** 和 **未检查（unchecked）**异常，检查异常在源代码里必须显示的进行捕获处理，是编译器检查的一部分。未检查异常就是所谓的运行时异常（RuntimeException），比如 NullPointerException、ArrayIndexOutOfBoundsException 、 ClassCastException、DateTimeException、SystemException，通过编码避免逻辑错误，按需要来判断是否需要进行捕获处理，并不会在编译器强制要求。

## 知识扩展

![Java核心技术-基础篇\accba531a365e6ae39614ebfa3273900](Java核心技术-基础篇\accba531a365e6ae39614ebfa3273900.png)

### RuntimeException 与 Exception 有什么区别？

Exception 是 RuntimeException 的父类，所有继承RuntimeException 的子类都属于**不检查（unchecked）**异常即运行时异常。

Exception 定义的异常必须处理，RuntimeException 定义的异常可选择处理。

### NoClassDefFoundError 和 ClassNotFoundException 有什么区别？

NoClassDefFoundError 是指 JVM 或者 ClassLoader 实例尝试加载类的时候，去找不到类的定义。

ClassNotFoundException 是指试图通过类名称或全限定名加载对应的 class 时，找不到对应的 class  文件抛出的异常。 

### 异常处理基本原则

一、捕获相匹配的异常，避免捕获 Exception 类似的通用异常。

二、不要生吞（swallo）异常。

生吞异常，往往是基于假设这段代码可能不会发生，或者忽略异常。在产品代码中最好不要有这种假设，如果没把异常抛出来或输出到日志文件，后续没法判断异常出现在那里。

printStackTrace()，也不要出现在产品代码中，尤其是分布式系统，如果发生异常，就无法找到堆栈轨迹（StackTrace）

### 从性能角度看 Java 的异常处理机制

- try-catch 代码会产生额外的性能开销，影响 JVM 对代码进行优化，建议仅捕获必要的代码段，尽量不要一个try 包住一大堆代码；也不建议使用异常控制代码流程，还不如选择（if/else、switch）

- Java 实例化一个 Exception 都会对当前堆进行快照，这是一个相对比较重的操作。若发生非常频繁，这个开销就不能被忽略了。

### 异常处理流程

![Java核心技术-基础篇\异常处理流程](Java核心技术-基础篇\异常处理流程.png)

## 小结

捕获匹配异常，不要直接捕获 Exception

不要生吞异常，避免在分布式系统和生产系统用printStackTrace() 。

Checked Exception 不兼容 function 编程

try-catch 捕获必要代码，尽量避免 JVM 对于代码进行优化，不要使用异常来控制代码流程

try-catch 也是性能优化的考察点。

# 谈谈final、finally、finalize 有什么不同？

final 用来修饰类表示不可继承；用来修饰方法表示不可重写；用来修饰变量表示不可修改，只是引用不可在被赋值。

finally 是 Java 用来保证代码一定要被执行的一种机制。使用 try-finally 或者 try-catch-finally 来保证连接或资源关闭、保证 unlock 锁等动作。

finalize 是基础类 java.lang.Object 的一个方法，最初的设计目的是用来保证对象在被垃圾收集前完成特定资源回收。不再推荐，JDK 9 已经标记为 deprecated。

对于 finally 的使用有些偏门的情况：

```java
try{
    System.exit(1);
}finally{
    System.out.println("print from finally!");
}
```

以上代码中的 finally 模块中的代码就不会执行。

## 知识扩展

final 不是 immutable

```java
 final List<String> strList = new ArrayList<>();
 strList.add("Hello");
 strList.add("world");  
 List<String> unmodifiableStrList = List.of("hello", "world");
 unmodifiableStrList.add("again");
```

fianl 只能约束 strList 引用不能够被赋值，但 strList 的行为不受 final 影响，添加元素操作是正常的。

如果需要对象本身不可变，需要相应的类支持不可变行为，在上面例子中 List.of()  方法创建的本身就是不可变的List，最后的添加操作在运行时将抛出异常。

实现 immutable 的类，需要做到以下几点：

- 将 class 声明为 final

- 将所有成员定义为 private 和 final，不要实现 setter 方法。

- 通常构造对象时，成员变量使用深度拷贝来初始化，而非直接赋值，这是一种防御措施，因为你无法确定输入对象不被其他人修改。

- 如果确实需要实现 setter 方法，或者其他可能返回内部状态的方法，使用 copy-on-write 原则，创建私有 copy。

的

java 平台逐步使用 java.lang.ref.Cleaner 来替换原有的 finalize 方法。Cleaner 的实现利用了幻想引用也称虚引用（PhantomReference）。利用幻想引用和引用队列，我们可以保证对象被彻底销毁前做一些资源回收工作。每个 Cleaner 的操作都是独立的，有自己的运行线程，所以可以避免以为死锁问题。

## 小结

推荐使用 final 关键字来明确代码的语义，也是保证平台安全的必要手段，数据的一致性。

finally 总是执行，特殊情况除外，比如程序或线程被中断情况。

对于关闭连接或资源，推荐使用 Java 7 添加的 try-with-resource 语句。

finallize 并不能真正完成资源回收可参考《Java 编程思想》第四版。

Java 平台逐步使用 java.lang.ref.Cleaner 来替换原有的 finalize 方法。

# 强引用、软引用、弱引用、幻象引用有什么区别？

不同的引用类型，主要体现的是**对象不同的可达性（Reachable）状态和垃圾收集的影响。**

## 强引用（StrongReference）

```java
Object obj = new Object();   //  强引用
obj = null; // 帮助垃圾收集器回收。
```

常见的普通对象引用，只要强引用指向一个对象，就表明对象还“活着”，垃圾收集器不会回收它。

如果没有其他引用关系，或超出引用的作用域或者显示地将相应强引用赋值为 null， 就可以被垃圾收集。至于什么时候回收要看垃圾收集策略。

## 软引用（SoftReference）

```java
 String str = new String("HelloWorld"); // 强引用

 SoftReference<String> softRef=new SoftReference<String>(str);     // 软引用
```

软引用是相对强引用弱化了些的引用，只有 JVM 认为内存空间不足是，垃圾收集器才会回收它。

软引用通常用来实现内存敏感的缓存，如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了缓存的同时，不会耗尽内存。

软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾收集器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。

## 弱引用（WeakReference)

```java
String str = new String("HelloWorld");    

WeakReference<String> strWeakRef = new WeakReference<String>(str);

str = null;
```

弱引用并不能使对象豁免垃圾收集，仅仅是提供一种访问弱引用状态下对象的途径。如果试图获取时对象还在，就使用它，否则重现实例化。同样也是很多缓存实现的选择。

垃圾收集器线程一旦发现具有弱引用的对象，无论内存空间是否足够，都要回收它。由于垃圾回收器是一个优先级很低的线程，因此不一定很快就发现了它。

## 虚引用（PhantomReference）

虚引用（也称幻想引用），形同虚设，并不会决定对象的生命周期。如果一个对象持有虚引用和没有任何引用是一样的，任何时候都可能被垃圾收集器回收。

虚引用主要用来跟踪对象被垃圾收集状态，可以通过检查与虚引用关联的引用队列中是否已经包含指定的虚引用，从而来判断虚引用引用的对象是否将被回收。

虚引用、幻象引用都是说的同一个，仅仅是提供了一种确保对象被 finalize 以后，做某些事情的机制。比如：通常用来做 Post-Mortem 清理机制；也有人利用幻象引用监控对象的创建和销毁。

## 知识扩展

### 对象可达性状态流转分析

![Java核心技术-基础篇\36d3c7b158eda9421ef32463cb4d4fb0](Java核心技术-基础篇\36d3c7b158eda9421ef32463cb4d4fb0.png)

### 

这是 Java 定义的不同可达级别（reachability level），具体如下：

- 强可达（Strongly Reachable）,当一个对象可以有一个或多个线程可以不通过各种引用访问到的情况。比如，我们创建一个对象，那么创建他的线程对它就是强可达。

- 软可达（Softly Reachable），当我们只能通过软引用才能访问到对象的状态。

- 弱可达（Weakly Reachable），无法通过强引用或弱引用访问，只能通过弱引用访问时的状态。这是十分临近 finalize 状态时机，当弱引用被清除的时候，就符合 finalize 的条件了。

- 幻象可达（Phantom Reachable），就是没有强、软、弱引用关联，并且 finalize 过了，只有幻象引用指向这个对象的时候。

- 不可达（unreachable），意味着对象被清除了。

# String、StringBuffer、StringBuilder

String 是 Immutable 类，类和属性的声明都是 fianl 。对字符串的操作拼接、分割等都会产生新的 String 对象。

StringBuffer 是为解决 String 拼接操作产生太多的中间对象而提供的。StringBuffer 是线程安全的，有额外的性能开销。

StringBuilder 是 Java1.5 提供，与 StringBuffer 作用一样，不同的是为了减少额外的性能开销去掉了线程安全。

## 知识扩展

### 字符串设计

StringBuffer 和 StringBuilder 底层是利用可修改的（char，JDK9 以后是 byte）数组，二者继承了 AbstractStringBuilder，区别仅在于最终方法是否加入了 synchronized

```java
public class StringConcat {
     public static String concat(String str) {
       return str + “aa” + “bb”;
     }
}
```

以上代码在 JDK1.8 编译后采用 StringBuilder 替代

### 字符串缓存

String 在 Java6 提供了 intern() 方法，目的是提示 JVM 把相应字符串缓存起来，便于重复使用。调用 intern() 方法，如果有缓存的字符串就直接返回缓存中的实例，否则就将其缓存起来。使用 new 创建的 String 对象需要手动调用 intern() 方法进行缓存。

历史版本 Java 6 中不推荐 大量使用 intern()？是因为被缓存的字符串是存在 PermGen （永久代），此空间容量有限，基本上不会被 FullGC 之外的垃圾收集。所以使用不当就会造成 OOM。

后续版本，这个缓存被放在了堆中，如此一来就避免了永久代占满的问题，甚至永久代在 Java 8 中被 MetaSpace（元数据）替代了。默认的缓存大小也在不断的扩大中。使用如下代码打印查看

```bash
-XX:+PrintStringTableStatistics
```

# 动态代理是基于什么原理？

反射机制，可以直接操作类或者对象。比如：获取对象的类定义，类声明的属性和方法，调用方法或者构造对象，甚至可以运行时修改类定义。

动态代理，一种方便运行时动态构建代理、动态处理代理方法调用的机制。比如：AOP 面向切面编程等。

实现动态代理的方式：jdk Proxy、ASM、cglig（基于 ASM）、Javassist

## 知识扩展

### 反射机制及其演进

Java 反射机制，java.lang 和 java.lang.reflect 包。利用 Class、Field、Method、Constructor 操作类和对象的元数据对应。值得注意的是反射提供了 **AccessibleObject.setAccessible（boolean flag）**它的子类也都重写了该方法，accessible 可以绕过 public 、protected、private 成员访问限制。 

Java 9 对其进行了改进 ，原因是 Jigsaw 项目新增的模块化系统，出于强封装的考虑，对反射访问进行了限制。 Jigsaw 引入了 Open 概念，只有当被反射操作的模块和指定的包对反射调用者模块 Open，才能使用 setAccessible；否则被认为是不合法的操作。

### 动态代理

通过代理可与实现调用者与实践者之间解耦。比如 RPC 调用，框架内部的寻址、序列化、反序列化，对于调用者没得太大意义，动过代理，提供更加友善的界面。

Jdk 自身的动态代理是以接口为中心才能生成代理对象。

### jdk Proxy 优势？

jdk 本身就支持；最小化依赖关系，减少依赖意味着简化开发和维护；平滑升级

### cglib 优势？

没有接口实现类，实现接口是有侵入性质。

### 小结

静态代理：需要手动或借助工具来生成代理类，缺点是每个业务类都需要一个代理类，缺乏灵活性。

动态代理：运行时自动生成代理对象，缺点是生成代理对象和调用代理方法需要额外时间。

JDK 动态代理：基于 Java 反射机制实现，必须实现接口的业务类才能生成代理对象。

cglib 动态代理：基于 ASM 实现，通过生成业务类子类作为代理类。

# int和Integer有什么区别？

Java 的 8 个原始数据类型：long、double、float、int、byte、short、char、boolean

Integer 是 int 的包装类，Java 5 中引入自动装箱和自动拆箱工功能，Java 可以根据上下文自动进行转换。

Integer 值缓存，Java 5 中新增静态工厂方法 valueOf，在调用时利用一个缓存机制，带来性能改进，这个值默认缓存范围是 -128 到 127 之间。

## 知识扩展

### 理解自动装箱、拆箱

自动装箱算是一种语法糖。 Java 平台为我们自动进行了一些转换，保证不同的写法在运行时等价，发生在编译阶段，也就是生成的字节码是一致的。

javac 把装箱替换成 Integer.valueOf() ，把拆箱替换成 Integer.intValue() 。

**原则上建议避免无意中的装箱和拆箱行为，特别是性能敏感场景。**创建 10 万个 Java 对象和 10 万个整数的开销可不是一个数量级的，不管是内存使用还是处理速度，光是对象头的空间占用就已经是数量级的差距了。

Java 对象保存在内存中由三部分组成：对象头、实例数据、对齐填充字节；

对象头的组成：Mark Word 、指向类的指针、数组长度（对象数组才有）：

计算对象大小可以利用：jol、jmap 或者 instrument api（java agent）等；

### 源码分析

#### 缓存机制

```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

Integer 缓存策略仅在自动装箱（autoboxing）时候有用，使用构造器创建的 Integer 对象不能不缓存。Integer.valueOf() 就是缓存执行的地方。

Integer 缓存范围是 -128 到 127 ,特殊场景也可以通过 JVM 启动参数 `-XX:AutoBoxCacheMax=N` 调整缓存上限值。

其他包装类的缓存机制：

- Boolean，缓存了 true/false 对应实例，确切说，只会返回两个常量实例 Boolean.TRUE/FALSE。
- Short，同样是缓存了 -128 到 127 之间的数值。
- Byte，数值有限，所以全部都被缓存。
- Character，缓存范围’\u0000’ 到 ‘\u007F’。

Integer 同样是不可变类型。

Integer 等包装类定义了类似 SIZE 和 BYTES 常量,这种设计的上的考虑了开发人员无需关心 32 位和 64 位平台上的数据位数差异。

#### Java 原始数据类型和引用类型局限性

- 原始数据类型并不能和泛型配合使用。Java 中的泛型，完全是一种编译阶段使用的技巧，Java 编译器会自动将类型转换为对应的特定类型，这就决定了使用泛型，必须保证相应类型必须转换成 Object。
- 无法高效的表达数据，也不便于表达复杂的数据结构，如 vector 和 tuple

  我们知道 Java 的对象都是引用类型，如果是一个原始数据类型数组，它在内存里是一段连续的内存，而对象数组则不然，数据存储的是引用，对象往往是分散地存储在堆的不同位置。这种设计虽然带来了极大灵活性，但是也导致了数据操作的低效，尤其是无法充分利用现代 CPU 缓存机制。

# 对比Vector、ArrayList、LinkedList有何区别？

Vector、ArrayList、LinkedList 都实现了集合框架中的 List（有序集合），都是按照位置进行定位、添加、删除操作、提供迭代器遍历内容等；**Vector 是 Java 早起提供的线程安全的动态数组**，Vector 内部使用对象数组保存数据，可更具需要自动增加容量，当数组满时，则创建新的数组，并拷贝原有的数组数据。

ArrayList 是应用更加广泛的**动态数组**实现，非线程安全，性能好。与Vector 的区别在于扩容，Vector 在扩容是提高 1 倍，ArrayList 则是增加 50%；

LinkedList 是 Java 提供的双向链表，非线程安全，不需要像上面两个那样扩容。

Vector 和 ArrayLIst 作为动态数组，很适合随机访问的场景，除了尾部插入和删除元素，性能较差，比如在中间插入一个元素，需要移动后续所有元素。LinkedList 弥补了前两个在插入和删除上的性能差，但是随机访问性能慢。

## 问题延伸

- Java 集合框架设计结构
- Java 容器（集合 & Map）类型，了解或掌握对应的数据结构、算法、思考具体技术选择。
- 将问题延伸到并发和性能领域。
- 集合框架的演进与发展。

## 知识扩展

排序算法

内部排序：归并排序、交换排序（冒泡、快排）、选择排序、插入排序等；

外部排序：利用内存和外部存储处理超大数据集

Java 集合框架

![Java核心技术-基础篇\java集合](Java核心技术-基础篇\java集合.png)

Coolection 是所有集合的根，扩展开提供三大类集合：

- List，有序集合

- Set，不允许元素重复

- Queue/Deque，Java 提供的标准队列结构的实现，除了集合的基本功能，还支持类似先入先出（FIFO，First-in-First-Out）或后进先出（LIFO，Last-in-First-Out）

每种集合的通用逻辑，都被抽象到了抽象类之中。比如：AbastractList。

TreeSet 实际上是利用 TreeMap 实现的；HashSet 也是基于 HashMap 来实现的；

Set

- TreeSet，支持自然顺序访问，但添加、删除、包含等操作相对低效。

- HashSet，利用哈希算法，如果哈希散列正常，可以提供常数时间的添加、删除、包含等操作，不保证有序。

- LinkedHashSet，内部构件一个记录插入顺序的双向链表，提供按照插入顺序遍历的能力，也保证常数时间的添加、删除、包含操作，性能略低于 HashSet，需要维护链表的开销。

- 遍历元素时 HashSet 性能受自身容量影响，所以初始化的时候，除非有必要，不然不要将其背后的 HashMap 容量设置过大。而 LinkedhashSet 由其内部链表方式，遍历性能和元素多少有关系。

以上的集合类都不是线程安全的，不像  java.util.concurrent 包内提供的线程安全容器，但是在 java.util.Collections 类中却提供了一系列 synchronized 方法。

```java
static <T> List<T> synchronizedList(List<T> list)
```

利用类似的方法来实现线程安全集合：

```java
List list = Collections.synchronizedList(new ArrayList());
```

**Java 提供的默认排序算法，具体是什么排序方式以及设计思路？**

这个问题本身就是有点陷阱的意味，因为需要区分是 Arrays.sort() 还是 Collections.sort() （底层是调用 Arrays.sort()）；什么数据类型；多大的数据集（太小的数据集，复杂排序是没必要的，Java 会直接进行二分插入排序）等。

- 对于原始数据类型，目前使用的是所谓双轴快速排序（Dual-Pivot QuickSort），是一种改进的快速排序算法，早期版本是相对传统的快速排序。

- 而对于对象数据类型，目前则是使用[TimSort](http://hg.openjdk.java.net/jdk/jdk/file/26ac622a4cab/src/java.base/share/classes/java/util/TimSort.java)，思想上也是一种归并和二分插入排序（binarySort）结合的优化排序算法。TimSort 并不是 Java 的独创，简单说它的思路是查找数据集中已经排好序的分区（这里叫 run），然后合并这些分区来达到排序的目的。

Java 8 引入了并行排序算法（直接使用 parallelSort 方法），利用现代多核处理器的计算能力，底层基于 fork-join 框架实现，当数据量增长到百万级别是，提供就很大，取决于处理器和系统环境。

Java 9 中新增了一系列静态工厂方法：比如 List.of()、Set.of() 用来简化构建容器较小容器实例的代码量。

# Hashtable、HashMap、TreeMap有什么不同？

三者都实现了 Map 接口也都是以**键值对**形式进行存储和操作数据的容器类型。

HashTable 是 Java 类库早起提供的哈希表实现，不支持键值为 null ，其本身是同步的，所以有额外的性能开销。

HashMap 也是基于哈希表的实现，行为和HashTable 一直，主要区别在于 HashMap 不是同步的，允许键值为 null。

TreeMap 是基于红黑树的一种提供顺序访问的 Map ，与 HashMap 不同它的 get 、pug 、remove 操作都是 O（long(n))）的时间复杂度，具体的顺序可以有指定的 Comparator 来决定，或者根据键的自然顺序来判断。

## 知识扩展

- 理解 Map 相关类似整体结构，尤其是有序数据结构的一些要点。

- 从源码去分析 HashMap 的设计和实现要点，理解容量、负载影因子等，为什么需要这些参数，如何影响 Map 的性能，实践中如何取舍等。

- 理解树化改造的相关原理和改进原因。

### Map 整体结构

Map 虽然包含在 Java 集合框架中，但本身并不是狭义的集合类型。

![Java核心技术-基础篇\Map结构](Java核心技术-基础篇\Map结构.png)

HashTable 比较特殊，作为类似 Vector 、 Stack 的早期集合相关类型，它继承至 Dictionary 类，类结构上和 HashMap 明显不同。

HashMap、EnumMap、TreeMap 继承至 AbstractMap 类，包含了通用的操作方法。

大部分使用 Map 场景通常都是插入、访问、删除而且对顺序没有要求的 HashMap 作为首选。

hashMap 的性能表现非常依赖于哈希码的有效性，必须掌握 hashCode 和 equals 的一些基本约定。

- equals 相等，hashCode 一定要相等。

- 重写 hashCode 也要重写 equals。

- hashCode 需要保持一致性，状态改变返回的哈希码要保持一致。

- equals 的对称、反射、传递等特性。

LinkedHashMap 和 TreeMap 都可以保证某种顺序，但二者还是非常不同的。

- LinkedHashMap 通常提供遍历顺序符合插入顺序，它的实现是通过为键值对维护一个双向链表。注意，通过特定构造函数，我们可以创建反映访问顺序的实例，所谓的 put、get、compute 等，都算作“访问”。

```java
import java.util.LinkedHashMap;
import java.util.Map;  
public class LinkedHashMapSample {
    public static void main(String[] args) {
        LinkedHashMap<String, String> accessOrderedMap = new LinkedHashMap<String, String>(16, 0.75F, true){
            @Override
            protected boolean removeEldestEntry(Map.Entry<String, String> eldest) { // 实现自定义删除策略，否则行为就和普遍 Map 没有区别
                return size() > 3;
            }
        };
        accessOrderedMap.put("Project1", "Valhalla");
        accessOrderedMap.put("Project2", "Panama");
        accessOrderedMap.put("Project3", "Loom");
        accessOrderedMap.forEach( (k,v) -> {
            System.out.println(k +":" + v);
        });
        // 模拟访问
        accessOrderedMap.get("Project2");
        accessOrderedMap.get("Project2");
        accessOrderedMap.get("Project3");
        System.out.println("Iterate over should be not affected:");
        accessOrderedMap.forEach( (k,v) -> {
            System.out.println(k +":" + v);
        });
        // 触发删除
        accessOrderedMap.put("Project4", "Mission Control");
        System.out.println("Oldest entry should be removed:");
        accessOrderedMap.forEach( (k,v) -> {// 遍历顺序不变
            System.out.println(k +":" + v);
        });
    }
}
```

- TreeMap ，整体顺序是由键的顺序关系决定的，通过 Comparator 或 Comparable（自然顺序）来决定。

### HashMap 源码分析

- HashMap 内部实现基本点分析

- 容量（capacity）和负载因子（load factor）

- 树化

HashMap 内部结构可以看做是数组（Node<K,V>[] talbe）和链表结合组成的复合结构，数组被分为一个个桶（bucket），通过哈希值决定了键值对这个数组的寻址；哈希值相同的键值对，则以链表形式存储；参考如下示意图。需要注意的是，如果链表大小超过阈值（TREEIFY_THRESHOLD,8）,图中的链表就会被改造成树形结构。

![Java核心技术-基础篇\桶数组](Java核心技术-基础篇\桶数组.png)

从非拷贝构造函数的实现来看，这个表格（数组）似乎并没有在最初就初始化好，仅仅设置了一些初始值而已。

```java
public HashMap(int initialCapacity, float loadFactor){  
    // ... 
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

所以，我们深刻怀疑，HashMap 也许是按照 lazy-load 原则，在首次使用时被初始化（拷贝构造函数除外，我这里仅介绍最通用的场景）。既然如此，我们去看看 put 方法实现，似乎只有一个 putVal 的调用：

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

看来主要的密码似乎藏在 putVal 里面，到底有什么秘密呢？为了节省空间，我这里只截取了 putVal 比较关键的几部分。

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbent,
               boolean evit) {
    Node<K,V>[] tab; Node<K,V> p; int , i;
    if ((tab = table) == null || (n = tab.length) = 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == ull)
        tab[i] = newNode(hash, key, value, nll);
    else {
        // ...
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for first 
           treeifyBin(tab, hash);
        //  ... 
     }
}
```

- 如果表格是 null，resize 方法会负责初始化它，这从 tab = resize() 可以看出。

- resize 方法兼顾两个职责，创建初始存储表格，或者在容量不满足需求的时候，进行扩容（resize）。

- 在放置新的键值对的过程中，如果发生下面条件，就会发生扩容。

```java
if (++size > threshold)
    resize();
```

具体键值对在哈希表中的位置（数组 index）取决于下面的位运算：

```java
i = (n - 1) & hash
```

仔细观察哈希值的源头，我们会发现，它并不是 key 本身的 hashCode，而是来自于 HashMap 内部的另外一个 hash 方法。注意，为什么这里需要将高位数据移位到低位进行异或运算呢？**这是因为有些数据计算出的哈希值差异主要在高位，而 HashMap 里的哈希寻址是忽略容量以上的高位的，那么这种处理就可以有效避免类似情况下的哈希碰撞。**

```java
static final int hash(Object kye) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>>16;
}
```

HashMap resice()方法

```java
final Node<K,V>[] resize() {
    // ...
    else if ((newCap = oldCap << 1) < MAXIMUM_CAPACIY &&
                oldCap >= DEFAULT_INITIAL_CAPAITY)
        newThr = oldThr << 1; // double there
       // ... 
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {  
        // zero initial threshold signifies using defaultsfults
        newCap = DEFAULT_INITIAL_CAPAITY;
        newThr = (int)(DEFAULT_LOAD_ATOR* DEFAULT_INITIAL_CAPACITY；
    }
    if (newThr ==0) {
        float ft = (float)newCap * loadFator;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?(int)ft : Integer.MAX_VALUE);
    }
    threshold = neThr;
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newap];
    table = n；
    // 移动到新的数组结构 e 数组结构 
   }
```

依据 resize 源码，不考虑极端情况（容量理论最大极限由 MAXIMUM_CAPACITY 指定，数值为 1<<30，也就是 2 的 30 次方），我们可以归纳为：

- 门限值等于（负载因子） x （容量），在构建 HashMap 没有指定它们，那就是依据相应的默认常量值。

- 门限值通常是以倍数进行调整（newThr = oldThr << 1）,根据 putVal 中的逻辑，当元素个数超过门限大小时则调整 Map 大小。

- 扩容后，需要将老的数组的元素重新放置到新的数组，这是扩容的一个组要开销来源。

### 容量、负载因子和树化

**为什么容量和负载因子这么重要？**因为容量和负载因子决定可用桶的数量，空桶多了浪费空间，若使用得太满则严重影响操作性能。极端情况下，假设只有一个桶，那么它就退化成了链表，完全不能提供所谓常数时间存的性能。

实践中如何选择呢？预估` 负载因子 * 容量 > 元素数量`

负载因子的建议：

- 没有特殊需求不建议改动， JDK 自身默认的负载因子满足了通用场景。

- 实在要调整，建议不要超过 0.75 的数值，因为会显著增加冲突，降低 HashMap 的性能。

- 若使用太小的负载因子，按上面的公式，预设容量值也要进行调整，否则可能会导致更加频繁的扩容。

### 树化改造

树化改造，对应逻辑主要在 putVal 和 treeifyBin 中。

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        // 树化改造逻辑
    }
}
```

- 如果容量小于 MIN_TREEIFY_CAPACITY，只会进行简单的扩容。

- 如果容量大于 MIN_TREEIFY_CAPACITY ，则会进行树化改造。

为什么HashMap 要树化呢？本质上是个安全问题。因为在元素的放置过程中，如果一个对象哈希冲突，都被放置到同一个桶里，则会形成链表，链表是线性查询，会严重影响存取的性能。
