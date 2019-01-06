title: 讨论下 RxJava 里的 onError
coverCaption: ''
date: 2016-06-01 14:12:26
categories: 技术
tags: 
- RxJava
- Java
thumbnailImage:
thumbnailImagePosition:
metaAlignment:
coverImage:
coverMeta:
---

因为上次在网络请求里用到了 `onErrorReturn` 所以想自己实践下和搞清楚 RxJava 里异常处理到底是怎么回事，以及自己如何更好的使用。

<!-- more -->

<!-- toc -->

### 栗子

```Java
public class RxJavaTest {
    static int[] arrayA = new int[2];
    static List<Integer> listB = new ArrayList<>();

    public static void main(String[] args) {
        listB.add(1);
        listB.add(2);
        listB.add(3);
        errorTest();
    }

    private static void errorTest() {
        Observable.from(listB)
            .doOnNext(integer -> arrayA[integer] = integer)
            .subscribe(integer -> {
            }, throwable -> {
                System.out.println("在onError处理了:" + throwable.toString());
            });
    }
}
```

可以看到我在这里模拟了一个越界异常，在 `errorTest()` 方法里做写了遍历列表 `listB` 并且将值赋值给数组 `arrayA` 对应角标。

运行以上程序在控制台输出的是：

```
在onError处理了:java.lang.ArrayIndexOutOfBoundsException: 2
```

现在我们做一点改变：

**在 `errorTest` 方法里加入 `onErrorReturn` 并且返回 null **

```java
    private static void errorTest() {
        Observable.from(listB)
            .doOnNext(integer -> arrayA[integer] = integer)
            .onErrorReturn(throwable -> {
                System.out.println("在onErrorReturn处理了:" + throwable.toString());
                return null;
            })
            .subscribe(integer -> {

            }, throwable -> {
                System.out.println("在onError处理了:" + throwable.toString());
            });
    }
```


现在再运行程序得到在控制台的输出是：

```
在onErrorReturn处理了:java.lang.ArrayIndexOutOfBoundsException: 2
```

发现异常只在 onErrorReturn 处理了，并没有在 onError 里。

难道只要写了 onErrorReturn 处理了异常就不会抛到 onError 里吗？

带着这个疑问，我们再修改下 errorTest() 方法：
```java
    private static void errorTest() {
        Observable.from(listB)
            .doOnNext(integer -> arrayA[integer] = integer)
            .onErrorReturn(throwable -> {
                System.out.println("在onErrorReturn处理了:" + throwable.toString());
                return 3;
            })
            .doOnNext(integer -> arrayA[integer] = integer)
            .subscribe(integer -> {
            }, throwable -> {
                System.out.println("在onError处理了:" + throwable.toString());
            });
    }
```

让 onErrorReturn 处理了异常后并且返回一个 3，随后继续进行赋值操作。

现在运行程序，控制台输出：
```
在onErrorReturn处理了:java.lang.ArrayIndexOutOfBoundsException: 2
在onError处理了:java.lang.ArrayIndexOutOfBoundsException: 3
```

### 总结

**onErrorRetrun** 能够捕获在它之前发生的异常，它之后流中的操作发生的异常就它就不会管了。
经过小栗子的演示，我们现在理解了 onErrorReturn 的作用域，从而明白该什么时候使用它。

参考文章：
- <https://mcxiaoke.gitbooks.io/rxdocs/content/operators/Catch.html>
- <https://dzone.com/articles/rxjavas-side-effect-methods>
- <http://blog.chengyunfeng.com/?p=976>




