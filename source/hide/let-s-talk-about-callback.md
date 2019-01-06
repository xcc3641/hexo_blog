title: "Java 基础复习实践 --- CallBack"
date: 2016-07-23 10:54:27
---

很早之前不知道如何解释清楚回调函数（CallBack），在知乎上看到一个回答特别形象，当时就收藏了，所以今天决定用代码的形式再让这个回答更加深刻点。

[知乎-回调函数（callback）是什么？](https://www.zhihu.com/question/19801131)

然后再看看 Android 里的应用。

<!-- more -->



- [什么是回调函数](#什么是回调函数)
- [Android 里常见的回调](#Android-里常见的回调)



# 什么是回调函数

维基的解释：
> 在计算机程序设计中，回调函数，或简称回调（Callback 即call then back 被主函数调用运算后会返回主函数），是指通过函数参数传递到其它代码的，某一块可执行代码的引用。这一设计允许了底层代码调用在高层定义的子程序。

我认为通俗易懂的解释：
> 你到一个商店买东西，刚好你要的东西没有货，于是你在店员那里留下了你的电话，过了几天店里有货了，店员就打了你的电话，然后你接到电话后就到店里去取了货。在这个例子里，你的电话号码就叫回调函数，你把电话留给店员就叫登记回调函数，店里后来有货了叫做触发了回调关联的事件，店员给你打电话叫做调用回调函数，你到店里去取货叫做响应回调事件。回答完毕。

可能看着通俗的解释能够知道是怎么回事，但是轮到自己要去实现一个回调的时候，就有点寸步难行了。所以我简单将那个解释，转换成了代码再解释一番。

首先我得定义一个接口（我的电话号码）：

```Java
public interface INumber {
	public void onCall();
}
```

我去商店买东西，该商店对象刚刚初始化：
```Java
Store store = new Store();
```
然后我才得知商店刚刚开门没有我要的货（商店构造函数）:
```Java
public class Store {
	public Store() {
		Utils.sout("才开店，没有货");
		Utils.sout("=============");
	}
}
```
所以我把我的电话号码留给店员，对于店员来说就是得到我的电话号码(注册事件)：
```Java
private INumber number;

public void setNumber(INumber number) {
    this.number = number;
}

public void call() {
    number.onCall();
}
/**************注册前的准备工作完毕**************/
public static void main(String[] args) {
    Store store = new Store();
    // 先注册好事件
    store.setNumber(new INumber() {
        @Override
        public void onCall() {
            Utils.sout("=============");
            Utils.sout("货到了");
        }
    });
    for (int i = 1; i < 4; i++) {
        Utils.sout("模拟取货：" + i + "小时");
    }
    // 货到了就打电话通知我
    store.call();
}
```
整体控制台输出：
> 才开店，没有货
= = = = = = = = = = = = =
模拟取货：1小时
模拟取货：2小时
模拟取货：3小时
= = = = = = = = = = = = =
货到了

- - -
商店类的完整代码：
```Java
public class Store {

	public Store() {
		Utils.sout("才开店，没有货");
		Utils.sout("=============");
	}

	private INumber number;

	public void setNumber(INumber number) {
		this.number = number;
	}

	public void call() {
		number.onCall();
	}

	public static void main(String[] args) {

		Store store = new Store();

		store.setNumber(new INumber() {
			@Override
			public void onCall() {
				Utils.sout("=============");
				Utils.sout("货到了");
			}
		});

		for (int i = 1; i < 4; i++) {
			Utils.sout("模拟取货：" + i + "小时");
		}

		store.call();
	}

}
```

# Android 里常见的回调

我们经常这样写：
```Java
btShowDialog = (Button) findViewById(R.id.bt_showdialog);
btShowDialog.setOnClickListener(new View.OnClickListener() {
	@Override
	public void onClick(View v) {
		// do something
	}
});
```
我们经常可以看到这样类似的代码，监听一个按钮的点击事件，当用户点击之后，我们执行一些逻辑。

这里的 `setOnClickListener()` 就跟我们之前店员获得我的电话号码的方法一样 `setNumber()` 注册好事件。
源码：
```Java

/**
 * Register a callback to be invoked when this view is clicked. If this view is not
 * clickable, it becomes clickable.
 *
 * @param l The callback that will run
 *
 * @see #setClickable(boolean)
 */
public void setOnClickListener(@Nullable OnClickListener l) {
	if (!isClickable()) {
		setClickable(true);
	}
	getListenerInfo().mOnClickListener = l;
}

/**
 * Interface definition for a callback to be invoked when a view is clicked.
 */
public interface OnClickListener {
	/**
	 * Called when a view has been clicked.
	 *
	 * @param v The view that was clicked.
	 */
	void onClick(View v);
}
```

到这里的时候是不是就很清晰了呢？
