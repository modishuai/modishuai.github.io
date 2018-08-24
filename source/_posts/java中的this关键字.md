---
title: java 中的 this 关键字
date: 2018-07-24 19:59:08
categories:
- Java
tags:
- Java
---
# this 关键字

* 实现类属性调用。

* 实现类方法调用。

* 表示当前类对象。



## 实现类属性调用

* 错误示例

  ```java
  public class Computer{
      private String name;
      private double price;
  
      public Computer(String name,double price) {
          name = name;
          price = price;
      }
  }
  ```

* 正确示例

  ```java
  public class Computer{
      private String name;
      private double price;
  
      public Computer(String name,double price) {
          this.name = name;
          this.price = price;
      }
  }
  ```

  以上代码块可以看出问题出在**作用域**上面。

## 实现类方法调用

* 普通方法

  ```java
  public class Computer{
      private String name;
      private double price;
  
      public Computer(String name,double price) {
          super();
          this.name = name;
          this.price = price;
      }
  
      public void getInfo(){
         print();
         System.out.println("商品名称："+this.name+"价格："+this.price);
      }
      public void print(){
          System.out.println("*********************");
      }
      public static void main(String[] args) {
          Computer computer = new Computer("MacBookPro 15",23000.0);
          computer.getInfo();
      }
  }
  ```

  普通方法调用前加或不加`this`没有必须要求，但是为了方法调用的严谨性，通常加上`this`关键字靠谱。

  在一个类中除了普通方法还有构造方法，多个构造方法间进行互相调用`this(arguments)`。

* 构造方法

  ```java
  public class Computer{
      private String name;
      private double price;
      public Computer() {
          this("2015 MacBookPro 15",17000.0);
      }
      public Computer(String name) {
          this(name,18000);
      }
  
      public Computer(String name,double price) {
          this.name = name;
          this.price = price;
      }
  
      public String getInfo(){
        return "商品名称："+this.name+"价格："+this.price;
      }
      public static void main(String[] args) {
          Computer computer0 = new Computer();
          Computer computer1 = new Computer("2017 MacBookPro 15");
          Computer computer2 = new Computer("2018 MacBookPro 15",23000.0);
          System.out.println(computer0.getInfo());
          System.out.println(computer1.getInfo());
          System.out.println(computer2.getInfo());
      }
  }
  ```

  * 构造方法互相调用一定要放在第一行。

  * 构造方法 互相调用一定要留出口（不要循环调用）。





## 表示当前类对象

```java
public class Computer{
    public static void main(String[] args) {
        //1. 实例化A类对象，执行A类无参构造
        A temp = new A();
    }
}

class A{
    private B b;
    //2. 执行A类构造
    public A() {
        //3. 执行B类实例化
        this.b = new B(this);//4. this 表示 temp 
        //7. 调用B类get()
        this.b.get();
    }
    //10. 调用print()
    public void print(){
        System.out.println("Hello World!");
    }
}

class B{
    private A a ;
    //5. 参数 a = temp
    public B(A a){
        this.a = a; //6. 保存a对象（保存temp）
    }
    //8. 调用此方法
    public void get(){
        //9. this.a = temp
        this.a.print();
    }
}
```

## 总结

*  类中的构造方法间的互相调用，不仅要放在第一行还要保留出口。 

* 类中的属性调用为避免作用域带来的问题，要使用 this 关键字。

* this 表示当前对象，指的是当前正在调用类中方法的对象，this不是一个固定的。
