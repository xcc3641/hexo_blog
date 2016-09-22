title: Java 基础 —— 多线程
date: 2016-09-21 22:51:32
---

多线程对于 Android 开发者来说是基础。而且这类知识在计算机里也是很重要的一环，所以很有必要整理一番。

<!-- more -->


#### 多线程的实现

来上代码：

```Java
// 最常见的两种方法启动新的线程
public static void startThread() {
    // 覆盖 run 方法
    new Thread() {
        @Override
        public void run() {
            // 耗时操作
        }
    }.start();

    // 传入 Runnable 对象
    new Thread(new Runnable() {
        public void run() {
            // 耗时操作
        }
    }).start();
}
```

其实第一个就是在 Thread 里覆写了 `run()` 函数，第二个是给 Thread 传了一个 Runnable 对象，在 Runnable 对象 `run()` 方法里进行耗时操作。
以前没有怎么考虑过他们两者的关系，今天我们来具体看看到底是什么鬼？

##### Thread 源码

进入 Thread 源码我们看看：

```Java
public class Thread implements Runnable {
    /* What will be run. */
    private Runnable target;

    /* The group of this thread */
    private ThreadGroup group;


    public Thread() {
        init(null, null, "Thread-" + nextThreadNum(), 0);
    }
    public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
    }
}
```

源码很长，我进行了一点分割。一点一点的来解析看看。
我们首先知道 Thread 也是一个 Runnable ，它实现了 Runnable 接口，并且在 Thread 类中有一个 Runnable 类型的 target 对象。

构造方法里我们都会调用 `init()` 方法，接下来看看在该方法里做了如何的初始化配置。

```Java
    private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize) {
    init(g, target, name, stackSize, null);
    }

    private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc) {

    Thread parent = currentThread();
    SecurityManager security = System.getSecurityManager();
    // group 参数如果为 null ，则获得当前线程的 group（线程组）
    if (g == null) {
            g = parent.getThreadGroup();
    }
    // 代码省略

    this.group = g;
    this.daemon = parent.isDaemon();
    this.priority = parent.getPriority();
    // 设置 target 
    this.target = target;

    }


    public synchronized void start() {


    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {

          }
        }
    }


```
