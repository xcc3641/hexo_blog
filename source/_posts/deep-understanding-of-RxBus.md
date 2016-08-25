title: 从 RxBus 这辆兰博基尼深入进去
coverCaption: ''
date: 2016-06-02 20:24:41
categories: 技术
tags:
- Android
- RxJava
- RxBus
thumbnailImage: http://ww2.sinaimg.cn/large/a17a2d22gw1f4h6dufx62j20go0afwf6.jpg
thumbnailImagePosition: right
---

很早之前有看过别人实现的 RxBus , 当初也只是随意用用而已，没有想过去研究。今天看到 brucezz 天哥在群里分享了一把，自己也加入了讨论，下来还实践了一把，所以想借此篇进入到源码层，深刻体验下 RxBus 这辆 “兰博基尼” 的设计美感和独特魅力。

**本篇文章已授权微信公众号 guolin_blog （郭霖）独家发布**

<!-- more -->




- [RxBus](#Rxbus)
	- [准备](#准备)
	- [解剖](#解剖)
- [从 Subject 开始发车](#从-Subject-开始发车)
	- [官方解释](#官方解释)
	- [Subject 源码](#Subject-源码)
	- [PublishSubject 解释](#PublishSubject-解释)
	- [串行化](#串行化)
	- [SerializedSubject](#SerializedSubject)
	- [SerializedObserver](#SerializedObserver)
	- [NotificationLite](#NotificationLite)
	- [CompositeSubscription](#CompositeSubscription)
	- [参考文章](#参考文章)
- [熄火休息](#熄火休息)



### RxBus

#### 准备

关于简单的实现和用法，这篇文章已经很好的说明了。

推荐先看看 RxBus 的简单实现和用法。

地址在这里：[RxBus 的简单实现](http://brucezz.github.io/articles/2016/06/02/a-simple-rxbus-implementation/)

#### 解剖

![](http://ww3.sinaimg.cn/large/a17a2d22gw1f4h6davxw6j20m80gotbq.jpg)

- - -


让我们看看这辆车到底用了些什么？


- Subject

- SerializedSubject

- PublishSubject

- CompositeSubscription




### 从 Subject 开始发车

#### 官方解释

这是 Subject 的中文解释：

Subject可以看成是一个桥梁或者代理，在某些ReactiveX实现中（如RxJava），它同时充当了Observer和Observable的角色。因为它是一个Observer，它可以订阅一个或多个Observable；又因为它是一个Observable，它可以转发它收到(Observe)的数据，也可以发射新的数据。

由于一个Subject订阅一个Observable，它可以触发这个Observable开始发射数据（如果那个Observable是"冷"的--就是说，它等待有订阅才开始发射数据）。因此有这样的效果，Subject可以把原来那个"冷"的Observable变成"热"的。


#### Subject 源码
源码：

```Java

public abstract class Subject<T, R> extends Observable<R> implements Observer<T> {
    protected Subject(OnSubscribe<R> onSubscribe) {
        super(onSubscribe);
    }

    public abstract boolean hasObservers();

    public final SerializedSubject<T, R> toSerialized() {
        if (getClass() == SerializedSubject.class) {
            return (SerializedSubject<T, R>)this;
        }
        return new SerializedSubject<T, R>(this);
    }
```

Subject 只有两个方法。

`hasObservers()`方法的解释是:

>Indicates whether the {@link Subject} has {@link Observer Observers} subscribed to it.
判断 Subject 是否已经有 observers 订阅了 有则返回 ture



`toSerialized()` 方法的解释是：

>Wraps a {@link Subject} so that it is safe to call its various {@code on} methods from different threads.
包装 Subject 后让它可以安全的在不同线程中调用各种方法

**为什么这个方法后就可以是线程安全了呢？**

我们看到 `toSerialized()` 返回了 `SerializedSubject<T, R>` 。我们先到这里打住，稍后我们再看看该类做了什么。


#### PublishSubject 解释

![](http://ww4.sinaimg.cn/large/a17a2d22gw1f4h8xwycjdj20ye0k2q5b.jpg)

在 RxJava 里有一个抽象类 Subject，既是 Observable 又是 Observer，可以把 Subject 理解成一个管道或者转发器，数据从一端输入，然后从另一端输出。

Subject 有好几种，这里可以使用最简单的 `PublishSubject`。订阅之后，一旦数据从一端传入，结果会里立刻从另一端输出。

源码里给了用法例子：

```Java
  PublishSubject<Object> subject = PublishSubject.create();
  // observer1 will receive all onNext and onCompleted events
  subject.subscribe(observer1);
  subject.onNext("one");
  subject.onNext("two");
  // observer2 will only receive "three" and onCompleted
  subject.subscribe(observer2);
  subject.onNext("three");
  subject.onCompleted();
```


#### 串行化

官方文档推荐我们：

>如果你把 Subject 当作一个 Subscriber 使用，注意不要从多个线程中调用它的onNext方法（包括其它的on系列方法），这可能导致同时（非顺序）调用，这会违反Observable协议，给Subject的结果增加了不确定性。

>要避免此类问题，你可以将 Subject 转换为一个 SerializedSubject ，类似于这样：
```java
mySafeSubject = new SerializedSubject( myUnsafeSubject );
```

所以我们可以看到在 RxBus 初始化的时候我们做了这样一件事情：

```java
    private final Subject<Object, Object> BUS;

    private RxBus() {
        BUS = new SerializedSubject<>(PublishSubject.create());
    }
```

为了保证多线程的调用中结果的确定性，我们按照官方推荐将 Subject 转换成了一个 SerializedSubject 。


#### SerializedSubject

![](http://ww4.sinaimg.cn/large/a17a2d22gw1f4i5h2l0anj20ww0nujuy.jpg)

该类同样是 `Subject` 的子类，这里贴出该类的构造方法。

```java
    private final SerializedObserver<T> observer;
    private final Subject<T, R> actual;

    public SerializedSubject(final Subject<T, R> actual) {
        super(new OnSubscribe<R>() {

            @Override
            public void call(Subscriber<? super R> child) {
                actual.unsafeSubscribe(child);
            }

        });
        this.actual = actual;
        this.observer = new SerializedObserver<T>(actual);
    }
```

我们发现，`Subject` 最后转化成了 `SerializedObserver`.

#### SerializedObserver

> When multiple threads are emitting and/or notifying they will be serialized by:
Allowing only one thread at a time to emit
Adding notifications to a queue if another thread is already emitting
Not holding any locks or blocking any threads while emitting

>一次只会允许一个线程进行发送事物
如果其他线程已经准备就绪，会通知给队列
在发送事物中，不会持有任何锁和阻塞任何线程

![](http://ww1.sinaimg.cn/large/a17a2d22gw1f4i6cogy7ej20yo0r6gpa.jpg)

通过介绍可以知道是通过 `notifications` 来进行并发处理的。

SerializedObserver 类中
`private final NotificationLite<T> nl = NotificationLite.instance();`

重点看看 nl 在 onNext() 方法里的使用：

```java

 @Override
    public void onNext(T t) {
   // 省略一些代码
        for (;;) {
            for (int i = 0; i < MAX_DRAIN_ITERATION; i++) {
                FastList list;
                synchronized (this) {
                    list = queue;
                    if (list == null) {
                        emitting = false;
                        return;
                    }
                    queue = null;
                }
                for (Object o : list.array) {
                    if (o == null) {
                        break;
                    }
                    // 这里的 accept() 方法
                    try {
                        if (nl.accept(actual, o)) {
                            terminated = true;
                            return;
                        }
                    } catch (Throwable e) {
                        terminated = true;
                        Exceptions.throwIfFatal(e);
                        actual.onError(OnErrorThrowable.addValueAsLastCause(e, t));
                        return;
                    }
                }
            }
        }
    }
```

#### NotificationLite

知道哪里具体调用了之后，我们再仔细看看  `NotificationLite` 。

先来了解它到底是什么：

> For use in internal operators that need something like materialize and dematerialize wholly within the implementation of the operator but don't want to incur the allocation cost of actually creating {@link rx.Notification} objects for every {@link Observer#onNext onNext} and {@link Observer#onCompleted onCompleted}.
> It's implemented as a singleton to maintain some semblance of type safety that is completely non-existent.

> 大致意思是：作为一个单例类保持这种完全不存在的安全类型的表象。  

![](http://ww4.sinaimg.cn/large/a17a2d22gw1f4i7btoa64j21ei0qygs6.jpg)

刚我们在 SerializedObserver 的 onNext() 方法中看到 `nl.accept(actual, o)`
所以我们再深入到 `accept()` 方法中：

```java
   public boolean accept(Observer<? super T> o, Object n) {
        if (n == ON_COMPLETED_SENTINEL) {
            o.onCompleted();
            return true;
        } else if (n == ON_NEXT_NULL_SENTINEL) {
            o.onNext(null);
            return false;
        } else if (n != null) {
            if (n.getClass() == OnErrorSentinel.class) {
                o.onError(((OnErrorSentinel) n).e);
                return true;
            }
            o.onNext((T) n);
            return false;
        } else {
            throw new IllegalArgumentException("The lite notification can not be null");
        }
    }
```

>Unwraps the lite notification and calls the appropriate method on the {@link Observer}.
判断 lite 通知类别，通知 observer 执行适当方法。


通过 NotificationLite 类图可以看到有三个标识

- ON_NEXT_NULL_SENTINEL （onNext 标识）
- ON_COMPLETED_SENTINEL （onCompleted 标识)
- OnErrorSentinel (onError 标识)

与 Observer 回调一致。通过分析得知 `accept()`  就是通过标识来判断，然后调用 Observer 相对应的方法。



#### CompositeSubscription

RxBus 这辆"兰博基尼"与 CompositeSubscription 车间搭配更好。

- - -


![](http://ww4.sinaimg.cn/large/a17a2d22gw1f4ijb9l4zwj20v80ps42f.jpg)

构造函数：

```java
    private Set<Subscription> subscriptions;
    private volatile boolean unsubscribed;

    public CompositeSubscription() {
    }

    public CompositeSubscription(final Subscription... subscriptions) {
        this.subscriptions = new HashSet<Subscription>(Arrays.asList(subscriptions));
    }
```

内部是初始化了一个 HashSet ，按照哈希算法来存取集合中的对象，存取速度比较快，并且没有重复对象。

所以我们推荐在基类里实例化一个 CompositeSubscription 对象，使用 CompositeSubscription 来持有所有的 Subscriptions ，然后在 onDestroy()或者 onDestroyView()里取消所有的订阅。


#### 参考文章

- <http://blog.csdn.net/lzyzsd/article/details/45033611>
- <https://mcxiaoke.gitbooks.io/rxdocs/content/Subject.html>

### 熄火休息

能力有限，文章错误还望指出，有任何问题都欢迎讨论 :)

转载请注明出处。

最后送上我女神 Gakki , 开心最好 ( ´͈ｖ `͈ )◞。

![](http://ww3.sinaimg.cn/large/a17a2d22gw1f4ijpqofquj20fi0gowgo.jpg)
