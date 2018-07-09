---
title: Java中的字符串
date: 2018-07-08 19:20:01
categories:
- Java
tags:
- Java

---

# String

> 几乎在所有的编程语言中都没有**字符串**概念，而是采用**字符串数组**来描述字符串的概念 Java 语言也不例外。但是在所有的开发里面都离不开**字符串**的应用，那么最终的结果就是Java 创造了字符串。但是这个字符串不属于基本数据类型，它是将字符串作为**String**类的匿名对象的形式存在。

## java 中 String 实例化的方式

String 实例化的方式有两种：

* String str = “Hello”; 直接赋值

* String str = new String("Hello"); 构造器实例化

这两者实例化的区别在于：

直接赋值后，如果后续声明相同的字符串内容，则不会在堆内存空间中开辟新空间。这里的`Hello` 相当于是一个匿名对象存放在堆内存空间中，而`str`是它的一个名称，存在栈内存空间中。

```java
    public static void main(String[] args) {
       String str1 = "Hello";
       String str2 = "Hello";
       System.out.println(str1 == str2);
       //output true
    }
```

这种设计模式成为**共享设计模式** 在`JVM`底层实际上会有一个**字符串池**（不一定保存`String`对象），当代码之中使用了直接赋值的方式定义了一个`String`类对象时，会将此字符串对象所使用的匿名对象入池保存，而如果后续还有其他`String`类对象也采用直接赋值方式，并且设置了同样的内容的时候，将不会开辟新的堆内存空间，而是使用已有的对象进行引用的分配，从而继续使用。

```java
    public static void main(String[] args) {
       String str1 = "Hello";
       String str2 = new String("Hello");
       System.out.println(str1 == str2);
       //output false
    }
```

构造器实例化一定会用到`new` 关键字，表示开辟一个新的堆内存空间。像上面的代码块，我们采用了两种方式来实例化而构造器实例化则是开辟了两个堆内存空间（其中通过**直接赋值**实例化的产生的堆内空间将成为垃圾空间）。

使用构造器产生的`String`类对象，其内容不会保存在**字符串池**中（因为使用了`new`关键字）。

但是我们可以手动入池，`java.lang.String;`中的源码如下：

看到关键字`native` 就要想到这是一个本地方法。

```java
 public native String intern();
```

```java
public class StringExample{
    public static void main(String[] args) {
        String str1 = new String("Hello").intern();
        String str2 = "Hello";
        System.out.println(str1==str2);
    }
}
//output true
```

## Stiring 是一个 final 修饰的类

经过 final 修饰的类表示该类不能有子类，字符串一经定义则不可改变。

```java
 public static void main(String[] args) {
        String str = "Hello";
        str = str + "World";
        str += "!!!";
        System.out.println(str);
    }
```

![Java中的字符串\String 内存分析图](Java中的字符串\String 内存分析图.png)

## String 比较

* `==`比较，`==` 是java提供的关系运算符

用在基本类型（`byte,short,char,int,long,float,double,boolean`）比较的时候，就是比较它们的值。

用在复合类型（类），比较的就是对象的引用地址（内存地址数值）

* `euqals()` 比较，`equals()`是String 类中的一个方法，该方法用来比较字符串内容，源码如下：

```java
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```
