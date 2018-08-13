---



layout:		post



title:		Git 系列文章（四）



subtitle:	GitHub Flow



date:		2018-08-13



author:		L



header-img:	img/post/2018-08/13.jpg



tags:



    - Git



---

[](https://upload-images.jianshu.io/upload_images/3426615-19a920f541db41bc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 正文之前

> 因为我现在还是学生，所以参加过的 GitHub 团队合作项目就只有[掘金翻译计划](https://github.com/xitu/gold-miner)，所以这一篇文章所讲述的内容是从我目前的经历和《GitHub入门与实践》([日]大塚弘记著) 中总结而来
>
> GitHub Flow，是一种开发流程，是一种概念，而不是什么框架

### 正文

#### 分支

之前说过，如果要给别人的仓库做贡献，就要先将这个仓库 fork 到自己的账号里，然后再进行改动，再 PR

![](https://upload-images.jianshu.io/upload_images/3426615-1c96ffebc795f426.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


但是，在 GitHub Flow 中，变得更加简单，不 fork，直接在源仓库创建分支：

![](https://upload-images.jianshu.io/upload_images/3426615-0172a93ccb7c17f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有几个要求：

> 1. master 分支是稳定的一个分支（也就是对外部署的分支）
> 2. 创建的其他分支的命名要具有描述性（别起一些乱七八糟的名字，害人害己）
> 3. 你所创建的分支，要在**本地仓库**和**远程仓库**同时存在（为了备份，同时也为了能让别人看到你的进展）
> 4. 提交的粒度要小，最好每次提交只针对某个功能，而不是一次提交中新增好几个功能
> 5. 定期 Push，不仅是备份代码，也能让别人看到你的进展

这里有一个关于 Git 开发流程的 [Gist](https://gist.github.com/yesmeck/4245406)，里面定义了各种功能的分支的命名方式

其实我也没怎么经历过 GitHub Flow，还是去找一些项目练手吧

<p><img src="https://upload-images.jianshu.io/upload_images/3426615-79919768b5365766.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" alt="" width="40%" height="40%" /></p>
