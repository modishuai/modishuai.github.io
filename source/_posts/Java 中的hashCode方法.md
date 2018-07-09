---
title: Java 中的hashCode方法
categories:
- Java
tags:
- Java
- 数据结构
---
# Java 中的hashCode方法

哈希码是算法中的一种，哈希码产生的依据：哈希码并不是完全唯一的，让同一个类的对象按照自身的特征尽量的有不同的哈希码，但不表示不同的对象哈希码完全不同。也有相同的情况，具体看开发人员如何实现哈希码的算法。

在Java 中，HashCode 代表对象的特征。

- Object 类的 hashCode 返回对象的内存地址经过处理后的结构（也就是说不一定是实际的内存地址），由于每个对象的内存地址都不一样，所以哈希码也不尽相同。

- String 类的 hashCode 根据String 类包含的字符串的聂荣，根据特殊的算法返回哈希码，只要字符串所在的堆空间相同，返回的哈希码也相同。

- Integer 类，返回的哈希码是Integer 对象包含的整数值，也就是说 Integer inte = 100; inte.hashCode() 也是100。

**equals**

在 Java 中 equals 默认是比较对象的引用，如果要比较内容是否相同那么都要重写Objec 类的equals方法。

## hashCode() 的作用

hashCode方法的主要作用是为了配合基于散列的集合一起正常运行，这样的散列集合包括HashSet、HashMap以及HashTable。

当向集合中插入对象时，如何判别在集合中是否已经存在该对象了？（注意：集合中不允许重复的元素存在）

也许大多数人都会想到调用equals方法来逐个进行比较，这个方法确实可行。但是如果集合中已经存在一万条数据或者更多的数据，如果采用equals方法去逐一比较，效率必然是一个问题。此时hashCode方法的作用就体现出来了，当集合要添加新的对象时，先调用这个对象的hashCode方法，得到对应的hashcode值，实际上在HashMap的具体实现中会用一个table保存已经存进去的对象的hashcode值，如果table中没有该hashcode值，它就可以直接存进去，不用再进行任何比较了；如果存在该hashcode值， 就调用它的equals方法与新元素进行比较，相同的话就不存了，不相同就散列其它的地址，所以这里存在一个冲突解决的问题，这样一来实际调用equals方法的次数就大大降低了

Java中的hashCode方法就是根据一定的规则将与对象相关的信息（比如对象的存储地址，对象的字段等）映射成一个数值，这个数值称作为散列值。

## hashCode 和 equals 基本约定

- equals 相等，hashCode 一定也要相等。

- 重写 hashCode 也要重写 equals。

- hashCode 需要保持一致性，状态改变返回的哈希值仍然要一致。

- equals 的对称、反射、传递等特性。

摘录：《Java 编程思想》

设计hashCode()时最重要的因素就是：无论何时，对同一个对象调用hashCode()都应该产生同样的值。如果在讲一个对象用put()添加进HashMap时产生一个hashCdoe值，而用get()取出时却产生了另一个hashCode值，那么就无法获取该对象了。所以如果你的hashCode方法依赖于对象中易变的数据，用户就要当心了，因为此数据发生变化时，hashCode()方法就会生成一个不同的散列码。

摘录：《Effective Java》

- 在程序执行期间，只要equals方法的比较操作用到的信息没有被修改，那么对这同一个对象调用多次，hashCode方法必须始终如一地返回同一个整数。

- 如果两个对象根据equals方法比较是相等的，那么调用两个对象的hashCode方法必须返回相同的整数结果。

- 如果两个对象根据equals方法比较是不等的，则hashCode方法不一定得返回不同的整数。

**重写 equals 方法**

```java
import java.util.HashMap;
public class User {
	public User(String name){
		this.name = name;
	}
	private String name;
	public String getName() {
		return name;
	}
  public void setName(String name) {
  	this.name = name;
  }
  public static void main(String[] args){
    User user1 = new User("limi");
    System.out.println(user1.hashCode());
    HashMap<User,String> hashMap = new HashMap<User,String>();
    hashMap.put(user1, "789456123");
    System.out.println(hashMap.get(new User("limi")));
  }
  @Override
  public boolean equals(Object obj){
    if (!(obj instanceof User)){
    	return false;
  	}
    User user = (User)obj;
    if(user.getName().equals(this.getName())){
      return true;
    }else{
      return false;
    }
  }
}
/** out put
* 366712642
* null
*/
```

在 User 类中只重写了 equals 方法，假设存在两个User 对象，他们的 name 都是 limi，我们则认为这是同一个对象。

在 User 类中只重写了 equals 方法，假设存在两个User 对象，他们的 name 都是 limi，我们则认为这是同一个对象。

我们期望最后打印的结果是“789456123" ，实际输出的是 null ，按照上面提到的约定重写equals 方法的同时一定要重写 hashCode 方法。

通过重写 equals 使得逻辑上两个姓名相同的两个对象判定为相同的对象。然而 hashCode 方法默认是将对象的内存地址进行映射， `System.out.println(hashMap.get(new User("limi"))); `这里新增了一个对象，也就意味着hashCode 方法对新对象生成了新的散列值，所以输出结果是 null

**hashMap.get() 实现**

```java
public V get(Object key) {
	if (key == null)
		return getForNullKey();
		int hash = hash(key.hashCode());
		for (Entry<K,V> e = table[indexFor(hash, table.length)];
			e != null;
			e = e.next) {
			Object k;
			if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
				return e.value;
			}
			return null;
}
```

**重写hashCode()**

```java
import java.util.HashMap;
public class User {
	public User(String name){
		this.name = name;
	}
	private String name;
  public String getName() {
  	return name;
  }
	public void setName(String name) {
    this.name = name;
  }
	public static void main(String[] args){
		User user1 = new User("limi");
		System.out.println(user1.hashCode());
		HashMap<User,String> hashMap = new HashMap<User,String>();
		hashMap.put(user1, "789456123");
		System.out.println(hashMap.get(new User("limi")));
	}
  @Override
	public boolean equals(Object obj){
    if (!(obj instanceof User)){
			return false;
		}
		User user = (User)obj;
		if(user.getName().equals(this.getName())){
			return true;
		}else{
			return false;
    }
	}
	@Override
	public int hashCode(){
		return name.hashCode();
	}
}
/**
* output
* 3321817
* 789456123
*/
```

如果重写equals后，如果不重写hashcode，则hashcode就是继承自Object的，返回内存编码的映射，这时候可能出现equals相等，而hashcode不等，你的对象使用集合时，就会得不到正确的结果。

如果重写equals后，如果不重写hashcode，则hashcode就是继承自Object的，返回内存编码的映射，这时候可能出现equals相等，而hashcode不等，你的对象使用集合时，就会得不到正确的结果。

**java.util.HashMap的中put方法实现**

```java
public  V put(K key, V value) {
	if  (key == null)
	return  putForNullKey(value);
	int  hash = hash(key.hashCode());
	int  i = indexFor(hash, table.length);
	for  (Entry<K,V> e = table\[i\]; e != null; e = e.next) {
		Object k;
		if  (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
			V oldValue = e.value;
			e.value = value;
			e.recordAccess(this);
			return  oldValue;
		}
	}
	modCount++;
	addEntry(hash, key, value, i);
	return  null;
}
```

put方法是用来向HashMap中添加新的元素，从put方法的具体实现可知，会先调用hashCode方法得到该元素的hashCode值，然后查看table中是否存在该hashCode值，如果存在则调用equals方法重新确定是否存在该元素，如果存在，则更新value值，否则将新的元素添加到HashMap中。从这里可以看出，hashCode方法的存在是为了减少equals方法的调用次数，从而提高程序效率。

---

参考文献：

https://www.cnblogs.com/dolphin0520/p/3681042.html
