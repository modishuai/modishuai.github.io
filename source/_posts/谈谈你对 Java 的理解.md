---

title: 笔记：谈谈你对 Java 平台的理解？
categories:
- Java
tags:
- Java 核心技术

---

## 谈谈你对 Java 平台的理解？“Java 是解释执行“，这句话正确吗？

Java 平台最为显著的两方面特性：

一是跨平台能力，即所谓的“一次编译，到处执行“。

二是垃圾回收器（GC，Garbage Collection），回收已分配的内存空间，开发人员无需操心对内存的分配和回收。

JRE（Java Runtime Enviroment) Java 运行时环境，包括 JVM 标准实现及 Java 核心类库。JRE 自带的核心类库主要是 `JRE\lib\rt.jar`这个文件。

JDK（Java Development Kit）Java 开发环境，可以看作是JRE 的超集。它包含了Java 的运行环境（JVM + Java 系统类库）和 Java 工具，提供更多的工具，比如编译器、调试器、诊断工具。

### JDK 的基础组件包括：

javac：编译器，将源程序转化为字节码。（.java 转 .class）

jar: 打包工具，将相关类文件打包成一个文件。

javadoc：Java 文档生成工具，将源代码中的注释提取成文档。

jdb：debugger，Java 调试工具。

java：运行编译后的 Java 程序（.class后缀）

appletviewer：小程序浏览器，一种可运行在 HTML 文件上的 Java 小程序的 Java 浏览器。

Jconsole：Java 进行系统调试和监控工具。

### JDK 的基础类库

java.lang：基础类库，比如 String ，这个包是唯一不需要 import 关键字就可以使用的包。 

java.io：有关输入输出的类，比如对文件的操作。

java.nio：为了完善 io 包的功能，提高 io 包的性能而新增的包，比如 nio 非阻塞应用。

java.net：有关网络的类，比如 URI、URLConnection等。

java.util：系统辅助类，特别是集合 Collection 、List、Map等。

java.sql：数据库操作相关类，Connection、Statement、ResultSet等。

javax.servlet：jsp、servlet 等所使用的类。

### Java 是解释执行，正确吗？

这个说法不太正确，我们开发好的 Java 源代码，首先是经过 javac 编译成字节码（bytecode），然后在运行时，通过JVM（Java 虚拟机）内嵌的解释器将编译好的字节码转化成最终的机器码。但是常用的JVM 比如Oracle 提供的 Hotspot JVM，都提供了 JIT （Just-In-Time Compiler）即时编译器，JIT 能够在运行时将热点代码编译成机器码，这个过程成为“编译执行“而非“解释执行“。

> JIT（Just-In-Time）即时编译器，通常通过 javac 将程序源代码编译，转换成 java 字节码，JVM 通过解释字节码将其翻译成对应的机器指令，逐条读入，逐条解释翻译。很显然，经过解释执行，其执行速度必然会比可执行的二进制字节码程序慢很多。为了提高执行速度，引入了 JIT 技术。
> 
> 在运行时 JIT 会把翻译过的机器码保存起来，以备下次使用，因此从理论上来说，采用该 JIT 技术可以接近以前纯编译技术。

我们通常把 Java 分为编译期和运行时。

这里说的 Java 的编译和 C/C++ 是有着不同的意义的，Javac 的编译，编译 Java 源码生成“.class”文件里面实际是字节码，而不是可以直接执行的机器码。Java 通过字节码和 Java 虚拟机（JVM）这种跨平台的抽象，屏蔽了操作系统和硬件的细节，这也是实现“一次编译，到处执行”的基础。

在运行时，JVM 会通过类加载器（Class-Loader）加载字节码，解释或者编译执行。就像我前面提到的，主流 Java 版本中，如 JDK 8 实际是解释和编译混合的一种模式，即所谓的混合模式（-Xmixed）。通常运行在 server 模式的 JVM，会进行上万次调用以收集足够的信息进行高效的编译，client 模式这个门限是 1500 次。Oracle Hotspot JVM 内置了两个不同的 JIT compiler，C1 对应前面说的 client 模式，适用于对于启动速度敏感的应用，比如普通 Java 桌面应用；C2 对应 server 模式，它的优化是为长时间运行的服务器端应用设计的。默认是采用所谓的分层编译（TieredCompilation）。

除了我们日常最常见的 Java 使用模式，其实还有一种新的编译方式，即所谓的 AOT（Ahead-of-Time Compilation），直接将字节码编译成机器代码，这样就避免了 JIT 预热等各方面的开销，比如 Oracle JDK 9 就引入了实验性的 AOT 特性，并且增加了新的 jaotc 工具。利用下面的命令把某个类或者某个模块编译成为 AOT 库。

```java
jaotc --output libHelloWorld.so HelloWorld.class
jaotc --output libjava.base.so --module java.base
```

然后在启动时直接指定就可以了

```java
java -XX:AOTLibrary=./libHelloWorld.so,./libjava.base.so HelloWorld
```

而且，Oracle JDK 支持分层编译和 AOT 协作使用，这两者并不是二选一的关系。如果你有兴趣，可以参考相关文档：[http://openjdk.java.net/jeps/295](http://openjdk.java.net/jeps/295)。AOT 也不仅仅是只有这一种方式，业界早就有第三方工具（如 GCJ、Excelsior JET）提供相关功能。

## 精彩评论

> Woj
> 
> “一次编译、到处运行”说的是Java语言跨平台的特性，Java的跨平台特性与Java虚拟机的存在密不可分，可在不同的环境中运行。比如说Windows平台和Linux平台都有相应的JDK，安装好JDK后也就有了Java语言的运行环境。其实Java语言本身与其他的编程语言没有特别大的差异，并不是说Java语言可以跨平台，而是在不同的平台都有可以让Java语言运行的环境而已，所以才有了Java一次编译，到处运行这样的效果。  
严格的讲，跨平台的语言不止Java一种，但Java是较为成熟的一种。“一次编译，到处运行”这种效果跟编译器有关。编程语言的处理需要编译器和解释器。Java虚拟机和DOS类似，相当于一个供程序运行的平台。  
程序从源代码到运行的三个阶段：编码——编译——运行——调试。Java在编译阶段则体现了跨平台的特点。编译过程大概是这样的：首先是将Java源代码转化成.CLASS文件字节码，这是第一次编译。.class文件就是可以到处运行的文件。然后Java字节码会被转化为目标机器代码，这是是由JVM来执行的，即Java的第二次编译。  
“到处运行”的关键和前提就是JVM。因为在第二次编译中JVM起着关键作用。在可以运行Java虚拟机的地方都内含着一个JVM操作系统。从而使JAVA提供了各种不同平台上的虚拟机制，因此实现了“到处运行”的效果。需要强调的一点是，java并不是编译机制，而是解释机制。Java字节码的设计充分考虑了JIT这一即时编译方式，可以将字节码直接转化成高性能的本地机器码，这同样是虚拟机的一个构成部分。
> 
> 一叶追寻
> 
> 对Java平台的理解，首先想到的是Java的一些特性，比如平台无关性、面向对象、GC机制等，然后会在这几个方面去回答。平台无关性依赖于JVM，将.class文件解释为适用于操作系统的机器码。面向对象则会从封装、继承、多态这些特性去解释，具体内容就不在评论里赘述了。另外Java的内存回收机制，则涉及到Java的内存结构，堆、栈、方法区等，然后围绕什么样的对象可以回收以及回收的执行。以上是我对本道题的理解，不足之处还请杨老师指出，希望通过这次学习能把Java系统的总结一下~
> 
> 三军
> 
> Java特性: 
> 
> 1. 面向对象（封装，继承，多态）
> 
> 2. 平台无关性（JVM运行.class文件)
> 
> 3. 语言（泛型，Lambda）
> 
> 4. 类库（集合，并发，网络，IO/NIO）
> 
> 5. JRE（Java运行环境，JVM，类库）
> 
> 6. JDK（Java开发工具，包括JRE，javac，诊断工具）
> 
> Java是解析运行吗？  不正确！
> 
> 1. Java源代码经过Javac编译成.class文件
> 
> 2. .class文件经JVM解析或编译运行。
>    
>    （1）解析:.class文件经过JVM内嵌的解析器解析执行。
>    
>    （2）编译:存在JIT编译器（Just In Time Compile 即时编译器）把经常运行的代码作为"热点代码"编译与本地平台相关的机器码，并进行各种层次的优化。
>    
>    （3） AOT编译器: Java 9提供的直接将所有代码编译成机器码执行。
> 
> jack：Java平台包括java语言，class文件结构，jvm，api类库，第三方库，各种编译、监控和诊断工具等。  
Java语言是一种面向对象的高级语言；通过平台中立的class文件格式和屏蔽底层硬件差异的jvm实现‘一次编写，到处运行’；  
通过‘垃圾收集器’管理内存的分配和回收。  
jvm通过使用class文件这种中间表示和具体语言解耦，使得任何在源码早期编译过程中以class文件为中间表示或者  
能够转换成class文件的具体语言，都能运行jvm之上，也就可以使用jvm的各种特性。  
api类库主要包含集合、IO/NIO、网络、并发等。  
第三方库包括各种商业机构和开源社区的java库，如spring、mybatis等。  
各种工具如javac、jconsole、jmap、jstack等。
> 
> 杨文奇： 解释器是把代码一行一行解释为二进制指令，编译器是把代码一次性编译为二进制指令，对JAVA语言来说，在执行阶段区分编译和解释，但对C语言来说，在执行阶段是直接执行二进制指令，不存在编译和解释，不知道理解对吗？
> 
> 作者回复：嗯，c语言直接编译为机器码，并不存在虚拟机，不存在Java的部分概念
