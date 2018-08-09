---



layout:		post



title:		Git 系列文章（一）



subtitle:	GitHub 介绍



date:		2018-08-02



author:		L



header-img:	img/post/2018-08/02.jpg



tags:



    - Git



---

### 正文之前

> 相信大部分都人都知道 GitHub，对，就是被微软收购的全球最大交友网站。目前博主还处于学生阶段，所以一直处于这样的循环中： 
>
> git add<br>
> git commit<br>
> git pull<br>
> git push<br>
> 
> 但是本着要深入学习的心态，以及目前参加[掘金翻译计划](https://github.com/xitu/gold-miner)，翻译的工作都是在 GitHub 上进行的，所以打算从头开始写 Git 系列文章来温故知新一下
>
> 那么这第一篇文章就是介绍一下 Git 和 GitHub，下一篇会用一个例子来讲述 GitHub 上进行协同开发的基本流程

### 正文之前

#### Git

先介绍一下什么是 Git，引用维基百科的介绍：

> Git 是一个分布式版本控制软件，最初由 Linus Torvalds 创作（对，就是那个英语不是他母语的林纳斯），于 2005 年以 GPL 发布。最初是为了更好地管理 Linux 内核开发而设计。

在上面的一段话中，有一个需要解释一下的概念 —— 分布式版本控制：

分布式版本控制是版本控制的方式之一，顾名思义，分布式就代表协同开发者不需要连接同一个网络，一个项目可以由许多国家的程序员来完成

关于 Git 这个名字，来自英国俚语，意思大约是“混账”

具体的 Git 操作等到下一篇讲到协作开发时会说明

#### GitHub

GitHub 是一个通过 Git 进行版本控制的软件源代码托管服务网站，说白了，就是一个网站，里面放着许多用 Git 进行版本控制的项目。因为它有着社交网站的功能 —— watch, star, fork, follow 等功能，也被戏称作“全球最大（同性）交友网站。不过我更喜欢把它当作一个社区，在 GitHub 里有一些很有意思的项目，也有一些很好用的开源软件。

关于 GitHub 账户的创建过程不讲述，付费用户能够拥有私有 (private) 仓库的使用权，学生可以免费使用私有仓库

这里有几个关于 GitHub 的功能是需要了解一下的：

##### 1. fork
这里拿 spring 的仓库来截图说明一下：

![](https://upload-images.jianshu.io/upload_images/3426615-31bccd70465091bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

fork 就表示在你的名下创建一个项目的副本，如果你想要学习一下某一个项目，或者想对别人的项目做一些改动时，都需要先将项目 fork 到自己的账户中，然后再进行进一步的操作

##### 2.  Issues 和 Pull requests

![](https://upload-images.jianshu.io/upload_images/3426615-48bd341e3737a835.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Issue 的目的是为了给开发者提供意见或建议的，开发者可以根据 Issues 中出现的内容来对项目进行改动。但是现在的 Issues 好像变成了讨论区，尤其是前一段事件的 Issues 出现“求不要更新了，老子学不动了”，真是丢脸

Pull requests（简称 PR）是你在 fork 别人的仓库之后，如果有改动想要通知开发者，就发起一个 PR，开发者审核之后，如果开发者同意了这个 PR，就会将 PR 合并到项目中去，你也算为这个项目做了贡献

##### 3. Wiki

Wiki 是一种在网络上开放且可供多人协同创作的超文本系统，支持那些面向社群的协作式写作，为协作式写作提供了必要的帮助

##### 4. Gists

上面讲述的都是基于仓库的，这里说的 Gist 是 GitHub 的一个子服务，用于小型代码片段的分享，比如说你有一些不错的代码片段，但是又没必要创建一个仓库来存放，就可以用 Gist 来存放

##### 5. Contributions

在你的公有仓库中进行代码提交，就会在个人信息页面的 Contribution 栏中出现标记，拿我自己的来做个示范：

![](https://upload-images.jianshu.io/upload_images/3426615-9754f66f95061748.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

两三个 Java Web 项目、掘金翻译计划、个人博客和 LeetCode 记录基本构成了我的 GitHub 的全部贡献，几乎在某一段时间内都有每天做一些

#### 中国人在 GitHub

在最后说明一下，听说现在国内的培训机构有包装 GitHub 的行为，中国人还真是什么都敢干，先是大闹 Issues，然后是抄袭行为和可以包装行为。在这里强烈鄙视那些在 GitHub 上抄袭的人。就个人而言，它只是一个有着开发功能的社区，即使包装的再好，连基本的协同开发流程都不知道是怎样的，不就是衣着光鲜亮丽的原始人吗？GitHub 带给我们这些普通开发者的最明显的好处就是十分便利的分布式协作开发流程，以及发现一些有趣项目的机会。目光短浅的人往往会走偏，去刻意包装一下，是太没自信了吗？

