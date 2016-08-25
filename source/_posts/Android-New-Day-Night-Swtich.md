title: 一次 Android Day/Night 实践（夜间模式）
coverCaption: ''
date: 2016-07-12 10:09:39
categories: 技术
tags: Android
thumbnailImage:
thumbnailImagePosition:
metaAlignment:
coverImage:
coverMeta:
---

用过 就看天气 的话，都会发现，它在 18：00 的时候会自动切换成夜间模式，当时使用的方法是在创建视图的时候就判断当前手机时间，再进行一些 UI 的改变。这样的做法不是特别的 cool ，无意之间发现 Android 23.2.0 的包下提供了 日/夜间 主题，所以立马就加入 就看天气 ，让我们来看看如何使用吧。

<!-- more -->

#### 主题配置

首先你得先在你的 `style.xml`  文件里，把你的主题继承 DayNight 主题，像这样：

```xml
<style name="AppTheme"
        parent="Theme.AppCompat.DayNight.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
</style>
```

它有四个可选值，分别是：
- MODE_NIGHT_NO： 使用亮色(light)主题，不使用夜间模式
- MODE_NIGHT_YES：使用暗色(dark)主题，使用夜间模式
- MODE_NIGHT_AUTO：根据当前时间自动切换 亮色(light)/暗色(dark)主题
- MODE_NIGHT_FOLLOW_SYSTEM(默认选项)：设置为跟随系统，通常为 MODE_NIGHT_NO

推荐在 Application 添加一个静态代码块来进行初始化全局设置：

```java
static {
    AppCompatDelegate.setDefaultNightMode(
        AppCompatDelegate.MODE_NIGHT_AUTO); // 选择你需要的
}
```


#### 切换主题时使用自定义资源

使用自定义资源，只需要在 res 目录下创建对应的 values-night 文件夹并创建对应的 themes.xml 文件：


##### res/values/colors.xml

```xml
    <color name="colorPrimary">#106c99</color>
    <color name="colorPrimaryDark">#0D4A6A</color>
    <color name="colorAccent">#FC413D</color>
```

##### res/values-night/colors.xml

```xml
    <color name="colorPrimary">#201D45</color>
    <color name="colorPrimaryDark">#201D45</color>
```

同理其他资源你只需要在末尾添加 `-night` 系统就会自动加载对应的文件了。

- - -

![](http://ww2.sinaimg.cn/large/006tNbRwgw1f5umljz70uj30zo1iqn2z.jpg)

![](http://ww1.sinaimg.cn/large/006tNbRwgw1f5umrv1sbpj30zo1iqjwv.jpg)


#### 参考文章

- [Android Support Library v23.2 Adds Auto-Switching Day/Night Theme, Support For Vector Drawables, And Much More](http://www.androidpolice.com/2016/02/24/android-support-library-v23-2-adds-auto-switching-daynight-theme-support-for-vector-drawables-and-much-more/)

- [AppCompat v23.2 - 夜间模式最佳实践](https://kingideayou.github.io/2016/03/07/appcompat_23.2_day_night/)
