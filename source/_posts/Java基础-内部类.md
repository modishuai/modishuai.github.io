---
title: java基础：内部类
date: 2018-08-01 19:24:08
categories:
- Java
tags:
- Java
---
# 内部类

- 内部类的基本定义结构

- 使用static 定义内部类

- 在方法中定义内部类

## 基本概念

所谓内部类就是在一个类的内部定义了其他内部结构的类。

```java
class OutClass{
    private String msg = "Hello World!";
    class InnerClass{
        public void print(){
            System.out.println(msg);
        }
    }
    public void fun(){
        //实例化内部类对象并调用print()方法。
        new InnerClass().print();
    }
}
public class Example{
    public static void main(String[] args) {
        OutClass out = new OutClass();
        out.fun();
    }
}
```

通常我们说一个类本身是由属性与方法组成，那么类中包含另外一个类似乎不那么合理。是什么样的目的要在一个类中定义一个内部类呢？

为了说明以上问题，我们将内部类从外部类中剥离出来。

```java
class OutClass{
    private String msg = "Hello World!";
    
    public void fun(){
        //实例化内部类对象并调用print()方法。
        new InnerClass().print();
    }
}

class InnerClass{
    public void print(){
        System.out.println(msg);
    }
}

public class Example{
    public static void main(String[] args) {
        OutClass out = new OutClass();
        out.fun();
    }
}
```

1. 内部类需要访问msg 属性，该属性来自`OutClas` 并且用`private`封装起来了，为了让`InnerClass` 访问该属性，则必须为属性定义get() 方法。

   ```java
   public String getMsg() {
           return msg;
   }
   ```

2.  现在OutClass 类中的getMsg() 方法是一个普通方法，所有类中的普通方法必须要通过类的实例化对象调用。

   此处我们声明了两个OutClass 对象，如何改进呢？

   ```java
   class OutClass{
       private String msg = "Hello World!";
       public String getMsg() {
           return msg;
       }
       public void fun(){
           //实例化内部类对象并调用print()方法。
           new InnerClass().print();
       }
   }
   
   class InnerClass{
       public void print(){
           System.out.println(new OutClass().getMsg());
       }
   }
   
   public class Example{
       public static void main(String[] args) {
           OutClass out = new OutClass();
           out.fun();
       }
   }
   ```

   改进后 

   ```java
   class OutClass{
       private String msg = "Hello World!";
       public String getMsg() {
           return msg;
       }
       public void fun(){
           //实例化内部类对象并调用print()方法。
           new InnerClass(this).print();
       }
   }
   
   class InnerClass{
       private OutClass out;
       public InnerClass(OutClass out) {
          this.out = out;
       }
       public void print(){
           System.out.println(this.out.getMsg());
       }
   }
   
   public class Example{
       public static void main(String[] args) {
           OutClass out = new OutClass();
           out.fun();
       }
   }
   ```

   以上代码只是为了让内部类访问外部类的一个私有属性。

## 内部类的优点

- 内部类可以方便的访问外部类私有属性及操作。

- 反之，外部类也可以通过内部类对象访问内部类的私有属性。

```java
class OutClass{
    private String msg = "Hello World!";
    class InnerClass{
        private String info = "Hello Java!";
        public void print(){
            System.out.println(msg);
        }
    }
    public void fun(){
        InnerClass inner = new InnerClass();
        System.out.println(inner.info);
    }
}



public class Example{
    public static void main(String[] args) {
        OutClass out = new OutClass();
        out.fun();
    }
}
```

## 内部类离不开外部类的实例化对象

```shell

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         2018/8/1     17:14            314 Example.class
-a----         2018/8/1     17:14            570 OutClass$InnerClass.class
-a----         2018/8/1     17:14            524 OutClass.class

```

编译后的文件名称中有个`$` 作为文件名的标识换回程序内就是`.` 

外部类.内部类 对象 = new 外部类().new 内部类();

```java
class OutClass{
    private String msg = "Hello World!";
    class InnerClass{
        public void print(){
            System.out.println(OutClass.this.msg);
        }
    }
    public void fun(){
        new InnerClass().print();
    }
}



public class Example{
    public static void main(String[] args) {
        OutClass.InnerClass oi = new OutClass().new InnerClass();
        oi.print();
    }
}
```

如何限制一个内部类只能被一个外部类访问，不能被外部调用。

答案就是利用`private` 来修饰内部类



# 使用static 定义内部类

我们最大使用 static 修饰的属性或方法是不受类实例化对象的控制，同样使用 static 修饰的内部类也不能被外部类实例化对象控制。

经过 static 修饰的内部类就变成了一个外部类，并且只能够访问外部类中定义的static 属性或方法。

```java
class OutClass{
    private static String msg = "Hello World!";
    static class InnerClass{
        public void print(){
            System.out.println(msg);
        }
    }
    public void fun(){
        new InnerClass().print();
    }
}
```

```java
class OutClass{
    private static String msg = "Hello World!";
    static class InnerClass{
        public void print(){
            System.out.println(msg);
        }
    }
    public void fun(){
        new InnerClass().print();
    }
}

public class Example{
    public static void main(String[] args) {
       // OutClass.InnerClass oi = new OutClass().new InnerClass();
        //oi.print();
        OutClass.InnerClass oi = new OutClass.InnerClass();
        oi.print();
    }
}
```

static声明的内部类，相当于定义了一个外部类，实例化对象格式：外部类.内部类 对象 = new 外部类.内部类();
