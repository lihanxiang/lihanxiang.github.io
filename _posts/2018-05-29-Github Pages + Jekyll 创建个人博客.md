---

layout:		post
title:		Github Pages + Jekyll 创建个人博客
subtitle:	随便说说
date:		2018-05-29
author:		L
header-img:	img/post/2018-05/29.jpg
tags:
    - 随笔
---

### 正文之前

> 在去年的时候接触到了Github，同时也了解到了Github Pages，这是Github为每位用户提供的一个能够作为个人主页的仓库，上个月才开始搭建个人仓库，应网友的请求，写一篇搭建个人博客的过程，最后当然要打一波[我的博客](http://lihanxiang.github.io)的广告了
>
> 关于题目中说到的Jekyll，是一个**静态博客网站生成器**，也就是说，用MarkDown写完的文章，经过你选定的Jekyll模板的转换，就能够生成面向公众的静态网页了，而且Jekyll集成了服务器，支持本地预览

### 正文

接下来说重点：

#### 1. 创建仓库

既然是每个用户只能有一个主页，所以仓库的名字肯定是要特殊化的：

**用户名.github.io**

我创建了一个小号来做这个示范：

![](https://upload-images.jianshu.io/upload_images/3426615-e5467e8c90dd40fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>

#### 2. Jekyll工具

准备一个Linux系统，来使用Jekyll工具，我用的是虚拟机里的Ubuntu系统

具体的Jekyll下载过程：

1. 需要配置按照官网给出的环境：

![](https://upload-images.jianshu.io/upload_images/3426615-66ffcbe2e43e423f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 安装Jekyll：

Ubuntu镜像应该自带上述环境，我直接进行了jekyll的安装：

```
gem install jekyll
```

![](https://upload-images.jianshu.io/upload_images/3426615-1d32d8018bbac443.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 挑选模板

在jekyll的[模板目录](http://jekyllthemes.org/)中挑选自己喜欢的，并找到模板的GitHub地址，这里要将一点，推荐选用那些有空白模板的，免得有很多需要修改的信息

我们以模板中的第一个模板为例

![](https://upload-images.jianshu.io/upload_images/3426615-f844cd7e83e470a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

找到模板的源码位置：

![](https://upload-images.jianshu.io/upload_images/3426615-9256e30599245e31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>

#### 3. 本地编辑

把这个模板仓库和你刚才创建的博客仓库下载到本地，并且认真阅读模板项目中的README文档，文档里有关于模板的基础配置，一些特点以及博客文章的规范等信息

具体的操作试模板而定，我是直接把模板里的所有文件（除了README文档）全部复制到我的仓库里，简单粗暴，然后push
<br>

#### 4. Jekyll目录

官网给出的基本的目录是这样的：

![](https://upload-images.jianshu.io/upload_images/3426615-b10e1e6cb5f80e02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我以我的博客目录来解释一下这些文件夹：

* _config.yml

是核心的配置文件，里面包含了你的博客的基本信息，网页的地址，其他的链接以及MarkDown格式等信息

* _includes

是一些可重用的组件，比如Github、知乎和简书的链接，谷歌分析的代码等等

* _layouts

是你博客网页的布局

* _posts

这里存放要发表的文章，也就是需要被转换成**.html**的**.md**文件

* _site

这个文件夹是存放Jekyll转换完成的页面，官方推荐最好放入.gitignore 文件中，以免被他人查看

其他没有提到的是我没有使用到的，当然我使用的博客模板中还有一些如**css**, **js**, **img**, **font**之类的目录，这些都视模板而定
<br>

#### 5. 发布文章

* 文章标题

年-月-日-标题.MARKUP，比如当前这篇文章就是：

**2018-05-28-Github Pages + Jekyll 创建个人博客.md**

* YAML头信息

每篇文章都需要YAML头信息，具体的信息是由你的模板决定的

* Jekyll转换

把添加了头信息和修改了标题的文章放至_posts文件夹中，然后在博客仓库内使用 **jekyll serve** 命令：

![](https://upload-images.jianshu.io/upload_images/3426615-82e2bd8f8ed4cf47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

运行之后可以在本地预览博客内容，端口为**http://localhost:4000**：

![](https://upload-images.jianshu.io/upload_images/3426615-9dd037396fef0e4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

预览完成之后，用git命令来进行add和commit，并且push本地仓库，过几分钟登陆博客地址，就能够看到内容更新了：

![](https://upload-images.jianshu.io/upload_images/3426615-d68cdbe54832bfbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> git add
> git commit
> git pull
> git push

这四条命令就够了，博客的基础搭建就这样结束了，祝大家都能挑选到心仪的模板，最后再次抛出[我的博客](http://lihanxiang.github.io)

<p><img src="https://upload-images.jianshu.io/upload_images/3426615-79919768b5365766.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" alt="" width="40%" height="40%" /></p>
