title: Android Studio 在 El Capitan 中切换闪屏问题
date: 2016-05-07 20:41:11
categories: 工具
tags: Android Studio
---

买来 RMBP 有一段时间了，确实用来写代码很爽和快乐。但是还是遇到一些小坑，所以决定记录一下。
我是 Android 开发爱好者，在使用 Android Studio 的时候全屏模式下，切换会闪屏。虽然是小问题，但是特别难受，自己谷歌了很多方法，终于相互结合后，已经成功解决。
<!-- excerpt -->

### 方案

1. 先在官网下载 jdk 1.8
2. 在 Android Studio 安装路径下 修改`info.plist`中`JVMversion`为 1.8*
3. 同时下载最新版本的 IDEA ，将它自身带有的 JAR 包拷贝到 Android Studio 对应位置
4. 提高 Android Studio Gradle 内存，在 studio.vmoptions 修改
