---
title: Java 核心技术-应用开发篇
date: 2019-03-31 16:40:00
categories:
- Java
tags:
- Java
- Java GC
- JVM
---

# 请介绍类加载过程，什么是双亲委派模型？

Java 类加载过程主要分为三个阶段：加载、链接、初始化。

- 加载（Loading），将 Java 字节码数据从不同的数据源读取到 JVM 中并映射为 JVM 认可的数据结构（Class 对象）。数据源可能的形态包括 jar 文件、class 文件、网络数据源等；如果输入数据不是 ClassFile 结构则抛出 ClassFormatError 错误。加载阶段是用户可参与的阶段，也就是说我们可以自定义类加载器，去实现自己的类加载过程。

- 链接（Linking），核心步骤，将原始的类定义信息平滑的转化入 JVM 运行的过程中。

  - 验证（Verification）这是虚拟机安全的重要保障，JVM 需要核验字节信息是符合 Java 虚拟机规范的，否则就被认为是 VerifyError，这样就防止了恶意信息或者不合规的信息危害 JVM 的运行，验证阶段有可能触发更多 class 的加载。

  - 准备（Preparation）创建类或接口中的静态变量，并初始化静态变量的初始值。但这里的“初始化”和下面的显式初始化阶段是有区别的，侧重点在于分配所需要的内存空间，不会去执行更进一步的 JVM 指令。

  - 解析（Resolution）在这一步会将常量池中的符号引用（symbolic reference）替换为直接引用。在[ Java 虚拟机]([https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4.3)规范 中，详细介绍了类、接口、方法和字段等各个方面的解析。

- 初始化（Initialization）执行类初始化的代码逻辑，包括静态字段赋值的动作，以及执行类定义中的静态初始化块内的逻辑，编译器在编译阶段就会把这部分逻辑整理好，父类型的初始化逻辑优先于当前类型的逻辑。

双亲委派模型，当类加载器（Class-Loader）试图加载某个类型的时候，先是委托给父加载器去完成，除非父加载器找不到相应类型，子加载器才会去加载。使用委派模型的目的是避免重复加载 Java 类型。

![Java核心技术-应用开发篇\类加载器](Java核心技术-应用开发篇\类加载器.png)

Java 9 ，由于 jigsaw 项目引入 Java 平台模块化系统（JPMS）,Java SE 源代码被划分为一系列模块。

所以很多类加载器，类文件容器都发生了重大变化。

## 双亲委派模型

- 什么是双亲委派模型，首先要知道什么是类加载器？

  当某个类加载器要加载某个类型时，先是将加载任务委托给父加载器去完成，依次递归，只有当父加载器找不到指定类时（ClassNotFoundException）,子加载器才会尝试自己加载。

- 为什么需要双亲委派模型？

  - 避免重复加载。

  - 避免恶意类的加载。

  - 使用双亲委派模型的好处在于Java 类随着他的类加载器一起具备了优先级的层次关系

# 有哪些方法可以在运行时动态生成一个Java类？

利用 Java 代码生成一段源码并保存到文件，接着借用 ProcessBuilder 之类启动 javac 进程，指定 Java 源文件进行编译，最后再利用类加载器，运行时加载即可。

不借用外部工具进行编译情况下，可以使用 [java.compiler ]([https://docs.oracle.com/javase/9/docs/api/javax/tools/package-summary.html](https://docs.oracle.com/javase/9/docs/api/javax/tools/package-summary.html)来编译源文件。

- 字节码和类加载到底是怎么无缝进行转换的？发生在整个类加载过程的哪一步？

- 如何利用字节码操纵技术，实现基本的动态代理逻辑？

- 除了动态代理，字节码操纵技术还有那些应用场景？

## 字节码 转换为 Class 对象

类从字节码到 Class 对象的转换，在类加载过程中是通过下面的方法提供的功能。

java.lang.ClassLoader

```java
 protected final Class<?> defineClass(String name, byte[] b, int off, int len)
        throws ClassFormatError{......}

 protected final Class<?> defineClass(String name, byte[] b, int off, int len,
                                         ProtectionDomain protectionDomain)
        throws ClassFormatError{......}

 protected final Class<?> defineClass(String name, java.nio.ByteBuffer b,
                                         ProtectionDomain protectionDomain)
        throws ClassFormatError{......}
```

```java
    private native Class<?> defineClass0(String name, byte[] b, int off, int len,
                                         ProtectionDomain pd);

    private native Class<?> defineClass1(String name, byte[] b, int off, int len,
                                         ProtectionDomain pd, String source);

    private native Class<?> defineClass2(String name, java.nio.ByteBuffer b,
                                         int off, int len, ProtectionDomain pd,
                                         String source);
```

## 字节码操纵逻辑

JDK 内部动态代理的逻辑，可以参考java.lang.reflect.ProxyGenerator的内部实现。我觉得可以认为这是种另类的字节码操纵技术，其利用了 DataOutputStrem 提供的能力，配合 hard-coded 的各种 JVM 指令实现方法，生成所需的字节码数组。你可以参考下面的示例代码。

```java
private void codeLocalLoadStore(int lvar, int opcode, int opcode_0,
                                DataOutputStream out)
    throws IOException
{
    assert lvar >= 0 && lvar <= 0xFFFF;
    // 根据变量数值，以不同格式，dump 操作码
    if (lvar <= 3) {
        out.writeByte(opcode_0 + lvar);
    } else if (lvar <= 0xFF) {
        out.writeByte(opcode);
        out.writeByte(lvar & 0xFF);
    } else {
        // 使用宽指令修饰符，如果变量索引不能用无符号 byte
        out.writeByte(opc_wide);
        out.writeByte(opcode);
        out.writeShort(lvar & 0xFFFF);
    }
}
```

## 实现一个动态代理的过程？

- 提供几个基础接口，作为被调用类型和代理类之间的统一入口。

- 实现 InvocationHandler 接口，对代理对象方法的调用，被分派到其 invoke 方法来真正实现动作。

- 通过 Proxy 类，调用 newProxyInstance 生成代理类实例。

## 动态代码的生成发生在那个阶段？

newProxyInstance 生成代理类实例的时候。

## 小结

本节有关动态代理技术及实现细节，常用的字节码操纵工具如 ASM、Javassit、cglib、JDK Proxy

Java 标准库也提供了与 javac 对等的编译器功能。

字节码操纵技术不仅仅用在动态代理，还应用在常见的ORM框架（MyBatis 用的javassit）、IOC容器、Mock 框架等；

# 谈谈JVM内存区域的划分，哪些区域可能发生OutOfMemoryError?

> [Java 虚拟机规范]([https://docs.oracle.com/javase/specs/jvms/se9/html/jvms-2.html#jvms-2.5](https://docs.oracle.com/javase/specs/jvms/se9/html/jvms-2.html#jvms-2.5)

## 运行时数据区(Run-Time Data Areas)

- 程序计数器（Program Counter Register）每个线程都有自己的程序计数器，并且任何时间一个线程都只有一个方法在执行，即该线程的当前方法。在 JVM 规范中规定，如果线程执行的是非native方法，则程序计数器中保存的是当前需要执行的指令的地址；如果线程执行的是native方法，则程序计数器中的值是undefined。

- Java 虚拟机栈（Java Virtual Machine Stack）也叫 Java 栈。每个线程在创建时都会创建一个虚拟机栈，其内部保存一个个的栈帧（Stack Frame），对应着一次次的 Java 方法调用。在一个时间点，对应的只会有一个活动的栈帧，通常叫作当前帧，方法所在的类叫作当前类。如果在该方法中调用了其他方法，对应的新的栈帧会被创建出来，成为新的当前帧，一直到它返回结果或者执行结束。JVM 直接对 Java 栈的操作只有两个，就是对栈帧的压栈和出栈。栈帧中包括局部变量表(Local Variables)、操作数栈(Operand Stack)、指向当前方法所属的类的运行时常量池的引用(Reference to runtime constant pool)、方法返回地址(Return Address)和一些额外的附加信息。

- 堆（Heap）Java 内存管理的核心区域，用来存放对象实例和数组。被所有线程共享，虚拟机启动时创建可以指定“Xmx”之类参数指定堆空间大小。堆也是垃圾收集器重点照顾的区域，所以堆内空间还会被不同的垃圾收集器进行进一步的细分，最有名的就是新生代、老年代的划分。

- 方法区（Method Area）用来存储元数据信息，类结构、运行时产量池、字段、方法代码等。被所有线程共享，虚拟机启动时创建。方法区逻辑上是堆的一部分，但是可以通过简单实现让垃圾收集器不碰它。由于早期的 Hotspot JVM 实现，很多人习惯于将方法区称为永久代（Permanent Generation）。Oracle JDK 8 中将永久代移除，同时增加了元数据区（Metaspace）。

- 运行时常量池（Run-Time Constant Pool）是方法区的一部分，每个运行时常量池都是从Java虚拟机的方法区域中分配的。当Java虚拟机创建类或接口时，将构造类或接口的运行时常量池。

- 本地方法栈（Native Method Stack）和 Java 虚拟机栈是非常相似的，支持对本地方法的调用，也是每个线程都会创建一个。在 Oracle Hotspot JVM 中，本地方法栈和 Java 虚拟机栈是在同一块儿区域，这完全取决于技术实现的决定，并未在规范中强制。

## 哪些区域可能发生OutOfMemoryError?

除了程序计数器以外，其他区域都可能发生 OutOfMemoryError。

## 总结

参考文献：[Java 内存区域划分](https://www.cnblogs.com/dolphin0520/p/3613043.html)

新生代和老年代的回收机制是什么？

《深入理解 Java 虚拟机》

# 如何监控和诊断JVM堆内和堆外内存使用？

> [JConsole 文档]([https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html](https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html)
> 
> [JRockit Mission Control 文档]([https://docs.oracle.com/cd/E15289_01/JRMCI/toc.htm](https://docs.oracle.com/cd/E15289_01/JRMCI/toc.htm)

监控及诊断 JVM 堆内和堆外内存可以使用 JConsole、VisualVM、JMC 图形化工具。

前面三种对于堆外内存中的直接内存不使用，但可以使用 JDK 自带的 Native Memory Tracking（NMT），它会从 JVM 本地内存分配的角度进行解读。

- 细化对各部分内存区域的理解，堆内结构是怎样的？如何通过参数调整。

- 堆外内存到底包括那些部分？具体大小受那些因素影响？

## 堆内部结构

![Java核心技术-应用开发篇\堆内结构](Java核心技术-应用开发篇\堆内结构.png)

关于 Virtual 区域：在 JVM 内部，如果 Xms 小于 Xmx，堆的大小并不会直接扩展到其上限，也就是说保留的空间（reserved）大于实际能够使用的空间（committed）。当内存需求不断增长的时候，JVM 会逐渐扩展新生代等区域的大小，所以 Virtual 区域代表的就是暂时不可用（uncommitted）的空间。

- 新生代

  大部分对象的创建和销毁都在新生代区域，绝大多数对象生命周期是很短暂的。其内部有分为 Eden 区域，作为对象初始化分配的区域；两个 Survivor，也叫 from、to 区域，被用来放置 Minor GC 中保留下来的对象。

  - JVM 会随意选一个 Survivor 区域作为“to”，然后会在 GC 过程中进行区域间拷贝，也就是将 Eden 中存活下来的对象和 from 区域的对象，拷贝到这个“to”区域。这种设计主要是为了防止内存的碎片化，并进一步清理无用对象。

  - 从内存模型的角度，又对 Eden 区域继续进行划分，Hotspot JVM 还有一个概念叫做 Thread Local Allocation Buffer（TLAB）。这是 JVM 为每个线程分配的一个私有缓存区域，否则，多线程同时分配内存时，为避免操作同一地址，可能需要使用加锁等机制，进而影响分配速度，你可以参考下面的示意图。从图中可以看出，TLAB 仍然在堆上，它是分配在 Eden 区域内的。其内部结构比较直观易懂，start、end 就是起始地址，top（指针）则表示已经分配到哪里了。所以我们分配新对象，JVM 就会移动 top，当 top 和 end 相遇时，即表示该缓存已满，JVM 会试图再从 Eden 里分配一块儿。

    ![Java核心技术-应用开发篇\Eden区域](Java核心技术-应用开发篇\Eden区域.png)

- 老年代

  老年代用来存放生命周期比较长的对象，也就是从 Survivor 区域拷贝过来的对象。特殊情况下，如果对象较大 JVM 会试图直接分配在 Eden 其他位置；如果对象太大，在新生代找不到足够长的连续空间，JVM 就会直接分配到老年代。

- 永久代

  早期 Hotspot JVM 的方法去实现方式，用来储存 Java 类元数据、常量池、Intern 字符串池，JDK 8 之后就不存在了。

## 通过 JVM 参数影响堆和内部区域大小。

- 最大堆体积 ，`-Xmx value`

- 初始最小的堆体积，`-Xms value`

- 老年代和新生代的比例，`-XX:NewRatio=value`，默认数值是 2 ，意味着老年代是新生代的 2 倍大；

- 指定具体的内存大小数值，`-XX:NewSize=value` 代替用比例的方式调整新生代的大小。

- Eden 和 Survivor 的大小是按照比例设置的，如果 SurvivorRatio 是 8，那么 Survivor 区域就是 Eden 的 1/8 大小，也就是新生代的 1/10，因为 YoungGen=Eden + 2*Survivor，JVM 参数格式是，`-XX:SurvivorRatio=value`

## 堆外内存

### 堆外内存分配方式

Java 分配堆外内存的两种方式如下：

- `java.nio.ByteBuffer.allocateDirect` 获得DirectByteBuffer 对象实例

  ```java
  public static ByteBuffer allocateDirect(int capacity) {
       return new DirectByteBuffer(capacity);
   }
  ```

  ```java
  // Primary constructor
      //
      DirectByteBuffer(int cap) {                   // package-private
          //1. 调用基类构造为字段赋值。
  
          super(-1, 0, cap, cap);
          boolean pa = VM.isDirectMemoryPageAligned();
          int ps = Bits.pageSize();
          long size = Math.max(1L, (long)cap + (pa ? ps : 0));
          //2. 预先分配内存
  
          Bits.reserveMemory(size, cap);
          //3. unsafe 分配内存。
  
          long base = 0;
          try {
              base = unsafe.allocateMemory(size);
          } catch (OutOfMemoryError x) {
              Bits.unreserveMemory(size, cap);
              throw x;
          }
          //4. 为已分配内存初始值为0
  
          unsafe.setMemory(base, size, (byte) 0);
          if (pa && (base % ps != 0)) {
              // Round up to page boundary
              address = base + ps - (base & (ps - 1));
          } else {
              address = base;
          }
          //5. 创建Cleaner 对象，用来释放堆外内存。
  
          cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
          att = null;  
      }
  ```

- `sun.misc.Unsafe.allocateMemory` 分配内存。

### 堆外内存适用场景

- 适合长生命周期存在的对象。

- 适合网络IO或外部文件读写。

- 堆外内存能有效避免因GC导致的暂停问题。

## 总结

掌握 Java 堆内结构以及监控工具的使用。

需要了解  [sun.misc.Unsafe](https://www.cnblogs.com/suxuan/p/4948608.html) 

# Java常见的垃圾收集器有哪些？

垃圾收集器（GC，Garbage Collector）和 JVM 紧密相关，不同厂商有不同版本的 JVM。

- 垃圾收集算法？如何判断一个对象是否可以回收？

- 垃圾收集器的工作流程？

## 多维度分析垃圾收集

- 按线程数目，可分为串行和并行垃圾收集器。在并行能力较强的 CPU 使用并行垃圾收集器可缩短应用程序停顿时间。

- 按工作模式，可分为并发式和独占式垃圾收集器。并发式垃圾收集器与应用程序交替工作，尽量减少应用停顿时间，独占式垃圾收集器（Stop-the-world）一旦运行，就停止应用程序中所有线程，知道垃圾收集工作完成。

- 按碎片处理方式，可分为压缩式和非压缩式垃圾收集器。压缩式垃圾收集器在工作完成之后，对存活对象进行压缩整理，消除收集后的碎片。

- 按工作的内存区域，可分为老年代和新生代垃圾收集器。

## 常见的垃圾收集器

- Serial GC，两个显著特点：第一，使用单线程进行垃圾收集；第二，它以独占的垃圾收集；进行垃圾收集工作时 Java 应用程序中的线程需要暂停，直到垃圾收集工作完成，也就是常说的（Stop-The-World）

  `-XX:+UseSerialGC`

  - 新生代串行收集器，采用复制（Copying）算法。

  - 老年代串行收集器，标记 - 压缩（Mark-Compact）算法；垃圾收集需要比新生代垃圾收集更长的时间。

- ParNew GC，新生代 GC 的实现，是 Serial GC 的多线程版本，常见的应用场景是配合老年代的 CMS GC 工作。

  `-XX:+UseConcMarkSweepGC -XX:+UseParNewGC`

- CMS（Concurrent Mark Sweep），主要关注减少系统停顿时间，采用标记-清除（Makr - Sweep）算法，使用多线程并发回收的垃圾收集器。CMS 主要工作步骤：初始标记、并发标记、重新标记、清除并发和并发重置。初始标记和重新标记是独占系统资源，并发标记和并发重置是可以和用户线程一起执行。采用标记-清除算法，将会产生大量内存碎片。由于CMS 和应用程序线程并发执行，也就意味着相互抢占 CPU ，工作期间对应用程序吞吐量造成影响。

- Parrallel GC，吞吐量优先的GC。并行收集器和串行收集器差不多一样，主要的区别在于多线程。在并发能力较强的 CPU 上产生的停顿时间更短，反之不如串行收集器。

  `-XX:+UseParallelGC` 开启并行回收器；`XX:ParallelGCThreads`指定并行收集器工作的线程数目，建议与 CPU 数量相当，避免过多的线程数影响垃圾收集性能。

  - 新生代并行（Parallel Scavenge）收集器，采用复制算法

  - 老年代并行收集器采用标记-压缩算法

- G1 GC，作为 JDK 9 中默认的GC，同时 CMS 并标记为废弃（deprecated）。G1 作为一款服务器的垃圾收集器，重点关注吞吐量和停顿。基于标记-压缩算法所以不会产生内存碎片。使用参数 `-XX:+UnlockExperimentalVMOptions –XX:+UseG1GC`来启用 G1 回收器，设置 G1 回收器的目标停顿时间：`-XX:MaxGCPauseMills=20,-XX:GCPauseIntervalMills=200`。

## 垃圾收集原理和基础概念

自动垃圾收集前需要明确哪些内存可以被释放，内存结构 和 类加载。

主要是两个方面，首先是存放在堆上的  Java 对象实例；其次是方法区中的元数据信息。

- 引用计数，引用计数的难题在于处理“循环引用”关系，所以 Java 垃圾收集器没有采用这种算法。

  引用计数器的实现很简单，对于一个对象 A，只要有任何一个对象引用了 A，则 A 的引用计数器就加 1，当引用失效时，引用计数器就减 1。只要对象 A 的引用计数器的值为 0，则对象 A 就不可能再被使用。

  所谓**循环引用**，对象 A 和对象 B，对象 A 中含有对象 B 的引用，对象 B 中含有对象 A 的引用。此时，对象 A 和对象 B 的引用计数器都不为 0。但是在系统中却不存在任何第 3 个对象引用了 A 或 B。也就是说，A 和 B 是应该被回收的垃圾对象，但由于垃圾对象间相互引用，从而使垃圾回收器无法识别，引起内存泄漏。

- 可达性分析，将对象及其引用关系看作一个图，选定活动的对象作为 GC Roots，然后跟踪引用链条，若一个对象和 GC Roots 之间不可达，也就是不存在引用链条，就可以认为是可回收的对象。JVM 会把虚拟机栈中和本地方法栈中长在引用的对象、静态属性引用的对象和常量，作为 GC Roots。

- 复制（Copying）将内存分为两块区域，每次只使用其中一块区域，进行垃圾回收时将活着的对象复制到另一块区域，复制过程将对象顺序放置（避免内存碎片化），然后清除正在使用的内存块中的所有对象，交换两个内存角色，完成垃圾回收。

  Java 的新生代串行垃圾回收器中使用了复制算法的思想。新生代分为 eden 空间、from 空间、to 空间 3 个部分。其中 from 空间和 to 空间可以视为用于复制的两块大小相同、地位相等，且可进行角色互换的空间块。from 和 to 空间也称为 survivor 空间，即幸存者空间，用于存放未被回收的对象。

  在垃圾回收时，eden 空间中的存活对象会被复制到未使用的 survivor 空间中 (假设是 to)，正在使用的 survivor 空间 (假设是 from) 中的年轻对象也会被复制到 to 空间中 (大对象，或者老年对象会直接进入老年带，如果 to 空间已满，则对象也会直接进入老年代)。此时，eden 空间和 from 空间中的剩余对象就是垃圾对象，可以直接清空，to 空间则存放此次回收后的存活对象。这种改进的复制算法既保证了空间的连续性，又避免了大量的内存空间浪费。

  新生代 GC 都是基于 复制算法。

- 标记 - 清除（Mark-Sweep）在标记阶段首先通过根节点，标记所有从根节点开始的较大对象。因此，未被标记的对象就是未被引用的垃圾对象。然后，在清除阶段，清除所有未被标记的对象。该算法最大的问题是存在大量的空间碎片，因为回收后的空间是不连续的。在对象的堆空间分配过程中，尤其是大对象的内存分配，不连续的内存空间的工作效率要低于连续的空间。

- 标记 - 压缩 （Mark - Compact）和标记-清除算法类似，在此基础上做了些优化。首先从根节点开始对所有可达对象做一次标记，但之后，它并不简单地清理未标记的对象，而是将所有的存活对象压缩到内存的一端。之后，清理边界外所有的空间。这种方法既避免了碎片的产生，又不需要两块相同的内存空间，因此，其性价比比较高。

- 增量算法（Incremental Collecting）为了减少系统的停顿时间，让垃圾收集和应用线程交替执行。每次，垃圾收集线程收集一小片区域的内存空间，接着切换到应用程序线程。但是线程切换和上下文转换的消耗，造成垃圾回收总成本上升，系统吞吐量下降。

- 分代（Generational Collecting）根据垃圾回收对象的特点，选择合适的回收算法。它将内存区间根据对象的特点分成几块，根据每块内存区间的特点，使用不同的回收算法，以提高垃圾回收的效率。以 Hot Spot 虚拟机为例，它将所有的新建对象都放入称为年轻代的内存区域，年轻代的特点是对象会很快回收，因此，在年轻代就选择效率较高的复制算法。当一个对象经过几次回收后依然存活，对象就会被放入称为老生代的内存空间。在老生代中，几乎所有的对象都是经过几次垃圾回收后依然得以幸存的。因此，可以认为这些对象在一段时期内，甚至在应用程序的整个生命周期中，将是常驻内存的。如果依然使用复制算法回收老生代，将需要复制大量对象。再加上老生代的回收性价比也要低于新生代，因此这种做法也是不可取的。根据分代的思想，可以对老年代的回收使用与新生代不同的标记-压缩算法，以提高垃圾回收效率。

## 垃圾收集过程

第一，Java 应用不断创建对象，通常都是分配在 Eden 区域，当其空间占用达到一定阈值时，触发 minor GC。仍然被引用的对象（绿色方块）存活下来，被复制到 JVM 选择的 Survivor 区域，而没有被引用的对象（黄色方块）则被回收。注意，我给存活对象标记了“数字 1”，这是为了表明对象的存活时间。

![Java核心技术-应用开发篇\Eden1](Java核心技术-应用开发篇\Eden1.png)

第二， 经过一次 Minor GC，Eden 就会空闲下来，直到再次达到 Minor GC 触发条件，这时候，另外一个 Survivor 区域则会成为 to 区域，Eden 区域的存活对象和 From 区域对象，都会被复制到 to 区域，并且存活的年龄计数会被加 1。

![Java核心技术-应用开发篇\Eden2](Java核心技术-应用开发篇\Eden2.png)

第三， 类似第二步的过程会发生很多次，直到有对象年龄计数达到阈值，这时候就会发生所谓的晋升（Promotion）过程，如下图所示，超过阈值的对象会被晋升到老年代。这个阈值是可以通过参数指定：`-XX:MaxTenuringThreshold=<N>`

![Java核心技术-应用开发篇\Eden3](Java核心技术-应用开发篇\Eden3.png)

后面就是老年代 GC，具体取决于选择的 GC 选项，对应不同的算法。下面是一个简单标记 - 整理算法过程示意图，老年代中的无用对象被清除后， GC 会将对象进行整理，以防止内存碎片化。

![Java核心技术-应用开发篇\老年代 标记-压缩](Java核心技术-应用开发篇\老年代 标记-压缩.png)

## GC 新的发展

- Epsilon GC，控制内存分配，不做垃圾收集。一旦Java 堆空间被耗尽，JVM 直接关闭。

- ZGC ，Oracle 开源产品，支持 T bytes 级别的堆大小，大部分情况下延迟不会超过 10 ms，目前处于试验阶段，仅支持 Linux 64 位平台。

## Minor GC 和 Full GC 的区别

Minor GC 也称新生代GC，值发生在新生代的垃圾收集动作；

Full GC又称 Major GC 或老年代GC，值只发生在老年代的GC；出现Full GC经常会伴随至少一次的Minor GC（不是绝对，Parallel Sacvenge收集器就可以选择设置Major GC策略）；

Major GC速度一般比Minor GC慢10倍以上；

## 其他资料

[Java虚拟机垃圾回收(三) 7种垃圾收集器](https://www.cnblogs.com/cxxjohnson/p/8625713.html)

[ JVM 垃圾回收器分类](https://www.ibm.com/developerworks/cn/java/j-lo-JVMGarbageCollection/index.html)

[ Java GC算法 垃圾收集器](http://www.importnew.com/23752.html)
