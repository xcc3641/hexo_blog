title: Python 初探 leancloud
date: 2016-04-27 09:03:58
categories: 技术
tags: 
- Python
- Leancloud
---

很久没有写 Python 了，自己阿里云的服务器也忘记续费而被释放。当初用的东西有点重，Ubuntu 下定时任务执行一个 Python 爬虫，将爬取到的数据写入 MySQL ，接口是 Java 那一套 tomcat 。

所以准备写个系列，从简单的使用 leancloud 的存储到 leancloud 的云引擎。

<!-- more -->


### 0x01 准备

##### 安装 virtualenv

使用 [virtualenv](https://virtualenv.pypa.io/) 可以创建一个与系统隔离的 Python 环境，在其中安装的第三方模块版本不会与系统自带的或者其他项目中的模块冲突。

安装 `virtualenv` > `pip install virtualenv`

#####  创建虚拟并激活虚拟环境

```shell
cd yourProject
virtualenv ENV(自取)
source bin/activate
```

命令行上出现 (ENV) 表示是在当前虚拟环境下

使用 `deactivate` 可以退出虚拟环境

##### 安装 leancloud-sdk

```
pip install leancloud-sdk
```



### 0x02 数据存储

##### 初始化 leancloud

```python
import leancloud
from leancloud import Object
leancloud.init('id', 'key')
```

id 和 key 在 **项目** > **设置** >**应用 Key** 中查看。

##### 存储数据

```python
class GirlImage(Object):
    @property
    def src(self):
        # 可以使用property装饰器，方便获取属性
        return self.get('src')

    @src.setter
    def src(self, value):
        # 同样的，可以给对象的score增加setter
        return self.set('src', value)


girl_image = GirlImage()
girl_image.set('src',"地址")
girl_image.save()
```

执行完之后，你会在项目**存储**>**数据**下看到已经新建好了 GirlImage 这个表并且保存了一条`src = 地址` 的数据。



##### 爬取存储

```python
def http():
    page = 1
    url = 'http://www.dbmeinv.com/dbgroup/current.htm?gid=haixiuzu&pager_offset=' + str(page)
    headers = {'content-type': 'text/html;charset=UTF-8',
               'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.87 Safari/537.36 QQBrowser/9.2.4907.400'}
    content = requests.post(url, headers).text
    images_codes = re.findall('<img class="height_min" .+? src="(.*?)"', content, re.S)

    # print images_codes
    for url in images_codes:
        girl_image = GirlImage()
        girl_image.set("src", url)
        girl_image.save()
```

一个简单的小爬虫，爬了该网站一页图片，20条存入 leancloud 。



##### 查询

```python

query = Query(GirlImage)  # 这里也可以直接传递一个 Class 名字的字符串作为构造参数
query.select('src')
reslut = query.find()

for i in reslut:
    print(i.get('src'))
```



### 0x03

初探就先这样结束了，我这里只是运用了很简单的功能。

不过爬取>存储>查询，都写全了。

后期我会再探 leancloud 写增量爬取加 leancloud 的云引擎。



##### 更多请参考

[leancloud 对象操作](https://leancloud.cn/docs/python_guide.html#对象)

[leancloud 查询操作](https://leancloud.cn/docs/python_guide.html#查询)

