title: Update Ruby With RVM About Brew Error
coverCaption: ''
date: 2016-07-13 10:51:52
categories: 工具
tags: 
- IOS
- Ruby
thumbnailImage:
thumbnailImagePosition:
metaAlignment:
coverImage:
coverMeta:
---

要运行 IOS 的应用程序，需要安装 [CocoaPods](http://code4app.com/article/cocoapods-install-usage) ，然后又需要 Ruby ，发现安装的时候说 Ruby 版本过低（我 MBP 自带的版本号是 2.0 ==> 2.2.2），所以又得找个 RVM 用来升级 Ruby。

<!-- more -->

# 安装 RVM

> $ curl -L get.rvm.io | bash -s stable

等待之后

>   $ source ~/.bashrc
>   $ source ~/.bash_profile

测试是否正常

>    $ rvm -v


# 升级 Ruby

> $ rvm install 2.2.4

```
Found remote file https://rvm_io.global.ssl.fastly.net/binaries/osx/10.11/x86_64/ruby-2.2.2.tar.bz2
Checking requirements for osx.
ERROR: '/usr/local/Cellar' is not writable - it is required for Homebrew, try 'brew doctor' to fix it!
Requirements installation failed with status: 1.
```

遇到上述错误。

解决方案：

> sudo chown -R $USER /usr/local

再次运行

> rvm install 2.2.2

等待升级完成。

# 安装 CocoaPods

因为GFW，所以我们可以用淘宝的 Ruby 镜像来访问 cocoapods 。按照下面的顺序在终端中敲入依次敲入命令：

> $ gem sources --remove https://rubygems.org/
> //等有反应之后再敲入以下命令
> $ gem sources -a https://ruby.taobao.org/

为了验证你的Ruby镜像是并且仅是taobao，可以用以下命令查看：

> ▶ gem sources -l

出现这个就可以了：

> *** CURRENT SOURCES ***
> https://ruby.taobao.org/

最后进行安装：

>$ sudo gem install cocoapods

