title: RxJava + Retrofit 的实际应用场景
date: 2016-05-24 14:38:50
categories: 技术
tags:
- Android
- RxJava
thumbnailImage: http://ww3.sinaimg.cn/large/a17a2d22gw1f46ltvuwx7j21hc0xce81.jpg
thumbnailImagePosition: right
---

关于`RxJava Retrofit`很多篇文章都有详细的说明，在这里我想分享一个具体的使用案例，在我的开源项目 [就看天气](https://github.com/xcc3641/SeeWeather) 里的实际应用。也希望跟大家探讨如何优雅的使用。

<!-- more -->

<!-- toc -->

- [前提](#前提)
- [实战](#实战)
	- [准备](#准备)
	- [组件](#组件)
		- [RxSchedulerHelper](#rxschedulerhelper)
		- [handleResult](#handleresult)
		- [RetrofitSingleton](#retrofitsingleton)
	- [使用](#使用)
- [结语](#结语)

<!-- tocstop -->

## 前提

需要知道什么是 `RxJava`

这里推荐下 扔物线写的 [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)

再感谢 [RxJava 与 Retrofit 结合的最佳实践](http://gank.io/post/56e80c2c677659311bed9841) 这篇满满的干货。


## 实战

### 准备

项目中用到的依赖：

```java
compile 'io.reactivex:rxjava:1.1.0'
compile 'io.reactivex:rxandroid:1.1.0'
compile 'com.google.code.gson:gson:2.4'
compile 'com.squareup.retrofit2:retrofit:2.0.2'
compile 'com.squareup.retrofit2:converter-gson:2.0.2'
compile 'com.squareup.retrofit2:adapter-rxjava:2.0.2'
compile 'com.squareup.okhttp3:okhttp:3.0.1'
compile 'com.squareup.okhttp3:logging-interceptor:3.0.1'
compile 'com.squareup.okio:okio:1.6.0'
```

因为要用到网络，所以千万别忘记了这个权限。
```java
    <uses-permission android:name="android.permission.INTERNET"/>
```

### 组件

Rx 封装的工具

使用`compose`操作符

`compose()`里接收一个`Transformer`对象，`Transformer`继承自`Func1<Observable<T>, Observable<R>>`，可以通过它将一种类型的`Observable`转换成另一种类型的`Observable`。

#### RxSchedulerHelper

封装 Rx 线程相关操作

```
public static <T> Observable.Transformer<T, T> rxSchedulerHelper() {
    return tObservable -> tObservable.subscribeOn(Schedulers.io())
        .unsubscribeOn(AndroidSchedulers.mainThread())
        .observeOn(AndroidSchedulers.mainThread());
}
```

#### handleResult

封装 API 请求后统一处理

```Java
public static <T> Observable.Transformer<Result<T>, T> handleResult() {
    return resultObservable -> resultObservable.flatMap(tResult -> {
        if (tResult.code == 1) {
            return createData(tResult.data);
        } else {
            return Observable.error(new ApiException(tResult.code));
        }
    });
}
```

#### RetrofitSingleton

自己封装了下 `Retrofit`。可以学习下[小艾的方式](https://drakeet.me/retrofit-2-0-okhttp-3-0-config)。

自己将请求是写在该类，使用者只需要关心如何处理拿到的数据和相应的 UI 操作。

```java
public Observable<Weather> fetchWeather(String city) {
    return apiService.mWeatherAPI(city, C.KEY)
        .filter(weatherAPI -> weatherAPI.mHeWeatherDataService30s.get(0).status.equals("ok"))
        .map(weatherAPI -> weatherAPI.mHeWeatherDataService30s.get(0))
        .compose(RxUtils.rxSchedulerHelper());
}

public Observable<VersionAPI> fetchVersion() {
    return apiService.mVersionAPI(C.API_TOKEN).compose(RxUtils.rxSchedulerHelper());
}
```

### 使用

将***网络拉取***和***读取缓存***用 Rx 结合。
这里就要使用 `concat` 操作符，[官方解释](http://reactivex.io/documentation/operators/concat.html).

首先看看获取网络是如何写的：

```java
private Observable<Weather> fetchDataByNetWork() {
    String cityName = Util.replaceCity(mSetting.getCityName());
    return RetrofitSingleton.getInstance()
        .fetchWeather(cityName)
        .onErrorReturn(throwable -> {
            PLog.e(throwable.getMessage());
            return null;
        });
}
```
这里的 onErrorReturn 待会儿说。

再来看看读取缓存的代码：
```java
private Observable<Weather> fetchDataByCache() {
    return Observable.defer(() -> {
        Weather weather = (Weather) aCache.getAsObject(C.WEATHER_CACHE);
        return Observable.just(weather);
    });
}
```

然后我们将他们连接起来：
```java
private void load() {
    Observable.concat(fetchDataByNetWork(), fetchDataByCache())
        .first(weather -> weather != null)
        .doOnError(throwable -> {
            mErroImageView.setVisibility(View.VISIBLE);
            mRecyclerView.setVisibility(View.GONE);
        })
        .doOnNext(weather -> {
            mErroImageView.setVisibility(View.GONE);
            mRecyclerView.setVisibility(View.VISIBLE);
        })
        .doOnTerminate(() -> {
            mRefreshLayout.setRefreshing(false);
            mProgressBar.setVisibility(View.GONE);
        })
        .subscribe(observer);
}
```

concat + first 连接和过滤的操作实现了，网络+缓存的逻辑。

刚刚为什么说要在网络代码那里使用 onErrorReturn 呢？

如果不写，网络发生异常的话，整个流就会直接走 onError ，不会执行到读取缓存的流。

## 结语

Rx 的各种操作符的不同组合就可以实现不同的效果。本身 Rx 封装已经足够好了，我们加工的时候一定要想到是否破坏了他本身的优雅。

因为 Rx 是一种数据流链式结构的编程思想，我们在封装时应该不能打断其链式结构。

欢迎互相讨论和探讨 :)

![](http://ww3.sinaimg.cn/large/a17a2d22gw1f46ltvuwx7j21hc0xce81.jpg)
