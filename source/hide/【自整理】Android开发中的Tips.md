title: 【自整理】Android开发中的Tips
date: 2015-12-17 19:24:00
categories: 技术
tags: Android
---

![](http://xcc3641.qiniudn.com/blogandroid.jpg)

本文是为了记录自己开发过程中遇到的坑和写过的可复用代码块。
updata: 2015.12.17

<!-- more -->
## 代码部分
### 双击退出
```
public boolean onKeyDown(int keyCode, KeyEvent event) {
	if (keyCode == KeyEvent.KEYCODE_BACK && event.getRepeatCount() == 0) {
		if ((System.currentTimeMillis() - exitTime) > 2000) {
			Toast.makeText(this, "再按一次退出程序", Toast.LENGTH_SHORT).show();
			exitTime = System.currentTimeMillis();
           	} 
		else {
            finish();
			}
        }
	return false;
    }
```
### 获取时间

```
 Calendar c = Calendar.getInstance();
 c.add(Calendar.DATE, -1);//获取昨天
 time = new SimpleDateFormat("yyyyMMdd").format(c.getTime());
```
[^ 最开始的时候自己不知道Calendar有这个方法，自己写获取昨天实在麻烦，因为需要考虑跨月和跨年]

```Java
    /**
     * 判断当前日期是星期几
     *
     * @param pTime 修要判断的时间
     * @return dayForWeek 判断结果
     * @Exception 发生异常
     */
public static int dayForWeek(String pTime) throws Exception {
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        Calendar c = Calendar.getInstance();
        c.setTime(format.parse(pTime));
        int dayForWeek = 0;
        if(c.get(Calendar.DAY_OF_WEEK) == 1){
            dayForWeek = 7;
        }else{
            dayForWeek = c.get(Calendar.DAY_OF_WEEK) - 1;
        }
        return dayForWeek;
    }
```
### 检测是否已经连接到网络
```
//需要权限
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" /> 
```

只关注是否联网：
```
public boolean isNetworkConnected(Context context) {
	if (context != null) {
		ConnectivityManager mConnectivityManager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
		NetworkInfo mNetworkInfo = mConnectivityManager.getActiveNetworkInfo();
	if (mNetworkInfo != null) {
		return mNetworkInfo.isAvailable();
		}
	}
    return false;
}
```
WIFI或者Internet：
```
ConnectivityManager con=(ConnectivityManager)getSystemService(Activity.CONNECTIVITY_SERVICE);
boolean wifi=con.getNetworkInfo(ConnectivityManager.TYPE_WIFI).isConnectedOrConnecting();
boolean internet=con.getNetworkInfo(ConnectivityManager.TYPE_MOBILE).isConnectedOrConnecting();
```

### 发送短信
```
public static void sendSms(Context context, String phoneNumber,
            String content) {
        Uri uri = Uri.parse("smsto:"
                + (TextUtils.isEmpty(phoneNumber) ? "" : phoneNumber));
        Intent intent = new Intent(Intent.ACTION_SENDTO, uri);
        intent.putExtra("sms_body", TextUtils.isEmpty(content) ? "" : content);
        context.startActivity(intent);
    }
```
### 唤醒屏幕并解锁

```
public static void wakeUpAndUnlock(Context context){  
        KeyguardManager km= (KeyguardManager) context.getSystemService(Context.KEYGUARD_SERVICE);  
        KeyguardManager.KeyguardLock kl = km.newKeyguardLock("unLock");  
        //解锁
        kl.disableKeyguard(); 
        //获取电源管理器对象
        PowerManager pm=(PowerManager) context.getSystemService(Context.POWER_SERVICE);  
        //获取PowerManager.WakeLock对象,后面的参数|表示同时传入两个值,最后的是LogCat里用的Tag  
        PowerManager.WakeLock wl = pm.newWakeLock(PowerManager.ACQUIRE_CAUSES_WAKEUP | PowerManager.SCREEN_DIM_WAKE_LOCK,"bright");  
        //点亮屏幕  
        wl.acquire();  
        //释放  
        wl.release();  
    }  
```
需要添加权限：
```
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.DISABLE_KEYGUARD" />
```
### 判断当前设备是否为手机
```
 public static boolean isPhone(Context context) {
        TelephonyManager telephony = (TelephonyManager) context
                .getSystemService(Context.TELEPHONY_SERVICE);
        if (telephony.getPhoneType() == TelephonyManager.PHONE_TYPE_NONE) {
            return false;
        } else {
            return true;
        }
    }
```
### 获取当前设备的IMEI，与上面的isPhone()一起使用
```
@TargetApi(Build.VERSION_CODES.CUPCAKE)
    public static String getDeviceIMEI(Context context) {
        String deviceId;
        if (isPhone(context)) {
            TelephonyManager telephony = (TelephonyManager) context
                    .getSystemService(Context.TELEPHONY_SERVICE);
            deviceId = telephony.getDeviceId();
        } else {
            deviceId = Settings.Secure.getString(context.getContentResolver(),
                    Settings.Secure.ANDROID_ID);

        }
        return deviceId;
    }
```
### 获取当前程序的版本号
```
public static String getAppVersion(Context context) {
        String version = "0";
        try {
            version = context.getPackageManager().getPackageInfo(
                    context.getPackageName(), 0).versionName;
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
        }
        return version;
    }
```

### 获取当前设备宽高，单位px
```
  public static int getDeviceWidth(Context context) {
        WindowManager manager = (WindowManager) context
                .getSystemService(Context.WINDOW_SERVICE);
        return manager.getDefaultDisplay().getWidth();
    }


    public static int getDeviceHeight(Context context) {
        WindowManager manager = (WindowManager) context
                .getSystemService(Context.WINDOW_SERVICE);
        return manager.getDefaultDisplay().getHeight();
    }
```
### Retrofit 2.0
```
//baseUrl="http://example/SaveImg/"
public interface imgApi {

	@GET("DouBanGirl")
	Call<Img> getImg(@Query("date") String date);
}
//假设传入date=20150101
```
最后实际访问的地址是：http://example/SaveImg/DouBanGirl?date=20150101
这个地址拼接最开始的时候坑了自己很久，官方也没有仔细讲明白。

### Jsoup
自己还是习惯Python里的正则或者bs4进行匹配，Java里目前只接触了Jsoup，每次都要去查阅用法。最近查看Jsoup源码的时候好像发现Jsoup也是可以用正则匹配的。
顺便说下当发现Android.text下的**HTML.fromHtml()**方法的时候，简直开心。
```
//<img class="xx" title="xx" alt="xx" onerror="img_error(this);" src="http://ww2.sinaimg.cn/bmiddle/xxx.jpg">
//目标是img标签中的src图片地址

Document doc = Jsoup.parse(content);
Elements elements = doc.select("img[src$=.jpg]");
for (Element src : elements) {
	Log.d(TAG, src.attr("abs:src"));
｝
```

### 圆角+点击变色（非监听）Button
Android中常常使用shape来定义控件的一些显示属性，今天看了一些shape的使用，对shape有了大体的了解，稍作总结：
在android的project中新建一个drawable文件夹（此文件夹位于res下），存放定义了shape的**.xml文件，然后在其他xml文件中引用，如，android:background=@drawable/****.xml
```xml
<shape>
<!-- 实心 -->
<solid android:color="#ff9d77"/>
<!-- 渐变 -->
<gradient
android:startColor="#ff8c00"
android:endColor="#FFFFFF"
android:angle="270" />
<!-- 描边 -->
<stroke
android:width="2dp"
android:color="#dcdcdc" />
<!-- 圆角 -->
<corners
android:radius="2dp" />
<!-- padding：Button里面的文字与Button边界的间隔 -->
<padding
android:left="10dp"
android:top="10dp"
android:right="10dp"
android:bottom="10dp" />
</shape>
```
组合起来
```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="false">
        <shape>
            <!-- 填充的颜色 -->
            <solid android:color="@color/loginButtonColor"/>
            <!-- 设置按钮的四个角为弧形 -->
            <!-- android:radius 弧形的半径 -->
            <corners android:radius="8dp"/>

            <!-- padding：Button里面的文字与Button边界的间隔 -->
            <padding android:left="10dp"
                android:top="10dp"
                android:right="10dp"
                android:bottom="10dp"/>

        </shape>

    </item>
    <item android:state_pressed="true">
        <shape>
            <!-- 填充的颜色 -->
            <solid android:color="@color/loginButtonColor"/>
            <!-- 设置按钮的四个角为弧形 -->
            <!-- android:radius 弧形的半径 -->
            <corners android:radius="8dp"/>

            <!-- padding：Button里面的文字与Button边界的间隔 -->
            <padding android:left="10dp"
                android:top="10dp"
                android:right="10dp"
                android:bottom="10dp"/>
        </shape>

    </item>

</selector>
```

### DP和PX

```java
public class DensityUtil {

    /**
     * 根据手机的分辨率从 dp 的单位 转成为 px(像素)
     */
    public static int dip2px(Context context, float dpValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (dpValue * scale + 0.5f);
    }


    /**
     * 根据手机的分辨率从 px(像素) 的单位 转成为 dp
     */
    public static int px2dip(Context context, float pxValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (pxValue / scale + 0.5f);
    }
}
```
### 高斯模糊（配合Glide效果拔群）
```java
public class BlurImageview {
    /** 水平方向模糊度 */
    private static float hRadius = 25;
    /** 竖直方向模糊度 */
    private static float vRadius = 25;
    /** 模糊迭代度 */
    private static int iterations = 25;


    /**
     * 图片高斯模糊处理 
     */
    public static Drawable BlurImages(Bitmap bmp, Context context) {

        int width = bmp.getWidth();
        int height = bmp.getHeight();
        int[] inPixels = new int[width * height];
        int[] outPixels = new int[width * height];
        Bitmap bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
        bmp.getPixels(inPixels, 0, width, 0, 0, width, height);
        for (int i = 0; i < iterations; i++) {
            blur(inPixels, outPixels, width, height, hRadius);
            blur(outPixels, inPixels, height, width, vRadius);
        }
        blurFractional(inPixels, outPixels, width, height, hRadius);
        blurFractional(outPixels, inPixels, height, width, vRadius);
        bitmap.setPixels(inPixels, 0, width, 0, 0, width, height);
        Drawable drawable = new BitmapDrawable(context.getResources(), bitmap);
        return drawable;
    }


    /**
     * 图片高斯模糊算法
     */
    public static void blur(int[] in, int[] out, int width, int height, float radius) {
        int widthMinus1 = width - 1;
        int r = (int) radius;
        int tableSize = 2 * r + 1;
        int divide[] = new int[256 * tableSize];

        for (int i = 0; i < 256 * tableSize; i++) {
            divide[i] = i / tableSize;
        }

        int inIndex = 0;

        for (int y = 0; y < height; y++) {
            int outIndex = y;
            int ta = 0, tr = 0, tg = 0, tb = 0;

            for (int i = -r; i <= r; i++) {
                int rgb = in[inIndex + clamp(i, 0, width - 1)];
                ta += (rgb >> 24) & 0xff;
                tr += (rgb >> 16) & 0xff;
                tg += (rgb >> 8) & 0xff;
                tb += rgb & 0xff;
            }

            for (int x = 0; x < width; x++) {
                out[outIndex] = (divide[ta] << 24) | (divide[tr] << 16) | (divide[tg] << 8) | divide[tb];

                int i1 = x + r + 1;
                if (i1 > widthMinus1) i1 = widthMinus1;
                int i2 = x - r;
                if (i2 < 0) i2 = 0;
                int rgb1 = in[inIndex + i1];
                int rgb2 = in[inIndex + i2];

                ta += ((rgb1 >> 24) & 0xff) - ((rgb2 >> 24) & 0xff);
                tr += ((rgb1 & 0xff0000) - (rgb2 & 0xff0000)) >> 16;
                tg += ((rgb1 & 0xff00) - (rgb2 & 0xff00)) >> 8;
                tb += (rgb1 & 0xff) - (rgb2 & 0xff);
                outIndex += height;
            }
            inIndex += width;
        }
    }


    /**
     * 图片高斯模糊算法 
     */
    public static void blurFractional(int[] in, int[] out, int width, int height, float radius) {
        radius -= (int) radius;
        float f = 1.0f / (1 + 2 * radius);
        int inIndex = 0;

        for (int y = 0; y < height; y++) {
            int outIndex = y;

            out[outIndex] = in[0];
            outIndex += height;
            for (int x = 1; x < width - 1; x++) {
                int i = inIndex + x;
                int rgb1 = in[i - 1];
                int rgb2 = in[i];
                int rgb3 = in[i + 1];

                int a1 = (rgb1 >> 24) & 0xff;
                int r1 = (rgb1 >> 16) & 0xff;
                int g1 = (rgb1 >> 8) & 0xff;
                int b1 = rgb1 & 0xff;
                int a2 = (rgb2 >> 24) & 0xff;
                int r2 = (rgb2 >> 16) & 0xff;
                int g2 = (rgb2 >> 8) & 0xff;
                int b2 = rgb2 & 0xff;
                int a3 = (rgb3 >> 24) & 0xff;
                int r3 = (rgb3 >> 16) & 0xff;
                int g3 = (rgb3 >> 8) & 0xff;
                int b3 = rgb3 & 0xff;
                a1 = a2 + (int) ((a1 + a3) * radius);
                r1 = r2 + (int) ((r1 + r3) * radius);
                g1 = g2 + (int) ((g1 + g3) * radius);
                b1 = b2 + (int) ((b1 + b3) * radius);
                a1 *= f;
                r1 *= f;
                g1 *= f;
                b1 *= f;
                out[outIndex] = (a1 << 24) | (r1 << 16) | (g1 << 8) | b1;
                outIndex += height;
            }
            out[outIndex] = in[width - 1];
            inIndex += width;
        }
    }


    public static int clamp(int x, int a, int b) {
        return (x < a) ? a : (x > b) ? b : x;
    }
}
```
#### 如何使用？
```java
        Glide.with(MainActivity.this)
		.load(R.drawable.back)
		.asBitmap()
		.into(new SimpleTarget<Bitmap>() {
            @Override public void onResourceReady(Bitmap resource, GlideAnimation<? super Bitmap> glideAnimation) {
                mImageView.setBackground(BlurImageview.BlurImages(resource, MainActivity.this));
            }
        });
```


