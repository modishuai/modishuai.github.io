---

title: Java中的引用传递示例分析
date: 2018-07-25 19:54:50
categories: 

- Java
tags:
- Java

---

> 对象的引用存放在栈内存，其对象存储在堆内存中。

> 引用传递的核心意义：同一堆内存空间可以被多个栈内存所指向（1堆 : N栈），不同的栈内存可以对同一堆内存进行内容修改。

# 引用传递分析实例一

## 案例一

```java
public class ReferenceDemo{
    public static void main(String[] args) {
        Message msg = new Message(30);
        fun(msg);
        System.out.println(msg.getNum());
    }

    public static void fun(Message temp){
        temp.setNum(100);
    }
}

class Message{
    private int num = 10;
    public Message(int num) {
       this.num = num;
    }

    public void setNum(int num) {
        this.num = num;
    }

    public int getNum() {
        return num;
    }
}
```

![引用传递分析实例\图片1](Java中的引用传递示例分析\图片1.png)

**基本类型和引用类型最大的区别是没有内存的关系匹配**

## 案例二

```java
public class ReferenceDemo{
    public static void main(String[] args) {
        String str = "Hello";
        fun(str);
        System.out.println(str);
    }

    public static void fun(String temp){
       temp = "World";
    }
}
```

![引用传递分析实例\图片2](Java中的引用传递示例分析\图片2.png)

- String 类被 final 修饰，意味着 String 内容一旦定义就不可改变。

- 字符串的内容不能被改变，字符串对象的改变的是内存地址的指向。

## 案例三

```java
public class ReferenceDemo{
    public static void main(String[] args) {
        Message msg = new Message("Hello");
        fun(msg);
        System.out.println(msg.getInfo());
    }

    public static void fun(Message temp){
       temp.setInfo("jason");
    }
}

class Message{
    private String info = "hi";
    public Message(String info) {
        this.info = info;
    }

    public void setInfo(String info) {
        this.info = info;
    }

    public String getInfo() {
        return info;
    }
}
```

**不严谨的分析图**

![引用传递分析实例\图片3](Java中的引用传递示例分析\图片3.png)

**严谨的分析图**

![引用传递分析实例\图片3-1](Java中的引用传递示例分析\图片3-1.png)

## 总结

String 属于引用类型，但是被 final 修饰后，不可变的特点，可以当基本类型使用。

String 类被 final 修饰，其变量内容一旦定义就不可改变，即一个 String 变量只能保存一个内容。

# 引用传递分析实例二

简单 Java 类编写原则：

- 类名称 = 表名称

- 属性名称（类型） =  表字段 （类型）

- 一个实例化对象 = 一行记录

- 多个实例化对象（对象数组）= 多行记录（外键）。

- 引用关系 = 外键约束

```java
class Member {
    private String name;
    private int mid;
    private Member child;
    private Car car;

    public Member(int mid, String name) {
        this.mid = mid;
        this.name = name;
    }

    public String getInfo(){
        return "编号"+this.mid+"姓名"+this.name;
    }

    public void setCar(Car car) {
        this.car = car;
    }

    public Car getCar() {
        return this.car;
    }

    public void setChild(Member child) {
        this.child = child;
    }

    public Member getchild() {
        return this.child;
    }
}

class Car {
    private int mid;
    private String carName;
    private Member member;

    public Car(int mid,String carName){
        this.mid=mid;
        this.carName=carName;
    }

    public String getInfo(){
        return "编号"+this.mid+"车名"+this.carName;
    }

    public void setMember(Member member) {
        this.member = member;
    }

    public Member getMember()
    {
        return this.member;
    }
}

public class ReferenceDemo {
    public static void main(String[] args) 
    {
        Member m = new Member(1,"素问");
        Member child = new Member(2,"血衣");
        Car c=new Car(1,"仙鹤");
        Car cc=new Car(2,"汗血宝马");
        m.setCar(c);
        c.setMember(m);

        child.setCar(cc);
        cc.setMember(child);

        m.setChild(child);
        System.out.println(m.getCar().getInfo());
        System.out.println(c.getMember().getInfo());
        System.out.println(m.getchild().getInfo());
    }
}
```
