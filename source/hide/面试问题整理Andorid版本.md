title: 面试问题整理Andorid版本
date: 2015-11-12 21:14:36
categories: 技术
tags: 面试
---
<!-- excerpt -->

### Acitivty的四中启动模式与特点。

- standard：默认的启动模式
- singleTop：适合那种接受通知启动的页面，比如新闻客户端之类的，可能会给你推送好几次 ，但是每次都是打开同一张页面调用onNewIntent
- singleTask：适合作为程序入口点，例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空浏览器主界面上面的其他页面。而之前打开过的页面不再新建
- singleInstance：适合需要与程序分离开的页面。例如闹铃，将闹铃提醒与设置分离，使得闹铃提醒成为系统范围内的唯一实例


Acitivity后台闲置退出或异常退出，如何保存数据

通过 onSaveInstanceState() 和 onRestoreInstanceState() 保存和重启非持久化数据。


### Service的生命周期，两种启动方法，有什么区别

#### Service的第一种启动方式

采用start的方式开启服务

使用Service的步骤：

> 1.定义一个类继承Service
2.在Manifest.xml文件中配置该Service
3.使用Context的startService(Intent)方法启动该Service
4.不再使用时，调用stopService(Intent)方法停止该服务

使用这种start方式启动的Service的生命周期如下：

** onCreate()--->onStartCommand()（onStart()方法已过时） ---> onDestory()**

** 说明：**

如果服务已经开启，不会重复的执行onCreate()， 而是会调用onStart()和onStartCommand()。
服务停止的时候调用 onDestory()。服务只会被停止一次。

** 特点：**

一旦服务开启跟调用者(开启者)就没有任何关系了。
开启者退出了，开启者挂了，服务还在后台长期的运行。
开启者不能调用服务里面的方法。

#### Service的第二种启动方式

采用bind的方式开启服务

使用Service的步骤：

>1.定义一个类继承Service
2.在Manifest.xml文件中配置该Service
3.使用Context的bindService(Intent, ServiceConnection, int)方法启动该Service
4.不再使用时，调用unbindService(ServiceConnection)方法停止该服务

使用这种start方式启动的Service的生命周期如下：

** onCreate() --->onBind()--->onunbind()--->onDestory()**

** 注意：**绑定服务不会调用onstart()或者onstartcommand()方法

** 特点：**bind的方式开启服务，绑定服务，调用者挂了，服务也会跟着挂掉。
绑定者可以调用服务里面的方法。

##### 绑定者如何调用服务里的方法呢？
首先定义一个` Service`的子类。
```Java
public class MyService extends Service {

    public MyService() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        //返回MyBind对象
        return new MyBinder();
    }

    private void methodInMyService() {
        Toast.makeText(getApplicationContext(), "服务里的方法执行了。。。",
                Toast.LENGTH_SHORT).show();
    }

    /**
     * 该类用于在onBind方法执行后返回的对象，
     * 该对象对外提供了该服务里的方法
     */
    private class MyBinder extends Binder implements IMyBinder {

        @Override
        public void invokeMethodInMyService() {
            methodInMyService();
        }
    }
}
```
自定义的` MyBinder`接口用于保护服务中不想让外界访问的方法。
```java
public interface IMyBinder {
     void invokeMethodInMyService();
}
```
接着在` Manifest.xml`文件中配置该Service
`<service android:name=".MyService"/>`
在Activity中绑定并调用服务里的方法
简单布局：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="start"
        android:text="开启服务"
        android:textSize="30sp" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="invoke"
        android:text="调用服务的方法"
        android:textSize="30sp" />
</LinearLayout>
```
绑定服务的Activity：
```java
public class MainActivity extends Activity {

    private MyConn conn;
    private Intent intent;
    private IMyBinder myBinder;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    //开启服务按钮的点击事件
    public void start(View view) {
        intent = new Intent(this, MyService.class);
        conn = new MyConn();
        //绑定服务，
        // 第一个参数是intent对象，表面开启的服务。
        // 第二个参数是绑定服务的监听器
        // 第三个参数一般为BIND_AUTO_CREATE常量，表示自动创建bind
        bindService(intent, conn, BIND_AUTO_CREATE);
    }

    //调用服务方法按钮的点击事件
    public void invoke(View view) {
        myBinder.invokeMethodInMyService();
    }

    private class MyConn implements ServiceConnection {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            //iBinder为服务里面onBind()方法返回的对象，所以可以强转为IMyBinder类型
            myBinder = (IMyBinder) iBinder;
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {
        }
    }
}
```

** 绑定本地服务调用方法的步骤：**

1.在服务的内部创建一个内部类 提供一个方法，可以间接调用服务的方法
2.实现服务的onbind方法，返回的就是这个内部类
3.在activity 绑定服务。bindService();
4.在服务成功绑定的回调方法onServiceConnected， 会传递过来一个 IBinder对象
5.强制类型转化为自定义的接口类型，调用接口里面的方法。

### RxJava&RxAndroid 
RxJava 到底是什么

异步，简洁。

RxJava 在 GitHub 主页上的自我介绍是 "a library for composing asynchronous and event-based programs using observable sequences for the Java VM"（一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库）。这就是 RxJava ，概括得非常精准。

然而，对于初学者来说，这太难看懂了。因为它是一个『总结』，而初学者更需要一个『引言』。

其实， RxJava 的本质可以压缩为异步这一个词。说到根上，它就是一个实现异步操作的库，而别的定语都是基于这之上的。

链式调用 线程切换容易 更加符合思维 不会出现迷之嵌套


其他：**详细阅读扔物线[给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)**




### Intent的使用方法，可以传递哪些数据类型

>参考这篇[文章](http://www.runoob.com/w3cnote/android-tutorial-intent-pass-data.html)

- ** Serializable** :
将 Java 对象序列化为二进制文件的 Java 序列化技术是 Java系列技术中一个较为重要的技术点，在大部分情况下，开发人员只需要了解被序列化的类需要实现 Serializable 接口，使用ObjectInputStream 和 ObjectOutputStream 进行对象的读写。
- ** charsequence**  :
在JDK1.4中，引入了CharSequence接口，实现了这个接口的类有：CharBuffer、String、StringBuffer、StringBuilder这个四个类。
CharBuffer为nio里面用的一个类，String实现这个接口理所当然，StringBuffer也是一个CharSequence，StringBuilder是Java抄袭C#的一个类，基本和StringBuffer类一样，效率高，但是不保证线程安全，在不需要多线程的环境下可以考虑。
提供这么一个接口，有些处理String或者StringBuffer的类就不用重载了。但是这个接口提供的方法有限，只有下面几个：charat、length、subSequence、toString这几个方法，感觉如果有必要，还是重载的比较好，避免用instaneof这个操作符。
- ** Parcelable**  :
android提供了一种新的类型：Parcel。本类被用作封装数据的容器，封装后的数据可以通过Intent或IPC传递。 除了基本类型以
外，只有实现了Parcelable接口的类才能被放入Parcel中。
是GOOGLE在安卓中实现的另一种序列化,功能和Serializable相似,主要是序列化的方式不同
- ** Bundle**:
Bundle是将数据传递到另一个上下文中或保存或回复你自己状态的数据存储方式。它的数据不是持久化状态。


可以直接通过调用Intent的putExtra()方法存入数据，然后在获得Intent后调用getXxxExtra获得 对应类型的数据；传递多个的话，可以使用Bundle对象作为容器，通过调用Bundle的putXxx先将数据 存储到Bundle中，然后调用Intent的putExtras()方法将Bundle存入Intent中，然后获得Intent以后， 调用getExtras()获得Bundle容器，然后调用其getXXX获取对应的数据！ 另外数据存储有点类似于Map的<键，值>！

传递对象的方式有两种：将对象转换为Json字符串或者通过Serializable,Parcelable序列化 不建议使用Android内置的抠脚Json解析器，可使用fastjson或者Gson第三方库！

### ContentProvider使用方法
>参考这篇[文章](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2012/0821/367.html)

因为在Android系统里面，数据库是私有的。一般情况下外部应用程序是没有权限读取其他应用程序的数据。如果你想公开你自己的数据，你有两个选择：你可以创建你自己的内容提供器（一个ContentProvider子类）或者你可以给已有的提供器添加数据-如果存在一个控制同样类型数据的内容提供器且你拥有写的权限。而外界根本看不到，也不用看到这个应用暴露的数据在应用当中是如何存储的，或者是用数据库存储还是用文件存储，还是通过网上获得，这些一切都不重要，重要的是外界可以通过这一套标准及统一的接口和程序里的数据打交道，可以读取程序的数据，也可以删除程序的数据，当然，中间也会涉及一些权限的问题。

### Thread、AsycTask、IntentService的使用场景与特点。
>参考这篇[文章](https://www.zhihu.com/question/30804052)


Android 原生的 AsyncTask.java 是对线程池的一个封装，使用其自定义的 Executor 来调度线程的执行方式（并发还是串行），并使用 Handler 来完成子线程和主线程数据的共享。



预先了解 AsyncTask，必先对线程池有所了解。一般情况下，如果使用子线程去执行一些任务，那么使用 new Thread 的方式会很方便的创建一个线程，如果涉及到主线程和子线程的通信，我们将使用 Handler（一般需要刷新 UI 的适合用到）。如果我们创建大量的（特别是在短时间内，持续的创建生命周期较长的线程）野生线程，往往会出现如下两方面的问题：每个线程的创建与销毁（特别是创建）的资源开销是非常大的；大量的子线程会分享主线程的系统资源，从而会使主线程因资源受限而导致应用性能降低。各位开发一线的前辈们为了解决这个问题，引入了线程池（ThreadPool）的概念，也就是把这些野生的线程圈养起来，统一的管理他们。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

##### 使用线程池的风险？

虽然线程池是构建多线程应用程序的强大机制，但使用它并不是没有风险的。用线程池构建的应用程序容易遭受任何其它多线程应用程序容易遭受的所有并发风险，诸如同步错误和死锁，它还容易遭受特定于线程池的少数其它风险，诸如与池有关的死锁、资源不足和线程泄漏。

- 死锁

	任何多线程应用程序都有死锁风险。当一组进程或线程中的每一个都在等待一个只有该组中另一个进程才能引起的事件时，我们就说这组进程或线程 死锁了。死锁的最简单情形是：线程 A 持有对象 X 的独占锁，并且在等待对象 Y 的锁，而线程 B 持有对象 Y 的独占锁，却在等待对象 X 的锁。除非有某种方法来打破对锁的等待（Java 锁定不支持这种方法），否则死锁的线程将永远等下去。
	
	虽然任何多线程程序中都有死锁的风险，但线程池却引入了另一种死锁可能，在那种情况下，所有池线程都在执行已阻塞的等待队列中另一任务的执行结果的任务，但这一任务却因为没有未被占用的线程而不能运行。当线程池被用来实现涉及许多交互对象的模拟，被模拟的对象可以相互发送查询，这些查询接下来作为排队的任务执行，查询对象又同步等待着响应时，会发生这种情况。
	
- 资源不足

	线程池的一个优点在于：相对于其它替代调度机制（有些我们已经讨论过）而言，它们通常执行得很好。但只有恰当地调整了线程池大小时才是这样的。线程消耗包括内存和其它系统资源在内的大量资源。除了 Thread 对象所需的内存之外，每个线程都需要两个可能很大的执行调用堆栈。除此以外，JVM 可能会为每个 Java 线程创建一个本机线程，这些本机线程将消耗额外的系统资源。最后，虽然线程之间切换的调度开销很小，但如果有很多线程，环境切换也可能严重地影响程序的性能。
	
	如果线程池太大，那么被那些线程消耗的资源可能严重地影响系统性能。在线程之间进行切换将会浪费时间，而且使用超出比您实际需要的线程可能会引起资源匮乏问题，因为池线程正在消耗一些资源，而这些资源可能会被其它任务更有效地利用。除了线程自身所使用的资源以外，服务请求时所做的工作可能需要其它资源，例如 JDBC 连接、套接字或文件。这些也都是有限资源，有太多的并发请求也可能引起失效，例如不能分配 JDBC 连接。
	
- 并发错误

	线程池和其它排队机制依靠使用 wait() 和 notify() 方法，这两个方法都难于使用。如果编码不正确，那么可能丢失通知，导致线程保持空闲状态，尽管队列中有工作要处理。使用这些方法时，必须格外小心；即便是专家也可能在它们上面出错。而最好使用现有的、已经知道能工作的实现，例如 util.concurrent 包。
	
- 线程泄漏

	各种类型的线程池中一个严重的风险是线程泄漏，当从池中除去一个线程以执行一项任务，而在任务完成后该线程却没有返回池时，会发生这种情况。发生线程泄漏的一种情形出现在任务抛出一个 RuntimeException 或一个 Error 时。如果池类没有捕捉到它们，那么线程只会退出而线程池的大小将会永久减少一个。当这种情况发生的次数足够多时，线程池最终就为空，而且系统将停止，因为没有可用的线程来处理任务。
	
	有些任务可能会永远等待某些资源或来自用户的输入，而这些资源又不能保证变得可用，用户可能也已经回家了，诸如此类的任务会永久停止，而这些停止的任务也会引起和线程泄漏同样的问题。如果某个线程被这样一个任务永久地消耗着，那么它实际上就被从池除去了。对于这样的任务，应该要么只给予它们自己的线程，要么只让它们等待有限的时间。
	
	
- 请求过载

	仅仅是请求就压垮了服务器，这种情况是可能的。在这种情形下，我们可能不想将每个到来的请求都排队到我们的工作队列，因为排在队列中等待执行的任务可能会消耗太多的系统资源并引起资源缺乏。在这种情形下决定如何做取决于您自己；在某些情况下，您可以简单地抛弃请求，依靠更高级别的协议稍后重试请求，您也可以用一个指出服务器暂时很忙的响应来拒绝请求。


### Android的数据存储形式


Shared PreferencesStore private primitive data in key-value pairs.

Internal StorageStore private data on the device memory.

External StorageStore public data on the shared external storage.

SQLite DatabasesStore structured data in a private database.

Network ConnectionStore data on the web with your own network server.

Content Provider不能算是一种数据存储方式。它只是给我们提供操作数据的接口，Content Provider背后其实还是SQLite、File I\O等其他方式

### MVC for Android

MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。
其中M层处理数据，业务逻辑等；V层处理界面的显示结果；C层起到桥梁的作用，来控制V层和M层通信以此来达到分离视图显示和业务逻辑层。

1.M层：适合做一些业务逻辑处理，比如数据库存取操作，网络操作，复杂的算法，耗时的任务等都在model层处理。
2.V层：应用层中处理数据显示的部分，XML布局可以视为V层，显示Model层的数据结果。
3.C层：在Android中，Activity处理用户交互问题，因此可以认为Activity是控制器，Activity读取V视图层的数据（eg.读取当前EditText控件的数据），控制用户输入（eg.EditText控件数据的输入），并向Model发送数据请求（eg.发起网络请求等）。

### Android抽象布局——include、merge 、ViewStub
> 参考[文章](http://blog.csdn.net/xyz_lmn/article/details/14524567)


##### ` <include /> `标签能够重用布局文件

- ` <include /> `标签可以使用单独的layout属性，这个也是必须使用的。
- 可以使用其他属性。` <include /> `标签若指定了ID属性，而你的layout也定义了ID，则你的layout的ID会被覆盖，解决方案。
- 在include标签中所有的android:layout_*都是有效的，前提是必须要写layout_width和layout_height两个属性。

##### 减少视图层级` <merge />`

<merge/>标签在UI的结构优化中起着非常重要的作用，它可以删减多余的层级，优化UI。` <merge/>`多用于替换FrameLayout或者当一个布局包含另一个时，` <merge/>`标签消除视图层次结构中多余的视图组。例如你的主布局文件是垂直布局，引入了一个垂直布局的include，这是如果include布局使用的LinearLayout就没意义了，使用的话反而减慢你的UI表现。这时可以使用` <merge/>`标签优化。

##### 需要时使用` <ViewStub />`

` <ViewStub />`标签最大的优点是当你需要时才会加载，使用他并不会影响UI初始化时的性能。各种不常用的布局想进度条、显示错误消息等可以使用` <ViewStub />`标签，以减少内存使用量，加快渲染速度。` <ViewStub />`是一个不可见的，大小为0的View。` <ViewStub />`标签使用如下：

```xml
<ViewStub  
    android:id="@+id/stub_import"  
    android:inflatedId="@+id/panel_import"  
    android:layout="@layout/progress_overlay"  
    android:layout_width="fill_parent"  
    android:layout_height="wrap_content"  
    android:layout_gravity="bottom" />  
```
当你想加载布局时，可以使用下面其中一种方法：

```java
((ViewStub) findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);  
// or  
View importPanel = ((ViewStub) findViewById(R.id.stub_import)).inflate();  
```

当调用inflate()函数的时候，ViewStub被引用的资源替代，并且返回引用的view。 这样程序可以直接得到引用的view而不用再次调用函数findViewById()来查找了。
注：ViewStub目前有个缺陷就是还不支持 <merge /> 标签。

### Json有什么优劣势

优点：
1. 数据格式比较简单，易于读写，格式都是压缩的，占用带宽小，浏览器解析快
2. 易于解析这种语言，客户端JavaScript可以简单的通过eval()进行JSON数据的读取
3. 构造友好，支持多种语言，包括ActionScript， C，C#，ColdFusion，Java，JavaScript，Per，PHP，Python，Ruby等语言服务器端语言，便于服务器端的解析
4. 在PHP世界，已经有PHP-JSON和JSON-PHP出现了，便于PHP序列化后的程序直接调用，PHP服务器端的对象、数组等能够直接生JSON格式，便于客户端的访问提取
5. 因为JSON格式能够直接为服务器端代码使用，大大简化了服务器端和客户端的代码开发量, 但是完成的任务不变, 且易于维护
6.相当稳定。JSON 的附加内容将成为超集

缺点：
1. 没有XML格式这么推广的深入人心和使用广泛，没有XML那么通用性
2. JSON格式目前在Web Service中推广还属于初级阶段

### Asset目录与res目录的区别

1.assets:不会在R.java文件下生成相应的标记，存放到这里的资源在运行打包的时候都会打入程序安装包中

2.res：会在R.java文件下生成标记，这里的资源会在运行打包操作的时候判断哪些被使用到了，没有被使用到的文件资源是不会打包到安装包中的。

在res文件夹下其实还可以定义一下目录：

res/anim:这里存放的是动画资源。

res/xml:可以在Activity中使用getResource().getXML()读取这里的资源文件

res/raw:该目录下的文件可以直接复制到设备上，编译软件时，这里的数据不需要编译，直接加入到程序安装包中，使用方法是getResource().OpenRawResources(ID),其中参数ID的形式是R.raw.XXX.

### 显示Intent和隐式Intent区别

对明确指出了目标组件名称的Intent，我们称之为“显式Intent”。
对于没有明确指出目标组件名称的Intent，则称之为“隐式 Intent”。
对于隐式意图，在定义Activity时，指定一个intent-filter，当一个隐式意图对象被一个意图过滤器进行匹配时，将有三个方面会被参考到：

动作(Action)
类别(Category ['kætɪg(ə)rɪ] )
数据(Data )


### 什么是线程池，线程池的作用是什么

线程池的基本思想还是一种对象池的思想，开辟一块内存空间，里面存放了众多(未死亡)的线程，池中线程执行调度由池管理器来处理。当有线程任务时，从池中取一个，执行完成后线程对象归池，这样可以避免反复创建线程对象所带来的性能开销，节省了系统的资源。就好比原来去食堂打饭是每个人看谁抢的赢，谁先抢到谁先吃，有了线程吃之后，就是排好队形，今天我跟你关系好，你先来吃饭。比如：一个应用要和网络打交道，有很多步骤需要访问网络，为了不阻塞主线程，每个步骤都创建个线程，在线程中和网络交互，用线程池就变的简单，线程池是对线程的一种封装，让线程用起来更加简便，只需要创一个线程池，把这些步骤像任务一样放进线程池，在程序销毁时只要调用线程池的销毁函数即可。
单个线程的弊端：a. 每次new Thread新建对象性能差b. 线程缺乏统一管理，可能无限制新建线程，相互之间竞争，及可能占用过多系统资源导致死机或者OOM,c. 缺乏更多功能，如定时执行、定期执行、线程中断。
java提供的四种线程池的好处在于：a. 重用存在的线程，减少对象创建、消亡的开销，性能佳。b. 可有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞。c. 提供定时执行、定期执行、单线程、并发数控制等功能。
#### Java 线程池
Java通过Executors提供四种线程池，分别为：

newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。

newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。

newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

*** (1). newCachedThreadPool***
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。线程池为无限大，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。

*** (2). newFixedThreadPool***
创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。

*** (3) newScheduledThreadPool***
创建一个定长线程池，支持定时及周期性任务执行。ScheduledExecutorService比Timer更安全，功能更强大

*** (4)、newSingleThreadExecutor***
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行

### IntentService的用法

#### 一、IntentService简介 



IntentService是Service的子类，比普通的Service增加了额外的功能。先看Service本身存在两个问题：  



Service不会专门启动一条单独的进程，Service与它所在应用位于同一个进程中；  



Service也不是专门一条新线程，因此不应该在Service中直接处理耗时的任务；  






#### 二、IntentService特征 



会创建独立的worker线程来处理所有的Intent请求；  



会创建独立的worker线程来处理onHandleIntent()方法实现的代码，无需处理多线程问题；  



所有请求处理完成后，IntentService会自动停止，无需调用stopSelf()方法停止Service；  



为Service的onBind()提供默认实现，返回null；  



为Service的onStartCommand提供默认实现，将请求Intent添加到队列中； 


IntentService不会阻塞UI线程，而普通Serveice会导致ANR异常
Intentservice若未执行完成上一次的任务，将不会新开一个线程，是等待之前的任务完成后，再执行新的任务，等任务完成后再次调用stopSelf


### Handler的实现原理

handler干了些什么：

运行在某个线程上，共享线程的消息队列；

接收消息、调度消息，派发消息和处理消息；

实现消息的异步处理；

建立消息处理模型/系统

[参考博客](http://www.cnblogs.com/bastard/archive/2012/06/08/2541944.html)




### Context与ApplicationContext的区别，分别用在什么情况下

Application的Context是一个全局静态变量，SDK的说明是只有当你引用这个context的生命周期超过了当前activity的生命周期，而和整个应用的生命周期挂钩时，才去使用这个application的context。

在android中context可以作很多操作，但是最主要的功能是加载和访问资源。在android中有两种context，一种是 application context，一种是activity context，通常我们在各种类和方法间传递的是activity context。


### View的绘制流程
1.onmesarue() 为整个View树计算实际的大小


2.onlayout() 为将整个根据子视图的大小以及布局参数将View树放到合适的位置上


3.ondraw() 


1 、绘制该View的背景
2 、为显示渐变框做一些准备操作(见5，大多数情况下，不需要改渐变框)
3、调用onDraw()方法绘制视图本身   (每个View都需要重载该方法，ViewGroup不需要实现该方法)
4、调用dispatchDraw ()方法绘制子视图(如果该View类型不为ViewGroup，即不包含子视图，不需要重载该方法)

### 触摸屏幕的分发机制
##### 1、基础知识
(1) 所有Touch事件都被封装成了MotionEvent对象，包括Touch的位置、时间、历史记录以及第几个手指(多指触摸)等。
(2) 事件类型分为ACTION_DOWN, ACTION_UP, ACTION_MOVE, ACTION_POINTER_DOWN, ACTION_POINTER_UP, ACTION_CANCEL，每个事件都是以ACTION_DOWN开始ACTION_UP结束。
(3) 对事件的处理包括三类，分别为
传递——dispatchTouchEvent()函数、
拦截——onInterceptTouchEvent()函数、
消费——onTouchEvent()函数和OnTouchListener

#### 2、传递流程

(1) 事件从Activity.dispatchTouchEvent()开始传递，只要没有被停止或拦截，从最上层的View(ViewGroup)开始一直往下(子View)传递。子View可以通过onTouchEvent()对事件进行处理。

(2) 事件由父View(ViewGroup)传递给子View，ViewGroup可以通过onInterceptTouchEvent()对事件做拦截，停止其往下传递。

(3) 如果事件从上往下传递过程中一直没有被停止，且最底层子View没有消费事件，事件会反向往上传递，这时父View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会到Activity的onTouchEvent()函数。

(4) 如果View没有对ACTION_DOWN进行消费，之后的其他事件不会传递过来。

(5) OnTouchListener优先于onTouchEvent()对事件进行消费。

