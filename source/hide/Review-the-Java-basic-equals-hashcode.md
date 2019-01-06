title: 'Java 基础复习实践 --- Hashcode Equals'
date: 2016-07-18 14:14:31
---

虽然很多知识点书籍都有整理，但是记性总是不好，所以决定将一些细小容易混淆的概念，通过简单的 Demo 实践，加深复习。特此开一个坑，坚持就是胜利。

本章内容主要为了理解以下几个知识点：
- equals() 的作用是什么？
- equals() 与 “==”的区别是什么？
- hashcode() 的作用是什么？
- hashcode() 与 equals（）之间有什么联系？

<!-- more -->

# 目录

<!-- toc -->

- [目录](#目录)
- [0x01 equals() 的作用](#0x01-equals-的作用)
  - [没有覆盖 equals() 方法](#没有覆盖-equals-方法)
  - [覆盖 equals() 方法](#覆盖-equals-方法)
  - [Tips](#tips)
- [0x02 equals() 与 == 的区别](#0x02-equals-与-的区别)
- [0x03 hashcode() 的作用](#0x03-hashcode-的作用)
- [0x04 hashCode() 和 equals() 的关系](#0x04-hashcode-和-equals-的关系)
- [参考文档](#参考文档)

<!-- tocstop -->



# 0x01 equals() 的作用

>Indicates whether some other object is "equal to" this one.

equals()是用来 __判断两个对象是否相等__。

equals() 定义在 JDK 的 Object.java 中。通过判断两个对象的地址是否相等(即，是否是同一个对象)来区分它们是否相等。源码如下：

```Java
public boolean equals(Object obj) {
    return (this == obj);
}
```
既然是在 Object 类中定义了该方法，就表明了 Java 所有类都实现了 equals() 方法，所以所有类都可以通过该方法去判断两个对象是否相等。
默认的 equals（） 方法等同于 ”==” 方法，所以我们一般会重写 equals（） 方法 —-> 两个对象的内容相等，返回 true ，否则返回 false。

所以我们可以根据是否 __重写 equals()__ 方法将类分为两类：

1.若某个类没有覆盖 equals() 方法，当它的通过 equals() 比较两个对象时，实际上是比较两个对象是不是同一个对象。这时，等价于通过“==”去比较这两个对象

2.覆盖类的 equals() 方法，来让 equals() 通过其它方式比较两个对象是否相等。通常的做法是：若两个对象的内容相等，则 equals()方法返回true；否则，返回 false 。


## 没有覆盖 equals() 方法
```java

public class HashcodeAndEquals {

	private static <T> void out(T t) {
		System.out.println(t);
	}

	public static void main(String[] args) {
		// 实例化两个 Person 对象
		Person p1 = new Person("小明", 12);
		Person p2 = new Person("小明", 12);
		// 通过 equals() 比较他们是否相等
		out(p1.equals(p2));

	}

	private static class Person {
		int age;
		String name;

		public Person(String name, int age) {
			this.age = age;
			this.name = name;
		}

		@Override
		public String toString() {

			return name + "--- age:" + age;
		}

	}

}

```
> 输出为
> false

结论：

<div class="tip">
当我们使用 `p1.equals(p2)` 来比较 p1 和 p2 是否相等时，实际上是调用了 object 类的 equals() 方法，即 “p1==p2”，它是比较 p1 和 p2 是否为一个对象。

由定义可知，p1 和 p2 虽然内容相同，但是它们是两个不同的对象。因此，返回 false 。
</div>

## 覆盖 equals() 方法

我们简单修改下 `HashcodeAndEquals.java` 文件，覆盖 __equals() 方法__：

```java
public class HashcodeAndEquals {

	private static <T> void out(T t) {
		System.out.println(t);
	}

	public static void main(String[] args) {
		// 实例化两个 Person 对象
		Person p1 = new Person("小明", 12);
		Person p2 = new Person("小明", 12);
		// 通过 equals() 比较他们是否相等
		out(p1.equals(p2));

	}

	private static class Person {
		int age;
		String name;

		public Person(String name, int age) {
			this.age = age;
			this.name = name;
		}

		@Override
		public String toString() {

			return name + "--- age:" + age;
		}

		@Override
		public boolean equals(Object obj) {
			if (obj == null) {
				return false;
			}

			// 如果是同一对象，返回 true
			if (this == obj) {
				return true;
			}
			// 判断是否类型相同
			if (this.getClass() != obj.getClass()) {
				return false;
			}
          	// 只有一下三种情况才能通过编译
			// 1.instanceof前面的类型与后面的类型相同
			// 2.instanceof前面的类型是后面的类型父类
			// 3.instanceof前面的类型是后面的类型子类
			// 故与我们判断类型相同有一点偏差
			//
          	// 这一点我们可以自己定义一个 Student 类继承 Person 类来进行实验
			// if (!(obj instanceof Person)) {
			// return false;
			// }

			Person per = (Person) obj;
			return name.equals(per.name) && age == per.age;
		}

	}

}
```
>输出为
>true

结论：

<div class="tip">
我们在新的 `HashcodeAndEquals.java` 文件中重写了 Person 类的 equals() 函数：当两个 Person 对象 name 和 age 都相等时，则返回 true ，因此结果为 true 。
</div>

## Tips

> Java 对 equals() 的要求

1. 对称性：如果 x.equals(y) 返回是" true "，那么 y.equals(x) 也应该返回是"true"。
2. 反射性：x.equals(x) 必须返回是 "true" 。
3. 类推性：如果 x.equals(y) 返回是"true"，而且 y.equals(z) 返回是"true"，那么 z.equals(x) 也应该返回是"true"。
4. 一致性：如果 x.equals(y) 返回是"true"，只要x和y内容一直不变，不管你重复 x.equals(y) 多少次，返回都是"true"。
5. 非空性，x.equals(null)，永远返回是"false"；x.equals (和x不同类型的对象)永远返回是"false"。


# 0x02 equals() 与 == 的区别

equals() : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况(前面第1部分已详细介绍过)

== : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是否为同一个对象。



只用修改下`HashcodeAndEquals.java` 主函数为：
```java
public static void main(String[] args) {
    // 实例化两个 Person 对象
    Person p1 = new Person("小明", 12);
    Person p2 = new Person("小明", 12);
    // 分别用 equals() 和 == 来判断
    out("equals: " + p1.equals(p2));
    out("==: " + (p1 == p2));
}
```
> 输出为：
> equals: true
> ==: false

结果与我们预想的一样，因为我们是复写了 Person 类的 equals() 方法，而且 p1 p2 内容相同所以返回 true ，而 p1 p2 并是两个不同对象，所以 == 判断它们地址不相同，返回 false。

# 0x03 hashcode() 的作用

hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。

hashCode() 定义在 JDK 的 Object.java 中，这就意味着Java中的任何类都包含有 hashCode() 函数。

虽然，每个 Java 类都包含 hashCode() 函数。但是，仅仅当创建并某个“类的散列表”(关于“散列表”见下面说明)时，该类的 hashCode() 才有用(作用是：确定该类的每一个对象在散列表中的位置；其它情况下(例如，创建类的单个对象，或者创建类的对象数组等等)，类的 hashCode() 没有作用。

上面的散列表，指的是：Java集合中本质是散列表的类，如HashMap，Hashtable，HashSet。
也就是说：hashCode() 在散列表中才有用，在其它情况下没用。
在散列表中 hashCode() 的作用是获取对象的散列码，进而确定该对象在散列表中的位置。

散列码的解释：
> 我们都知道，散列表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！
> 散列表的本质是通过数组实现的。当我们要获取散列表中的某个“值”时，实际上是要获取数组中的某个位置的元素。而数组的位置，就是通过“键”来获取的；更进一步说，数组的位置，是通过“键”对应的散列码计算得到的。

我们以 Hashset 为例来说明下 hashcode() 的作用：
首先我们都知道 HashSet 是 Set 的集合，不允许有重复的元素。当该 Set 已经有了 1000 个元素时，当插入第1001个元素时，需要怎么处理？ “将第1001个元素逐个的和前面1000个元素进行比较”？显然，这个效率是相等低下的。散列表很好的解决了这个问题，它根据元素的散列码计算出元素在散列表中的位置，然后将元素插入该位置即可。对于相同的元素，自然是只保存了一个。由此可知，若两个元素相等，它们的散列码一定相等；但反过来确不一定。
在散列表中，
1. 如果两个对象相等，那么它们的hashCode()值一定要相同；
2. 如果两个对象hashCode()相等，它们并不一定相等。
   注意：这是在散列表中的情况。在非散列表中一定如此！

# 0x04 hashCode() 和 equals() 的关系

我们修改主函数为：
```java
public static void main(String[] args) {
    Person p1 = new Person("小明", 12);
    // HashMap
    HashMap<Person, Integer> map = new HashMap<>();
    map.put(p1, 1);
    out(map.get(new Person("小明", 12)));

}
```
按照理想中，我们输出的结果应该为 “1”，因为我们存入的 Person 和查找的 Person 都是“小明”，是同一个人。
但是最终运行该程序输出结果为：

> null

所以按照设计标准我们应该在重写 equals() 方法的同时也要写重写 hashcode()。

虽然通过重写equals方法使得逻辑上姓名和年龄相同的两个对象被判定为相等的对象（跟String类类似），但是要知道默认情况下，hashCode 方法是将对象的存储地址进行映射。那么上述代码的输出结果为“null”就不足为奇了。

因为 p1 对象和 `new Person("小明", 12)` 生成的对象是两个不同的对象，它们的存储地址肯定不同，所以得到的 hashcode 值不同（不绝对，因为有哈希冲突的情况）。

所以现在重写下我们的 Person 类的 hashcode() 方法，利用 eclipse 自动生成如下：

```java
@Override
public int hashCode() {
    final int prime = 31;
    int result = 1;
    result = prime * result + age;
    result = prime * result + ((name == null) ? 0 : name.hashCode());
    return result;
}
```

现在我们再次运行程序得到结果：

> 1

与预期一致。

摘自Effective Java一书：

> - 在程序执行期间，只要equals方法的比较操作用到的信息没有被修改，那么对这同一个对象调用多次，hashCode方法必须始终如一地返回同一个整数。
> - 如果两个对象根据equals方法比较是相等的，那么调用两个对象的hashCode方法必须返回相同的整数结果。
> - 如果两个对象根据equals方法比较是不等的，则hashCode方法不一定得返回不同的整数。

对于第一条，我们通过一个例子来验证：
```java
public static void main(String[] args) {
    Person p1 = new Person("小明", 12);
    out("初始值：" + p1.hashCode());
    HashMap<Person, Integer> map = new HashMap<>();
    map.put(p1, 1);
    p1.age = 13;
    out("更新后：" + p1.hashCode());
    out(map.get(p1));
}
```

输出结果为：

> 初始值：758036
> 更新后：758067
> null

其中原因我就不用多说了，因此，在设计 hashCode 方法和 equals 方法的时候，如果对象中的数据易变，则最好在 equals 方法和 hashCode 方法中不要依赖于该字段。

# 参考文档

- [如何生成一个合适的hashcode方法](http://www.importnew.com/8189.html)
- [Java hashCode() 和 equals()](http://www.cnblogs.com/skywang12345/p/3324958.html)
