title: Java 基础 —— 多线程（读书笔记）「二」
date: 2016-09-29 16:02:31
---

第二章内容讲讲与多线程相关的方法—— Callable ， Future  和 FutureTask

<!-- more -->
#### 目录



- [目录](#目录)
- [三个类的介绍](#三个类的介绍)
  - [Callable](#Callable)
  - [Future](#Future)
  - [FutureTask](#FutureTask)
- [实例演示](#实例演示)
- [总结与参考](#总结与参考)

#### 三个类的介绍

##### Callable
Callable 与  Runnable 的功能大致相似，不同的是 Callable 是泛型接口，它有一个泛型参数 V ，该接口中有一个返回值「类型为 V 」的 `call()` 函数，而 Runnable 的 `run()` 函数是没有返回参数的。

Callable 的声明如下：
```java
public interface Callable<V> {
    // 泛型返回结果
    V call() throws Exception;
}
```

##### Future

书中比喻 Runnable 与 Callable 就像『脱缰的野马』，无法控制。对于应用开发，我们需要『战马』，Future 就是这类『战马』的标准。

Future 为线程池制定了一个可管理的任务标准。提供了对 Runnable 或者 Callable 任务的执行结果进行取消，查询是否完成，获取结果，设置结果操作。

Future 的声明如下：
```java
public interface Future<V> {

    boolean cancel(boolean mayInterruptIfRunning);
    // 该任务是否被取消
    boolean isCancelled();
    // 该任务是否完成
    boolean isDone();
    // 获取结果，如果任务未完成，则等待，直到完成，因此该函数会阻塞
    V get() throws InterruptedException, ExecutionException;
    // 获取结果，如果任务未完成，则等待，直到 timeout 或者返回结果，该函数会阻塞
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

##### FutureTask

Future 只是定义了一些规范的接口，而 FutureTask 是它的实现类。FutureTask 实现了 RunnableFuture<V> ，而 RunnableFuture 实现了 Runnable, Future<V> ，因此 FutureTask 具备了它们的能力。

```java
public class FutureTask<V> implements RunnableFuture<V> {
    // 代码省略
}
```

RunnableFuture 的定义如下：
```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```

FutureTask 会像 Thread 一样对 Runnable 那样对 Runnable 和 Callable<V> 进行包装，是在 FutureTask 构造函数里注入：

```java

    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }
    
    public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
    }

```
从上面代码我们可以知道，如果传入的是 Runnable 会被 `Executors.callable()` 方法转化成 Callable 类型，可知， FutureTask 最终都是执行 Callable 类型的任务。

我们进入 `Executors.callable()` 函数，看看是如何适配转换的：
```java
    public static <T> Callable<T> callable(Runnable task, T result) {
        if (task == null)
            throw new NullPointerException();
        return new RunnableAdapter<T>(task, result);
    }
    // Runnable 的适配器，将 Runnable 转换成 Callable 
    static final class RunnableAdapter<T> implements Callable<T> {
        final Runnable task;
        final T result;
        RunnableAdapter(Runnable task, T result) {
            this.task = task;
            this.result = result;
        }
        public T call() {
            task.run();
            return result;
        }
    }
```

因为 FutureTask 实现了 Runnable 接口，因此它可以通过 Thread 包装执行，也可以提交给 ExecuteService 来执行。
并且还可以直接通过[ `get()` 方法](#Future)获取执行结果，该方法会阻塞，直到结果返回。
所以可以这样理解 __FutureTask__ ： 既是 Runnable 又是 Future ，同时也包装了 Callable (如果是 Runnable 最终会转化成 Callable )。

#### 实例演示

```java
public class FutureDemo {
	// 线程池
	static ExecutorService mExecutorService = Executors.newSingleThreadExecutor();

	public static void main(String[] args) {
		try {
			futureWithRunnable();
			futureWithCallable();
			futureTask();
			
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
	}

	// 向线程池中提交 Runnable 对象
	private static void futureWithRunnable() throws InterruptedException, ExecutionException {
		Future<?> rFuture = mExecutorService.submit(new Runnable() {

			@Override
			public void run() {
				fibc(20);
			}
		});
		System.out.println("future result from runnable : " + rFuture.get());
	}

	// 提交 Callable 对象，
	private static void futureWithCallable() throws InterruptedException, ExecutionException {
		Future<Integer> rFuture2 = mExecutorService.submit(new Callable<Integer>() {

			@Override
			public Integer call() throws Exception {

				return fibc(20);
			}
		});

		System.out.println("future result from Callable : " + rFuture2.get());

	}

	// 提交 FutureTask 对象
	private static void futureTask() throws InterruptedException, ExecutionException {
		FutureTask<Integer> mTask = new FutureTask<>(new Callable<Integer>() {

			@Override
			public Integer call() throws Exception {

				return fibc(20);
			}
		});
		mExecutorService.submit(mTask);

		System.out.println("future result from futureTask : " + mTask.get());

	}

	// 斐波那契数列 模拟耗时操作
	private static int fibc(int num) {
		if (num == 0) {
			return 0;
		}
		if (num == 1) {
			return 1;
		}

		return fibc(num - 1) + fibc(num - 2);
	}

}
```

运行之后，控制台的输出如下：
> future result from runnable : null
> future result from Callable : 6765
> future result from futureTask : 6765

`futureWithRunnable()` 方法中提交了一个 Runnable 对象，在 `run()` 方法里直接操作计算，这个方法没有返回值，所以 Future 对象 `get()` 拿到的值是 Null 。

`futureWithCallable()` 方法里提交的是一个 Callable 对象
```java
    <T> Future<T> submit(Callable<T> task);
```
Callable 实现的是 `V call()` 方法，最后会返回一个 Future 对象。

`futureTask()`  则是一个  RunnableFuture<V> 既实现了 Runnable 又实现了 Future<V> 这两个接口。提交给 ExecutorService 执行后，可以通过返回的 Future 对象 `get()` 方法得到执行结果。

#### 总结与参考

本章内容是作为铺垫，了解和实践了会用到线程池中的三个类。之前我很少接触到，所以写了这篇文章用来加深印象和笔记翻阅。

- [Android 开发进阶 --- 从小工到专家](https://book.douban.com/subject/26744163/)
