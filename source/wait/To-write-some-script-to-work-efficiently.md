title: 写点脚本方便开发和工作吧
date: 2016-11-08 23:23:28
---

这里放着的都是自己平时开发过程中觉得很实用的脚本代码。
一直重复操作做一些事情，有点繁琐的对吧。

<!--more-->

### Python

#### 批量安装 Debug Apk
```python
# encoding: utf-8
import os


def extract_lines(content):
    """ Extract lines from terminal response. """
    return content.strip().split('\n')[1:]


def extract_id(line):
    """ Extract device id from line """
    return line.split('\t')[0].strip()


def install(device_id):
    """ Execute adb tasks """
    os.system('adb -s ' + device_id + ' install -r app/build/outputs/apk/app-debug.apk')
    os.system('adb -s '+ device_id +' shell am start -n "com.ruguoapp.jike/com.ruguoapp.jike.business.main.ui.SplashActivity" -a android.intent.action.MAIN -c android.intent.category.LAUNCHER')


map(install, map(extract_id, extract_lines(os.popen('adb devices').read())))
```





### Bash