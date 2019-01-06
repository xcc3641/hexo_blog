title: 一次简单地封装 SharedPreferences
date: 2016-03-12 20:31:17
---

有一段时间开发，经常用 SharedPreferences 所以可以想尝试下简单封装，让 SharedPreferences 更易于开发使用。

<!-- more -->

### 单一

```java
    public SPUtil() {
        throw new AssertionError("no instance");
    }

    private static class Holder {
        private static SharedPreferences sp = App.getContext().getSharedPreferences(C.DEFAULT, Activity.MODE_PRIVATE);
    }

    private static SharedPreferences getSP() {
        return Holder.sp;
    }
```

工具类本身不需要构造，内部类里实例化一个生命周期是 Application 的 SharedPreferences 对象。

内部使用调用 `getSP()` 方法即可。


统一方法（我这里只写了两个）：

```java
    public static void putInt(String key, int value) {
        getSP().edit().putInt(key, value).apply();
    }

    public static int getInt(String key, int defValue) {
        return getSP().getInt(key, defValue);
    }

    public static void putString(String key, String value) {
        getSP().edit().putString(key, value).apply();
    }

    public static String getString(String key, String defValue) {
        return getSP().getString(key, defValue);
    }
```

### 具体使用

```java
    public static void setUuid(String uuid) {
        putString("uuid", uuid);
    }

    public static String getUuid() {
        return getString("uuid", "");
    }
```

上层只需要使用 `SPUtil.get/setUuid()` 即可对该 `Value` 进行获取和设置。

- - - 


> 如果有更好方便的思路，请留言告知交流谢谢