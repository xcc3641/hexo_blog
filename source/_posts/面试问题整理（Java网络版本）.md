title: 面试问题整理（Java网络版本）
date: 2015-11-10 22:14:07
categories: 技术
tags: 面试
---
<!-- excerpt -->

### JavaSE
#### 九种基本数据类型的大小，以及他们的封装类

Java提供了一组基本数据类型，包括boolean, byte, char, short,  int, long, float, double, void. 同时，java也提供了这些类型的封装类，分别为Boolean, Byte, Character, Short, Integer, Long, Float, Double, Void。
##### 既然提供了基本类型，为什么还要使用封装类呢？

- 某些情况下，数据必须作为对象出现,此时必须使用封装类来将简单类型封装成对象。
1. 比如，如果想使用List来保存数值，由于List中只能添加对象，因此我们需要将数据封装到封装类中再加入List。在JDK5.0以后可以自动封包，可以简写成list.add(1)的形式，但添加的数据依然是封装后的对象。 
2. 另外，有些情况下，我们也会编写诸如func(Object o)的这种方法，它可以接受所有类型的对象数据，但对于简单数据类型，我们则必须使用封装类的对象。
- 某些情况下，使用封装类使我们可以更加方便的操作数据。比如封装类具有一些基本类型不具备的方法，比如valueOf(), toString(), 以及方便的返回各种类型数据的方法，如Integer的shortValue(), longValue(), intValue()等。


基本数据类型与其对应的封装类由于本质的不同，具有一些区别：
- 基本数据类型只能按值传递，而封装类按引用传递。
- 基本类型在堆栈中创建；而对于对象类型，对象在堆中创建，对象的引用在堆栈中创建。基本类型由于在堆栈中，效率会比较高，但是可能会存在内存泄漏的问题。

#### Switch能否用string做参数？

在 Java 7之前，switch 只能支持 byte、short、char、int或者其对应的封装类以及 Enum 类型。在 Java 7中，String支持被加上了。
```java

switch (ctrType)
	{
    case "01" :  
        exceptionType = "读FC参数数据"; 
        break; 
    case "03" : 
        exceptionType = "读FC保存的当前表计数据"; 
        break; 
    default: 
        exceptionType = "未知控制码："+ctrType; 
    }
```
其中ctrType为字符串。
　　如在jdk 7 之前的版本使用, 会提示如下错误：
　　Cannot switch on a value of type String for source level below 1.7. Only convertible int values or enum variables are permitted
　　意为jdk版本太低，不支持。

#### equals与==的区别

总之：
“==”比较的是值【变量(栈)内存中存放的对象的(堆)内存地址】
equal用于比较两个对象的值是否相同【不是比地址】


【特别注意】Object类中的** equals**方法和** 双等号**是一样的，没有区别，而String类，Integer类等等一些类，是重写了equals方法，才使得equals和“=="不同。

所以，当自己创建类时，自动继承了Object的equals方法，要想实现不同的等于比较，必须重写equals方法。

** 双等比equal运行速度快,因为"=="只是比较引用.**

#### Object有哪些公用方法？

- protected Object clone()
 创建并返回此对象的一个副本。
- boolean equals(Object obj) 
 指示其他某个对象是否与此对象“相等”。 
- protected void finalize() 
 当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。 
- Class<?> getClass() 
 返回此 Object 的运行时类。 
- int hashCode() 
 返回该对象的哈希码值。 
- void notify() 
 唤醒在此对象监视器上等待的单个线程。 
- void notifyAll() 
 唤醒在此对象监视器上等待的所有线程。 
- String toString() 
 返回该对象的字符串表示。 
- void wait() 
 在其他线程调用此对象的 notify() 方法或 notifyAll() 方法前，导致当前线程等待。 
- void wait(long timeout) 
 在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量前，导致当前线程等待。 
 void wait(long timeout, int nanos) 

#### Java中四种引用（强、软、弱、虚）
##### 强引用(StrongReference)
强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。


##### 软引用(SoftReference)
如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。


##### 弱引用(WeakReference)
弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。


##### 虚引用(PhantomReference)
"虚引用"顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。

虚引用主要用来跟踪对象被垃圾回收器回收的活动。虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之 关联的引用队列中。

##### 总结
WeakReference与SoftReference都可以用来保存对象的实例引用，这两个类与垃圾回收有关。
WeakReference是弱引用，其中保存的对象实例可以被GC回收掉。这个类通常用于在某处保存对象引用，而又不干扰该对象被GC回收，通常用于Debug、内存监视工具等程序中。因为这类程序一般要求即要观察到对象，又不能影响该对象正常的GC过程。
最近在JDK的Proxy类的实现代码中也发现了Weakrefrence的应用，Proxy会把动态生成的Class实例暂存于一个由Weakrefrence构成的Map中作为Cache。

SoftReference是强引用，它保存的对象实例，除非JVM即将OutOfMemory，否则不会被GC回收。这个特性使得它特别适合设计对象Cache。对于Cache，我们希望被缓存的对象最好始终常驻内存，但是如果JVM内存吃紧，为了不发生OutOfMemoryError导致系统崩溃，必要的时候也允许JVM回收Cache的内存，待后续合适的时机再把数据重新Load到Cache中。这样可以系统设计得更具弹性。

#### Hashcode的作用
HashCode的官方文档定义：
>hashcode方法返回该对象的哈希码值。支持该方法是为哈希表提供一些优点，例如，java.util.Hashtable 提供的哈希表。   
  
>hashCode 的常规协定是：   
>在 Java 应用程序执行期间，在同一对象上多次调用 hashCode 方法时，必须一致地返回相同的整数，前提是对象上 equals 比较中所用的信息没有被修改。从某一应用程序的一次执行到同一应用程序的另一次执行，该整数无需保持一致。   
>
如果根据 equals(Object) 方法，两个对象是相等的，那么在两个对象中的每个对象上调用 hashCode 方法都必须生成相同的整数结果。   
>
以下情况不 是必需的：如果根据 equals(java.lang.Object) 方法，两个对象不相等，那么在两个对象中的任一对象上调用 hashCode 方法必定会生成不同的整数结果。但是，程序员应该知道，为不相等的对象生成不同整数结果可以提高哈希表的性能。  
>
实际上，由 Object 类定义的 hashCode 方法确实会针对不同的对象返回不同的整数。（这一般是通过将该对象的内部地址转换成一个整数来实现的，但是 JavaTM 编程语言不需要这种实现技巧。）   
当equals方法被重写时，通常有必要重写 hashCode 方法，以维护 hashCode 方法的常规协定，该协定声明相等对象必须具有相等的哈希码。  

以上这段官方文档的定义，我们可以抽出成以下几个关键点：
1. hashCode的存在主要是用于查找的快捷性，如Hashtable，HashMap等，hashCode是用来在散列存储结构中确定对象的存储地址的；
2. 如果两个对象相同，就是适用于equals(java.lang.Object) 方法，那么这两个对象的hashCode一定要相同；
3. 如果对象的equals方法被重写，那么对象的hashCode也尽量重写，并且产生hashCode使用的对象，一定要和equals方法中使用的一致，否则就会违反上面提到的第2点；
4. 两个对象的hashCode相同，并不一定表示两个对象就相同，也就是不一定适用于equals(java.lang.Object) 方法，只能够说明这两个对象在散列存储结构中，如Hashtable，他们“存放在同一个篮子里”。

再归纳一下就是hashCode是用于查找使用的，而equals是用于比较两个对象的是否相等的。以下这段话是从别人帖子回复拷贝过来的：

>1.hashcode是用来查找的，如果你学过数据结构就应该知道，在查找和排序这一章有  
例如内存中有这样的位置  
0  1  2  3  4  5  6  7    
>
而我有个类，这个类有个字段叫ID,我要把这个类存放在以上8个位置之一，如果不用hashcode而任意存放，那么当查找时就需要到这八个位置里挨个去找，或者用二分法一类的算法。  
但如果用hashcode那就会使效率提高很多。  
>
我们这个类中有个字段叫ID,那么我们就定义我们的hashcode为ID％8，然后把我们的类存放在取得得余数那个位置。比如我们的ID为9，9除8的余数为1，那么我们就把该类存在1这个位置，如果ID是13，求得的余数是5，那么我们就把该类放在5这个位置。这样，以后在查找该类时就可以通过ID除 8求余数直接找到存放的位置了
2.但是如果两个类有相同的hashcode怎么办那（我们假设上面的类的ID不是唯一的），例如9除以8和17除以8的余数都是1，那么这是不是合法的，回答是：可以这样。那么如何判断呢？在这个时候就需要定义 equals了。  
>
也就是说，我们先通过 hashcode来判断两个类是否存放某个桶里，但这个桶里可能有很多类，那么我们就需要再通过 equals 来在这个桶里找到我们要的类。  
>
那么。重写了equals()，为什么还要重写hashCode()呢？  
想想，你要在一个桶里找东西，你必须先要找到这个桶啊，你不通过重写hashcode()来找到桶，光重写equals()有什么用啊  


#### ArrayList、LinkedList、Vector的区别
- ArrayList 是一个可改变大小的数组.当更多的元素加入到ArrayList中时,其大小将会动态地增长.内部的元素可以直接通过get与set方法进行访问,因为ArrayList本质上就是一个数组.
- LinkedList 是一个双链表,在添加和删除元素时具有比ArrayList更好的性能.但在get与set方面弱于ArrayList.
当然,这些对比都是指数据量很大或者操作很频繁的情况下的对比,如果数据和运算量很小,那么对比将失去意义.
- Vector 和ArrayList类似,但属于强同步类。如果你的程序本身是线程安全的(thread-safe,没有在多个线程之间共享同一个集合/对象),那么使用ArrayList是更好的选择。
- Vector和ArrayList在更多元素添加进来时会请求更大的空间。Vector每次请求其大小的双倍空间，而ArrayList每次对size增长50%.
- 而 LinkedList 还实现了 Queue 接口,该接口比List提供了更多的方法,包括 offer(),peek(),poll()等.
注意: 默认情况下ArrayList的初始容量非常小,所以如果可以预估数据量的话,分配一个较大的初始值属于最佳实践,这样可以减少调整大小的开销。


# Final
final是java的关键字，它所表示的是“这部分是无法修改的”。不想被改变的原因有两个：效率、设计。使用到final的有三种情况：数据、方法、类。

>基本数据类型不可变的是其内容，而引用数据类型不可变的是** 其引用**，引用所指定的对象内容是可变的。

Final对象里的内容是可变的。