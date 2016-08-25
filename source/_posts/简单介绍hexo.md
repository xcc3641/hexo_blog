title: Hexo 常用命令以及配置
date: 2015-11-15 09:32:01
categories: 技术
tags: Hexo
---
Hexo 常用命令以及配置

<!-- more -->

#### 命令
##### 常用命令
```
 hexo new "postName" #新建文章
 hexo new page "pageName" #新建页面
 hexo generate #生成静态页面至public目录
 hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
 hexo deploy #将.deploy目录部署到GitHub
```
##### 常用复合命令
```
 hexo deploy -g
 hexo server -g
```
##### 简写
```
 hexo n == hexo new
 hexo g == hexo generate
 hexo s == hexo server
 hexo d == hexo deploy
```

#### 目录内容
##### 默认目录结构
```
.
├── .deploy
├── public
├── scaffolds
├── scripts
├── source
|   ├── _drafts
|   └── _posts
├── themes
├── _config.yml
└── package.json

```
.deploy：执行hexo deploy命令部署到GitHub上的内容目录
public：执行hexo generate命令，输出的静态网页内容目录
scaffolds：layout模板文件目录，其中的md文件可以添加编辑
scripts：扩展脚本目录，这里可以自定义一些javascript脚本
source：文章源码目录，该目录下的markdown和html文件均会被hexo处理。该页面对应repo的根目录，404文件、favicon.ico文件，CNAME文件等都应该放这里，该目录下可新建页面目录。

_drafts：草稿文章
_posts：发布文章
themes：主题文件目录
_config.yml：全局配置文件，大多数的设置都在这里
package.json：应用程序数据，指明hexo的版本等信息，类似于一般软件中的关于按钮

接下来是重头戏_config.yml，做个简单说明：
```
# Site #整站的基本信息
title: IM XIE #网站标题
subtitle: 代码如诗 #网站副标题
description: #网站描述，给搜索引擎用的，在生成html中的head->meta中可看到
author: Hugo Xie #网站作者，在下方显示
email: Hugo3641@gmail.com #联系邮箱
language: zh-Hans #语言

# URL #域名和文件结构
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://ibruce.info #你的域名
root: /
permalink: :year/:month/:day/:title/
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code

# Writing #写文章选项
new_post_name: :title.md # File name of new posts
default_layout: post #默认layout方式
auto_spacing: false # Add spaces between asian characters and western characters
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
max_open_file: 100
multi_thread: true
filename_case: 0
render_drafts: false
highlight: #代码高亮
  enable: true #是否启用
  line_number: false #是否显示行号
  tab_replace:

# Category & Tag #分类与标签
default_category: uncategorized # default
category_map:
tag_map:

# Archives #存档，这里的说明好像不对。全部选择1，这个选项与主题中的选项有时候会有冲突
## 2: Enable pagination
## 1: Disable pagination
## 0: Fully Disable
archive: 1
category: 1
tag: 1

# Server #本地服务参数
## Hexo uses Connect as a server
## You can customize the logger format as defined in
## http://www.senchalabs.org/connect/logger.html
port: 4000
logger: true
logger_format:

# Date / Time format #日期显示格式
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: MMM D YYYY
time_format: H:mm:ss

# Pagination #分页设置
## Set per_page to 0 to disable pagination
per_page: 10 #每页10篇文章
pagination_dir: page

# Disqus #社会化评论disqus，我使用多说，在主题中配置
disqus_shortname:

# Extensions #插件，暂时未安装插件
## Plugins: https://github.com/tommy351/hexo/wiki/Plugins
## Themes: https://github.com/tommy351/hexo/wiki/Themes
## 主题
theme: modernist # raytaylorism # pacman # modernist # light
exclude_generator:

# Deployment #部署
## Docs: http://zespia.tw/hexo/docs/deploy.html
deploy:
  type: git
  repository: https://github.com/xcc3641/xcc3641.github.io
  branch: master #你的GitHub Pages仓库
```

##### 修改局部页面
页面展现的全部逻辑都在每个主题中控制，源代码在hexo\themes\你使用的主题\中，以next主题为例：
```
.
├── languages          #多语言
|   ├── default.yml    #默认语言
|   └── zh-Hans.yml    #中文语言
├── layout             #布局，根目录下的*.ejs文件是对主页，分页，存档等的控制
|   ├── _partial       #局部的布局，此目录下的*.ejs是对头尾等局部的控制
|   └── _widget        #小挂件的布局，页面下方小挂件的控制
├── source             #源码
|   ├── css            #css源码 
|   |   ├── _base      #*.styl基础css
|   |   ├── _partial   #*.styl局部css
|   |   ├── fonts      #字体
|   |   ├── images     #图片
|   |   └── style.styl #*.styl引入需要的css源码
|   ├── fancybox       #fancybox效果源码
|   └── js             #javascript源代码
├── _config.yml        #主题配置文件
└── README.md          #用GitHub的都知道
```
如果你需要修改头部，直接修改hexo\themes\modernist\layout\_partial\header.ejs，比如头上加个搜索框：
```
<div>
<form class="search" action="//google.com/search" method="get" accept-charset="utf-8">
 <input type="search" name="q" id="search" autocomplete="off" autocorrect="off" autocapitalize="off" maxlength="20" placeholder="Search" />
 <input type="hidden" name="q" value="site:<%- config.url.replace(/^https?:\/\//, '') %>">
</form>
</div>
```
将如上代码加入即可，您需要修改css以便这个搜索框比较美观。

再如，你要修改页脚版权信息，直接编辑hexo\themes\next\layout\_partial\footer.ejs。同理，你需要修改css，直接去修改对应位置的styl文件。

##### “不蒜子”计数插件
普通用户只需两步走：一行脚本+一行标签，搞定一切。追求极致的用户可以进行任意DIY。

要使用不蒜子必须在页面中引入busuanzi.js，目前最新版如下。
```
<script async src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js">
</script>
```
> 不蒜子可以给任何类型的个人站点使用，如果你是用的hexo，打开themes/你的主题/layout/_partial/footer.ejs添加上述脚本即可，当然你也可以添加到 header 中。


**显示站点总访问量**

要显示站点总访问量，复制以下代码添加到你需要显示的位置。有两种算法可选：

算法a：pv的方式，单个用户连续点击n篇文章，记录n次访问量。

```
<span id="busuanzi_container_site_pv">
    本站总访问量<span id="busuanzi_value_site_pv"></span>次
</span>
```

算法b：uv的方式，单个用户连续点击n篇文章，只记录1次访客数。

```
<span id="busuanzi_container_site_uv">
  本站访客数<span id="busuanzi_value_site_uv"></span>人次
</span>
```
> 如果你是用的hexo，打开themes/你的主题/layout/_partial/footer.ejs添加即可。

##### RSS 链接

编辑 主题配置文件，将 rss 字段设置为：

rss: false，这将会禁用Feed链接。
rss:，当值为空的时候，默认会使用站点的 Feed 链接。在此之前需要使用 hexo-generator-feed 插件生成 Feed。


```
# You can configure this plugin in _config.yml.
feed:
  type: atom
  path: atom.xml
  limit: 20
  hub:
```

依照 hexo-generator-feed 插件的安装说明进行 Feed 生成，当配件配置完毕后，主题将自动显示 Feed 链接。


##### 写文章

执行new命令，生成指定名称的文章至`hexo\source\_posts\postName.md`。
```
hexo new [layout] "postName" #新建文章

```
其中layout是可选参数，默认值为post。有哪些layout呢，请到scaffolds目录下查看，这些文件名称就是layout名称。当然你可以添加自己的layout，方法就是添加一个文件即可，同时你也可以编辑现有的layout，比如post的layout默认是hexo\scaffolds\post.md

```
title: {{ title }}
date: {{ date }}
tags:
---
```
我想添加categories，以免每次手工输入，只需要修改这个文件添加一行，如下：
```
title: {{ title }}
date: {{ date }}
categories:
tags:
---
```
postName是md文件的名字，同时也出现在你文章的URL中，postName如果包含空格，必须用”将其包围，postName可以为中文。

##### fancybox

可能有人对这个 Reading 页面中图片的 fancybox 效果感兴趣，这个是怎么做的呢。
很简单，只需要在你的文章*.md文件的头上添加photos项即可，然后一行行添加你要展示的照片：
```
layout: photo
title: xxx
date: 2085-01-16 07:33:44
tags: [hexo]
photos:
- http:xxx.jpg
- http:xxx.jpg
```
【注】经过测试，文件头上的layout: photo可以省略。

不想每次都手动添加怎么办？同样的，打开您的hexo\scaffolds\photo.md

```
layout: { { layout } }
title: { { title } }
date: { { date } }
tags:
photos:
-
---
```
然后每次可以执行带layout的new命令生成照片文章：
```
hexo new photo "photoPostName" #新建照片文章
```


