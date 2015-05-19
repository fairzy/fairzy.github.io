---
layout: post
title:  "Part I: The Layer Beneath - Chapter 1: The Layer Tree"
date:   2015-05-19 11:51:07
categories: Core Animation
---
> Ogres have layers. Onions have layers. You get it? We both have layers.
> 
> --Sherk

***

	Core Animation is misleadingly named. You might assume that its primary purpose is animation, but actually animation is only one facet of the framework, which originally bore the less animation-centric name of Layer Kit.

Core Animation的名字有点误导性，看到这个名字你可能会认为它的主要功能是animaton动画，但其实动画只是这个framework的一方面，它原来的名字其实是Layer Kit，看起来就没那么让你觉得它是以动画为中心。

	Core Animation is a compositing engine; its job is to compose different pieces of visual content on the screen, and to do so as fast as possible. The content in question is divided into individual layers stored in a hierarchy known as the layer tree. This tree forms the underpinning for all of UIKit, and for everything that you see on the screen in an iOS application.

Core Animation是一个`合成(compositing)引擎`，它的主要任务就是尽可能快的把要显示的不同的内容合成并显示在屏幕上，要显示的内容被分成不同的layer并按照层级存储，这些层级存储的layer就被成为layer树(layer tree)。这些tree组成了UIKit的基础，也是你在iOS应用中看到屏幕上的所有东西的基础。

	Before we discuss animation at all, we’re going to cover Core Animation’s static compositing and layout features, starting with the layer tree.

在我们讨论任何动画之前，我们要先把Core Aniamtion的静态合成、布局特点讲清楚，让我们先从layer tree开始。

###Layers and Views
	If you’ve ever created an app for iOS or Mac OS, you’ll be familiar with the concept of a view. A view is a rectangular object that displays content (such as images, text, or video), and intercepts user input such as mouse clicks or touch gestures. Views can be nested inside one another to form a hierarchy, where each view manages the position of its children (subviews). Figure 1.1 shows a diagram of a typical view hierarchy.

如果你以前创建过iOS或者Mac OS的app，那么你对View的概念就会很熟悉，一个View是一个显示内容(比如图片，文字，或者视频)的矩形对象，除此之外View还负责获取用户事件，比如鼠标点击或者触摸事件。View可以互相嵌套组成层级，其中的每个View都负责管理他的子View的位置，图1.1表示了一个典型的View层级。
![左边：一个典型的iOS屏幕，右边：组成屏幕内容的View层级](http://ww3.sinaimg.cn/large/6cdd1b93gw1es9gugwrq3j20hn0c3jsa.jpg "左边：一个典型的iOS屏幕，右边：组成屏幕内容的View层级")

图1.1 左边：一个典型的iOS屏幕，右边：组成屏幕内容的View层级


	In iOS, views all inherit from a common base class, UIView. UIView handles touch events and supports Core Graphics-based drawing, affine transforms (such as rotation or scaling), and simple animations such as sliding and fading.

在iOS中，所有的View都继承自一个公共类，`UIView`。UIView处理触摸事件和基于Core Graphics的绘制，仿射变换(affine transforms)(比如旋转和缩放)，以及处理简单的动画，比如移动或渐变。
	What you may not realize is that UIView does not deal with most of these tasks itself. Rendering, layout, and animation are all managed by a Core Animation class called CALayer.
你不知道的是，大部分这些任务其实并不是UIView来处理的。渲染，布局以及动画全部都是由Core Animation的类来处理的，这个类就是CALayer。

###`CALayer`

	The CALayer class is conceptually very similar to UIView. Layers, like views, are rectangular objects that can be arranged into a hierarchical tree. Like views, they can contain content (such as an image, text, or a background color) and manage the position of their children (sublayers). They have methods and properties for performing animations and transforms. The only major feature of UIView that isn’t handled by CALayer is user interaction.

`CALayer`这个类在概念上与`UIView`非常类似，Layers也是可以被组成层级树的矩形对象，像Views一样，可以包含不同的内容（比如图片，文字，或者背景颜色）并且设置他们子Layer的坐标，他们有一些property可以用来执行动画和3d变换。CALayer与UIView最大的不同在于，CALayer不处理用户事件。
	CALayer is not aware of the responder chain (the mechanism that iOS uses to propagate touch events through the view hierarchy) and so cannot respond to events, although it does provide methods to help determine whether a particular touch point is within the bounds of a layer (more on this in Chapter 3, “Layer Geometry”).
CALayer不参与响应链(响应链是iOS里面事件在view层级中传递的机制)，而且不响应事件，尽管它提供了一些方法来帮助判断一个特定的触摸点是否在laeyer的边框之内（在第三章“Layer中的几何概念”中有介绍）。###Parallel Hierarchies（平行的层级结构）
	Every UIView has a layer property that is an instance of CALayer. This is known as the backing layer. The view is responsible for creating and managing this layer and for ensuring that when subviews are added or removed from the view hierarchy that their corresponding backing layers are connected up in parallel within the layer tree (see Figure 1.2).
每一个UIView都有一个layer的property，这个property的类型是CALayer，这个layer被称为底层layer（`backing layer`），view负责创建和管理这个layer，并且确保view在添加和移除到view层级树中时，view对应的底层layer也相应的添加或移除到layer的层级树。

![图 1.2 layertree的层级结构（左边）与view的层级结构（右边）一一对应的](http://ww1.sinaimg.cn/large/6cdd1b93gw1es9h6jfsbsj20g20b974x.jpg)

图 1.2 layertree的层级结构（左边）与view的层级结构（右边）一一对应的


	It is actually these backing layers that are responsible for the display and animation of everything you see onscreen. UIView is simply a wrapper, providing iOS-specific functionality such as touch handling and high-level interfaces for some of Core Animation’s low-level functionality.

你在屏幕上看到的所有内容和动画，其实都是这些backing layer在负责。UIView只是一个简单的封装，提供了针对iOS的触摸事件支持，以及对底层动画的高级封装。
	Why does iOS have these two parallel hierarchies based on UIView and CALayer? Why not a single hierarchy that handles everything? The reason is to separate responsibilities, and so avoid duplicating code. Events and user interaction work quite differently on iOS than they do on Mac OS; a user interface based on multiple concurrent finger touches (multitouch) is a fundamentally different paradigm to a mouse and keyboard, which is why iOS has UIKit and UIView and Mac OS has AppKit and NSView. They are functionally similar, but differ significantly in the implementation.Drawing, layout, and animation, in contrast, are concepts that apply just as much to touchscreen devices like the iPhone and iPad as they do to their laptop and desktop cousins. By separating out the logic for this functionality into the standalone Core Animation framework, Apple is able to share that code between iOS and Mac OS, making things simpler both for Apple’s own OS development teams and for third-party developers who make apps that target both platforms.
那么，为什么iOS会有两个基于UIView和CALayer的平行的层级关系呢？为什么不用一个层级关系处理所有的事情？原因就在于把责任分开，避免重复的代码，iOS和Mac OS在事件处理上差别非常大，多点触摸和鼠标键盘是有非常大的区别的，于是，iOS上有UIKit和UIView，而在Mac OS上，是AppKit和NSView，他们功能类似，但是有不同的实现。绘制，布局，动画，概念上在触摸屏和电脑上都是类似的。通过把功能在逻辑分开为单独的Core Aniamtion框架，苹果就可以在iOS和Mac OS之间共享的这部分代码，对于苹果自己的系统开发团队和为两个平台开发应用的第三方的开发者来说都非常有利。
	In fact, there are not two, but four such hierarchies, each performing a different role. In addition to the view hierarchy and layer tree, there are the presentation tree and render tree, which we discuss in Chapter 7, “Implicit Animations,” and Chapter 12, “Tuning for Speed,” respectively.
实际上，不止有两个层级，而是由四个平行的层级，每一种都有不同的任务。除了View树和layer树，还有presentation tree和render tree，我们将会在第七章，“隐式动画”，和第12章，“加速动画”中分别讨论这两个树。###Layer Capabilities（Layer的能力）
	So if CALayer is just an implementation detail of the inner workings of UIView, why do we need to know about it at all? Surely Apple provides the nice, simple UIView interface precisely so that we don’t need to deal directly with gnarly details of Core Animation itself?
so你可能会问了，如果CALayer只是UIView的内部的详细实现，为什么我们还要知道这玩意儿的全部内容呢？牛逼的苹果不是已经给我们封装了了简洁漂亮的，我们可以直接调用的接口了么？
	This is true to some extent. For simple purposes, we don’t really need to deal directly with CALayer, because Apple has made it easy to leverage powerful features like animation indirectly via the UIView interface using simple high-level APIs.

一定程度上，你说的的确没错。在简单的情况下，我们确实不需要直接跟CALayer打交道，因为苹果已经把强大的动画直接封装为简单的UIView接口提供给我们了。	But with that simplicity comes a loss of flexibility. If you want to do something slightly out of the ordinary, or make use of a feature that Apple has not chosen to expose in the UIView class interface, you have no choice but to venture down into Core Animation to explore the lower-level options.

但是，这样也少了很多灵活性，如果你想做点非常规的的事，或者评估没有暴露接口的事儿，你就没有偶选择了，只能去深入研究一下Core Animation的底层选项。	We’ve established that layers cannot handle touch events like UIView can, so what can they do that views can’t? Here are some features of CALayer that are not exposed by UIView:	* Drop shadows, rounded corners, and colored borders 
	* 3D transforms and positioning	* Nonrectangular bounds	* Alpha masking of content	* Multistep, nonlinear animations
	We explore these features in the following chapters, but for now let’s look at how CALayer can be utilized within an app.
我们已经知道layer不处理触摸事件，那么，layer可以干什么view不可以干的事儿呢？下面这些特点是CALayer没有暴露给UIView的：
* 阴影，圆角，有颜色边框
* 3D变换
* 非矩形的区域
* 内容的Alpah通道蒙版
* 多步的，非线性的动画

我们将会在接下来的章节中讨论这些特性，现在，让我们看一下CALyer在app中可以怎样应用。

###Working with Layers(使用layers)
	Let’s start by creating a simple project that will allow us to manipulate the properties of a layer. In Xcode, create a new iOS project using the Single View Application template.

让我们创建一个简单的项目，并且操作一下layer的属性，在xcode里面建一个Single View的模板建一个iOS的项目。	Create a small view (around 200×200 points) in the middle of the screen. You can do this either programmatically or using Interface Builder (whichever you are more comfortable with). Just make sure that you include a property in your view controller so that you can access the small view directly. We’ll call it layerView.
在屏幕中将创建一个小的view（大约200x200像素）。你可以用代码或者IB。记得在view Controller里面增加一个property来随时访问这个view。我们就叫这个property为layerView。If you run the project, you should see a white square in the middle of a light gray background (see Figure 1.3). If you don’t see that, you may need to tweak the background colors of the window/view.
如果你运行这个项目，你就会看到一个白色的正方形在灰色的背景之中（图1.3），如果你没看到这个白色矩形，你需要调整window或者view的背景颜色。
![Figure1.3 A white UIView on a gray background](http://ww4.sinaimg.cn/large/6cdd1b93gw1es9h9p0yt6j20hb08ddg8.jpg)

Figure1.3 A white UIView on a gray background


	That’s not very exciting, so let’s add a splash of color. We’ll place a small blue square inside the white one.

这个不怎么激动人心啊，下面让我们来加点颜色吧，让我们在白色矩形中放一个蓝色的矩形吧。
	We could achieve this effect by simply using another UIView and adding it as a subview to the one we’ve already created (either in code or with Interface Builder), but that wouldn’t really teach us anything about layers.
我们可以简单的用一个UIView的subview添加到view中来实现，但这不是我们想讨论的。
	Instead, let’s create a CALayer and add it as a sublayer to the backing layer of our view. Although the layer property is exposed in the UIView class interface, the standard Xcode project templates for iOS apps do not include the Core Animation headers, so we cannot call any methods or access any properties of the layer until we add the appropriate framework to the project. To do that, first add the QuartzCore framework in the applicationtarget’s Build Phases tab (see Figure 1.4), and then import <QuartzCore/QuartzCore.h> in the view controller’s .m file.
让我们换一种方法，我们创建一个CALyer然后把它作为子layer添加到view的backing layer中。尽管view的layer属性石暴露在UIView类的中，标准的XCode模板项目并没有包括Core Animation框架，所以我们不能直接调用layer的任何方法或者访问任何属性，知道我们把Core animation的framework添加到项目中。要添加框架，在项目的Build Phase中加入QuartzCore.framework，然后在view Controller的.m文件中import <QuartzCore/QuartzCore.h> 

![Figure 1.4 Adding the QuartzCore framework to the project](http://ww2.sinaimg.cn/large/6cdd1b93gw1es9hbl2mmqj20g708sq40.jpg)

Figure 1.4 Adding the QuartzCore framework to the project


	After doing that, we can directly reference CALayer and its properties and methods in our code. In Listing 1.1, we create a new CALayer programmatically, set its backgroundColor property, and then add it as a sublayer to the layerView backing layer. (The code assumes that we created the view using Interface Builder and that we have already linked up the layerView outlet.) Figure 1.5 shows the result.

完成这些之后，我们就可以直接访问CALayer的属性和方法了。在列表1.1中，我们用代码创建了一个新的CALayer，设置了它的背景颜色，并把它作为一个子layer添加到layerView的backing layer中（这里的代码假设你用的是IB并且已经吧IB里面的view连接到代码的IBOutlet了）。图1.5显示了运行结果。
	The CALayer backgroundColor property is of type CGColorRef, not UIColor like the UIView class’s backgroundColor, so we need to use the CGColor property of our UIColor object when setting the color. You can create a CGColor directly using Core Graphics methods if you prefer, but using UIColor saves you from having to manually release the color when you no longer need it.

CALyer的backgroundColor属性是CGColorRef类型，并不像UIView一样是UIColor，所以我们需要UIColor的CGColor属性。你也可以用Core Graphics的方法直接创建一个CGColor，但是用UIColor，你可以不用担心color的内存释放问题。
Listing 1.1 Adding a Blue Sublayer to the View
	#import "ViewController.h"	#import <QuartzCore/QuartzCore.h>	@interface ViewController ()	@property (nonatomic, weak) IBOutlet UIView *layerView;	@end	@implementation ViewController	- (void)viewDidLoad {		[super viewDidLoad];		//创建子layer		CALayer *blueLayer = [CALayer layer];		blueLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
		blueLayer.backgroundColor = [UIColor blueColor].CGColor;		//添加到view中		[self.layerView.layer addSublayer:blueLayer];
	}	@end![Figure1.5 AsmallblueCALayernestedinsideawhiteUIView](http://ww3.sinaimg.cn/large/6cdd1b93gw1es9hg4gtb4j20hn099wex.jpg)

Figure1.5 A small blue CALayer nested inside a white UIView


	A view has only one backing layer (created automatically) but can host an unlimited number of additional layers. As Listing 1.1 shows, you can explicitly create standalone layers and add them directly as sublayers of the backing layer of a view. Although it ispossible to add layers in this way, more often than not you will simply work with views and their backing layers and won’t need to manually create additional hosted layers.

一个View只能有一个backing layer（自动创建的）但是可以有无限个子layer，就像在代码1.1中，我们显示的创建了一个单独的layer并把它直接添加到view的backing layer中，但通常，我们只是与view和他们的backing layer打交道，并不需要手动创建一些layer
	On Mac OS, prior to version 10.8, a significant performance penalty was involved in using hierarchies of layer-backed views instead of standalone CALayer trees hosted inside a single view. But the lightweight UIView class in iOS barely has any negative impact on performance when working with layers. (In Mac OS 10.8, the performance of NSView is greatly improved, as well.)

在10.8之前的Mac OS中，用多个有backing layer的view去构成一个view层级结构比用一个view，然后view中有是一个layer tree的方式有严重的性能问题。但是在iOS中，UIView是轻量级的，用layers工作并不会对性能造成负面影响（在10.8的Mac OS版本中，NSView的这个性能问题已经极大的被解决了）。	The benefit of using a layer-backed view instead of a hosted CALayer is that while you still get access to all the low-level CALayer features, you don’t lose out on the high-level APIs (such as autoresizing, autolayout, and event handling) provided by the UIView class.

用view的backing layer而不是单独建一个根layer的好处是，你可以在获得CALayer的底层功能时，也可以获取UIView的高级封装api（比如autoresizing，autolayout以及事件处理）。		You might still want to use a hosted CALayer instead of a layer-backed UIView in a real- world application for a few reasons, however:
	* You might be writing cross-platform code that will also need to work on a Mac.	* YoumightbeworkingwithmultipleCALayersubclasses(seeChapter6, “Specialized Layers”) and have no desire to create new UIView subclasses to host them all.	* You might be doing such performance-critical work that even the negligible overhead of maintaining the extra UIView object makes a measurable difference (although in that case, you’ll probably want to use something like OpenGL for your drawing anyway).
当然，在实际的工作过程中，你可能还是会需要单独创建CALayer而不是用view的backing layer，比如以下情况：
* 你可能想写一个跨平台的在mac os也能运行的代码。
* 你可能需要用到CALyer的多个子类（参考第6章，“定制的Layer”）并且根本不需要创建UIView的子类来托管他们。
* 你可能再做一些对性能要求非常高的的工作，即使维护UIView的对象这样微不足道的工作都会产生可以测量的性能影响（尽管在那种情况下，你可能需要用OpenGL来绘制）。
	But these cases are rare, and in general, layer-backed views are a lot easier to work with than hosted layers.
但是呢，这些情况大体来说比较少见，基于layer的view对于单独的layer来说，减少了很多的工作量。#Summary（小结）
	This chapter explored the layer tree, a hierarchy of CALayer objects that exists in parallel beneath the UIView hierarchy that forms the iOS interface. We also created our own CALayer and added it to the layer tree as an experiment.
这一章我们探索了layer树，一个与组成iOS界面的UIView层级结构平行的CALyer对象的层级结构，我们也创建了我们自己的layer并且把它添加到到layer树中了。
	In Chapter 2, “The Backing Image,” we look at the CALayer backing image and at the properties that Core Animation provides for manipulating how it is displayed.

在第2章，“底层图像”中，我们将会看一下CALayer的底层图像（backing image）和CA提供的操控它如何显示的一些属性。