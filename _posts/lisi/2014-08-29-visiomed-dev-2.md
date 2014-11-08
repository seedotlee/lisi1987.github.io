---
    layout: post
    author: 李思
    title: 一个法国iPhone客户端开发之软件底层架构设计
    category : visiomed系列
    tagline: ""
    tags : [项目开发, 架构设计]
---
{% include JB/setup %}

<!-- create time: 2014-08-27 22:43:30  -->

[在上一篇](http://172.18.80.61:4000/lessons/2014/07/29/visiomed-dev-1/)中整体的介绍了一下这个客户端的`需求`以及`要求`。在这一篇中将要介绍一下我们针对这样一个需求做出的软件架构上的设计。

##UML
![UML](http://lisi1987-blog-images.qiniudn.com/visiomed_visiomed_uml_002.png)

上图为总体结构的简版UML图，从图中我们就可以比较清楚的了解到整个代码结构。

图中有将属性和方法列出的类图是框架中起到关键作用的几个类：

## CoordinatingViewController
在上一篇我们讲到在这个应用中包括着N个子应用，这些子应用基本上是相对独立，但使用了同样的账号和用户，所以在这个应用里我们需要一个统筹者，来进行应用切换、数据共享。

在这里我们就需要使用到`中介者模式`：

	中介者模式：用一个对象来封装一系列对象的互交方式。中介者使各对象不需要显示的引用，从而使其耦合松散，而且可以独立地改变它们之间的互交。

CoordinatingViewController在应用中就是起到了这个作用，所有相关的ViewController都由它控制。CoordinatingViewController我们对于它是用一个单例来实现。这样我们就能做到数据共享的功能。

```objc
[CoordinatingViewController sharedInstance].currentController;
```

在UML图中看到的`showContentWithController:animation:`就是进行ViewController切换的方法。

```objc
- (void)showContentWithController:(UIViewController *)controller animation:(SwitchAppAnimation)animation
{
    UIViewController *toController;
    UIViewController *fromController;
    toController = controller;
    fromController = self.currentController;
    if (self.currentController == toController) {
        return;
    }
    self.currentController = toController;
    [self addChildViewController:toController];
    [_containerView addSubview:toController.view];
    [toController didMoveToParentViewController:self];
    [fromController willMoveToParentViewController:nil];
    [fromController.view endEditing:YES];
    if (animation) {
        animation(fromController.view,toController.view);
    }
    else {
        [fromController.view removeFromSuperview];
    }
    [fromController removeFromParentViewController];
}
```

SwitchAppAnimation为一个场景切换动画blok，在底层架构中已经封装了部分标准场景切换，如果某些场景需要自己定制动画也可自己实现然后传入。

CoordinatingViewController除了本职工作场景切换外，还有一些派生出来工作如：处理token过期、应用内语言切换后的应用重启、转屏的事件传递等。这些细节的功能实现我们留到后面慢慢道来。今天我们的主题是架构。

###BaseAppViewController
BaseAppViewController就是应用中各子应用的基类了，在上篇中我们知道虽然各子应用之间并没有存在太多的联系的，但是从UI上可以看到他们整体结构基本一致，都是包括了`SideBar、TabBar、Content`。所以各应用是有必要拥有一套基本UI框架。

由UML图中可以看出BaseAppViewController有三个属性：baseSidebarController、baseTabbarController、mainController，这些就构成了子应用的基本UI结构。然而每个子应用也仅仅只是在大结构上一致，还有许许多多的细节是完全不相同，比如：SideBar的内容、Tabbar的按钮多少与类型（图片或文字）、主题颜色等等。在这里我们需要用到的就是*抽象工厂模式*。

	抽象工厂模式：提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们的具体的类。

在BaseAppViewController中只需要定义方法 `createTabBar、createSideBar、loadMainViewController` 就能完成产品的组装。

```objc
-(void)loadMainViewController
{
    UIViewController *vc = [self initializeViewController];
    self.mainController = vc;
    NSString *className = NSStringFromClass(vc.class);
    _controllersDict[className] = vc;
    self.mainController.view.frame = self.mainView.bounds;
}
-(void)createSideBar
{
    self.baseSidebarController = [self initializeSideBarViewController];
    self.baseSidebarController.themeColor = self.themeColor;
    [self.baseSidebarController.view setFrame:CGRectMake(0, 0, CGRectGetWidth(_sidebarView.frame), CGRectGetHeight(_sidebarView.frame))];
    [self.baseSidebarController layoutViews];
    [self closeSideBar];
}
-(void)createTabBar
{
    self.baseTabbarController = [self initializeTabBarViewController];
    self.baseTabbarController.themeColor = self.themeColor;
    [self.baseTabbarController.view setFrame:_tabbarView.bounds];
}
```

做完这些组装之后我们只需要到各子应用的实现类中去实现各自的初始化方法就行了。

```objc
-(BaseTabBarViewController *)initializeTabBarViewController
{
    NSAssert(NO, @"子类必须重写initializeTabBarViewController！");
    return nil;
}

-(BaseSideBarViewController *)initializeSideBarViewController
{
    NSAssert(NO, @"子类必须重写initializeSideBarViewController！");
    return nil;
}

-(UIViewController *)initializeViewController
{
    NSAssert(NO, @"子类必须重写initializeViewController！");
    return nil;
}
```

##小结

软件需求变幻无穷，计划没有变化快，但是我们还是要寻找出不变的东西，并将它和变化的东西分离开来。所以软件的架构是十分重要的，我们需要通过不断实践去理解各种设计模式的精髓，正确的运用它，才能写出优雅的代码。


未完待续...



