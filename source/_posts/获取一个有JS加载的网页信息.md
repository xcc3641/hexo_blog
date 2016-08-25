title: 获取一个有JS加载的网页信息
date: 2015-12-26 20:02:24
categories: 技术
tags: 
	- Java
	- Python
---


现在很多网页内容是通过JS加载出来的，而不是静态不变的。本文以[网易摄影](http://pp.163.com/pp/#p=10&c=-1&m=3&page=1)为例子分 Java 和 Python 两种语言爬取图片信息。

<!-- more -->
### Java
Java 找了很久找到了 Htmlunit 网络请求库，主要的特点就是模拟浏览器运行。

```java
//初始化
WebClient wc = new WebClient(BrowserVersion.CHROME);
```

浏览器种类可更改。

```java
//相关配置。
wc.getOptions().setUseInsecureSSL(true);
wc.getOptions().setJavaScriptEnabled(true); // 启用JS解释器，默认为true
wc.getOptions().setCssEnabled(false); // 禁用css支持
wc.getOptions().setThrowExceptionOnScriptError(false); // js运行错误时，是否抛出异常
wc.getOptions().setTimeout(100000); // 设置连接超时时间 ，这里是10S。如果为0，则无限期wc.getOptions().setDoNotTrackEnabled(false);
wc.getOptions().setThrowExceptionOnFailingStatusCode(false);
```
```
HtmlPage page = wc.getPage("http://pp.163.com/pp/#p=10&c=-1&m=3&page=1");
// 该方法在getPage()方法之后调用才能生效
wc.waitForBackgroundJavaScript(1000 * 3);
```

等待网页JS代码运行后我们才能得到我们需要的数据。

```java
/**
* <div class="pic"> <a class="img js-anchor etag noul" hidefocus="true"
* target="_blank" href="http://pp.163.com/oneness/pp/16356042.html"
* title="登登与小猫笨笨 ">
*/

//div[@class="pic"]/a/@href

List<DomAttr> links = (List<DomAttr>)page.getByXPath("//div[@class=\"pic\"]/a/@href");
```

我这里用得是 XPath 进行匹配数据。看下简单的实例，就蛮容易上手的。得到之后，剩下的事情就很简单了。

但是 Htmlunit 进行爬取成功率会有问题，偶尔会有一两次访问不到，不过我暂时没有在 Java 里找到更好的解决方案。

### Python

Python 写这个代码有点早，是用的 Spynner 库，是一个操控一个无 GUI 的 Webkit 核心实现 http访 问的 Python 模块，用来爬使用 js 运行才有结果的网页最好。

```python
//初始化
browser = spynner.Browser()
browser.create_webview()
```
然后调用
```python
browser.load(url=url, tries=5)
browser.load_jquery(True)

content = browser.html.encode("utf-8")
```

这样 JS 运行后的页面就可以获取到了，通过 bs4 或者 re 正则都可以获取想要的东西。

最后还是放一张图：


![](http://xcc3641.qiniudn.com/blog75c3c8a451cc91ad59f2f9d2cbd386cf_b.jpg)

