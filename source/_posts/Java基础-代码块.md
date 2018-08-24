---
title: java基础：代码块
date: 2018-08-01 19:24:08
categories:
- Java
tags:
- Java
---

代码块分为四中：普通代码块、构造块、静态块、同步代码块

## 普通代码块

普通代码块写在`{}`内



## 构造块

```java
class Message{
    public Message() {
        System.out.println("[A]\t A 类构造方法");
    }
    {
        System.out.println("[B]\t B 类构造块");
    }
}

public class CodeBlock{
    public static void main(String[] args) {
        new Message();
        new Message();
        new Message();
        new Message();
    }
}
//out put
[B]      B 类构造块
[A]      A 类构造方法
[B]      B 类构造块
[A]      A 类构造方法
[B]      B 类构造块
[A]      A 类构造方法
[B]      B 类构造块
[A]      A 类构造方法
```

分析以上代码块可以得出以下结论：

- 构造块的调用优先与构造方法。

- 如果产生了多个类对象，那么构造块将会被重复执行多次。



## 静态块

静态块的使用场景分以下情况：

**情况一：在非主类中使用**

如果代码使用 static 进行定义的话，就称为静态块。

```java
class Message{
    public Message() {
        System.out.println("[A]\t A 类构造");
    }
    {
        System.out.println("[B]\t B 类构造");
    }
    static {
        System.out.println("[C]\t C 静态块");
    }
}

public class CodeBlock{
    public static void main(String[] args) {
        new Message();
        new Message();
        new Message();
        new Message();
    }
}
//out put
[C]      C 静态块
[B]      B 类构造
[A]      A 类构造
[B]      B 类构造
[A]      A 类构造
[B]      B 类构造
[A]      A 类构造
[B]      B 类构造
[A]      A 类构造
```

静态块优先与构造块执行，无论多少个实例化对象，静态块有且只执行一次。其重要功能是为了类中的 static 属性初始化。

**情况二：在主类中使用**

```java
public class CodeBlock{
    static{
        System.out.println("Static Block");
    }
    public static void main(String[] args) {
        System.out.println("Hello Java");
    }
}
```

静态块优先于`main`方法

## 同步代码块




