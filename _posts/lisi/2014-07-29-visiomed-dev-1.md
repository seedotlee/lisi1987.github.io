---
    layout: post
    author: 李思
    title: 一个法国iPhone客户端的开发（前言）
    category : visiomed系列
    tagline: ""
    tags : [项目开发, 架构设计]
---

{% include JB/setup %}

<!--  发布地址 https://stackedit.io/viewer#!url=  -->

##前言
这将是一个系列的博文，这个系列将介绍一个iPhone软件项目中遇到的各种坑以及我们的解决方案，其中包括：`架构设计`、`UI组件设计`、`storyboard使用`、`autolayout使用`、`coredata使用`、`蓝牙解决方案` 等。

##项目背景
这是一个以`客户设计`为导向的软件。客户设计？这是我自己杜撰出来的一个词，如有雷同纯属巧合。

>与以往的客户需求导向不同，已经不是单一的从客户那进行需求调研然后我们给出解决方案，做出原型然后确认需求，最后开发。
>客户不仅仅有自己的需求，而且已经把UI、互交、软件层级都已经给出。

看到这里可能大家一定想的是：都设计完了那还不好？直接开发就OK了啊，真是帮你们省事啊~
在之前我也是这么想的，可是拿到设计稿之后心中顿时出现几个字：`尼玛！`。

简单介绍一下项目背景。这是一个配合硬件来使用的软件，其中目前设想和规划支持的硬件大致有 16 种。可以分为3大类分别是：`健康、安全、家庭`。暂时支持的硬件有：Thermometer（体温计）、Activity Tracker（运动手环）、Pet Tracker（宠物追踪器）。

##客户的想法
客户想做这样的一个软件：
> 该软件在最外层有一个类似于Android众的Launcher的功能，能够在这个Launcher中统一管理各个子软件以及作为各子软件的入口。

大家要注意的是这个软件到最后需要支持的子软件有16种之多。我们曾经建议他们将这些子软件都独立成一个App，仅仅将Launcher作为入口和身份验证，不要将子软件完全在Lancher中实现，否则这个Launcher将是一个非常庞大的App，不仅生成的安装文件会非常的大，而且越到后面维护成本也会成指数增加。

但是~甲方如果能够听乙方的意见的话，那这个世界上就没有难做的项目。最后也只有我们来妥协。

所以我们需要比较好一点的底层架构来支持我们这个软件，以至于到后期`扩展应用与维护`的时候不会那么困难，对于巨大的安装文件我们是没有办法了。

##UI、互交与层级
部分UI如下图，关于美观的问题这里就不做评价，各人审美不同（据客户说是一个20多人的设计团队设计出来的#-_-）。

----
![img](http://lisi1987-blog-images.qiniudn.com/visiomed_952DFAD5-8971-4D36-8D72-DB1F0C601035.png?imageView/2/w/213/q/85|watermark/2/text/bGlzaTE5ODc=/font/5b6u6L2v6ZuF6buR/fontsize/200/fill/I0VGRUZFRg==/dissolve/50/gravity/SouthEast/dx/10/dy/10) ![img](http://lisi1987-blog-images.qiniudn.com/visiomed_38606599-DECF-452C-A860-7FAAC7B0A8CD.png?imageView/2/w/213/q/85|watermark/2/text/bGlzaTE5ODc=/font/5b6u6L2v6ZuF6buR/fontsize/200/fill/I0VGRUZFRg==/dissolve/50/gravity/SouthEast/dx/10/dy/10) ![img](http://lisi1987-blog-images.qiniudn.com/visiomed_25D9E3BC-E1BC-4877-BDD1-11262F246F00.png?imageView/2/w/213/q/85%7Cwatermark/2/text/bGlzaTE5ODc=/font/5b6u6L2v6ZuF6buR/fontsize/200/fill/I0VGRUZFRg==/dissolve/50/gravity/SouthEast/dx/10/dy/10)

![img](http://lisi1987-blog-images.qiniudn.com/visiomed_94FF24D6-5319-436E-AC80-AA612B1064F0.png?imageView/2/w/213/q/85%7Cwatermark/2/text/bGlzaTE5ODc=/font/5b6u6L2v6ZuF6buR/fontsize/200/fill/I0VGRUZFRg==/dissolve/50/gravity/SouthEast/dx/10/dy/10) ![img](http://lisi1987-blog-images.qiniudn.com/visiomed_FB66729F-92B8-4E7A-AC71-D6C255521BDD.png?imageView/2/w/213/q/85|watermark/2/text/bGlzaTE5ODc=/font/5b6u6L2v6ZuF6buR/fontsize/200/fill/I0VGRUZFRg==/dissolve/50/gravity/SouthEast/dx/10/dy/10) ![img](http://lisi1987-blog-images.qiniudn.com/visiomed_92D8A8B4-6749-41D7-ADAE-507181081702.png?imageView/2/w/213/q/85%7Cwatermark/2/text/bGlzaTE5ODc=/font/5b6u6L2v6ZuF6buR/fontsize/200/fill/I0VGRUZFRg==/dissolve/50/gravity/SouthEast/dx/10/dy/10)
----

由截图可以看出整个App的一个UI架构可以分为三块`SideBar`，`Tabbar`，`MainView`。
这种UI架构是没有什么问题的很主流，**但是**各位看官请注意侧边栏内容，再看看底部导航栏，是不是有几项是相似的？没错了这几个相似的功能就是一样的功能点击后的效果是一致的！
客户对侧边栏的定义是`快捷方式`。![img](http://pic.sc.chinaz.com/Files/pic/faces/2659/211.jpg)

做过App的尤其是iPhone App的应该都知道，在开发之前都会设计一条优质的路径去引导用户如何来使用（也就是UINavigation）。这次完全打破了传统，要用Native App开发出Web App的特性，任意跳转。
这只是整个项目中的一个坑，在后续的介绍中我们会一一指出，并提供我们的解决方案。

好废话说了一大堆，现在是时候来干货了，现在进入我们的第一篇：

**[一个法国iPhone客户端开发之软件底层架构设计]()**


