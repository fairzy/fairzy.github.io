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
CALayer is not aware of the responder chain (the mechanism that iOS uses to propagate touch events through the view hierarchy) and so cannot respond to events, although it does provide methods to help determine whether a particular touch point is within the bounds of a layer (more on this in Chapter 3, “Layer Geometry”).###Parallel Hierarchies
Every UIView has a layer property that is an instance of CALayer. This is known as the backing layer. The view is responsible for creating and managing this layer and for ensuring that when subviews are added or removed from the view hierarchy that their corresponding backing layers are connected up in parallel within the layer tree (see Figure 1.2).
![Figure 1.2 The structure of the layer tree (left) mirrors that of the view hierarchy (right)](http://ww1.sinaimg.cn/large/6cdd1b93gw1es9h6jfsbsj20g20b974x.jpg)
Figure 1.2 The structure of the layer tree (left) mirrors that of the view hierarchy (right)

It is actually these backing layers that are responsible for the display and animation of everything you see onscreen. UIView is simply a wrapper, providing iOS-specific functionality such as touch handling and high-level interfaces for some of Core Animation’s low-level functionality.
Why does iOS have these two parallel hierarchies based on UIView and CALayer? Why not a single hierarchy that handles everything? The reason is to separate responsibilities, and so avoid duplicating code. Events and user interaction work quite differently on iOS than they do on Mac OS; a user interface based on multiple concurrent finger touches (multitouch) is a fundamentally different paradigm to a mouse and keyboard, which is why iOS has UIKit and UIView and Mac OS has AppKit and NSView. They are functionally similar, but differ significantly in the implementation.Drawing, layout, and animation, in contrast, are concepts that apply just as much to touchscreen devices like the iPhone and iPad as they do to their laptop and desktop cousins. By separating out the logic for this functionality into the standalone Core Animation framework, Apple is able to share that code between iOS and Mac OS, making things simpler both for Apple’s own OS development teams and for third-party developers who make apps that target both platforms.
In fact, there are not two, but four such hierarchies, each performing a different role. In addition to the view hierarchy and layer tree, there are the presentation tree and render tree, which we discuss in Chapter 7, “Implicit Animations,” and Chapter 12, “Tuning for Speed,” respectively.###Layer Capabilities
So if CALayer is just an implementation detail of the inner workings of UIView, why do we need to know about it at all? Surely Apple provides the nice, simple UIView interface precisely so that we don’t need to deal directly with gnarly details of Core Animation itself?
This is true to some extent. For simple purposes, we don’t really need to deal directly with CALayer, because Apple has made it easy to leverage powerful features like animation indirectly via the UIView interface using simple high-level APIs.
But with that simplicity comes a loss of flexibility. If you want to do something slightly out of the ordinary, or make use of a feature that Apple has not chosen to expose in the UIView class interface, you have no choice but to venture down into Core Animation to explore the lower-level options.
We’ve established that layers cannot handle touch events like UIView can, so what can they do that views can’t? Here are some features of CALayer that are not exposed by UIView:* Drop shadows, rounded corners, and colored borders ▪ 3D transforms and positioning* Nonrectangular bounds* Alpha masking of content* Multistep, nonlinear animations
We explore these features in the following chapters, but for now let’s look at how CALayer can be utilized within an app.
###Working with Layers
Let’s start by creating a simple project that will allow us to manipulate the properties of a layer. In Xcode, create a new iOS project using the Single View Application template.
Create a small view (around 200×200 points) in the middle of the screen. You can do this either programmatically or using Interface Builder (whichever you are more comfortable with). Just make sure that you include a property in your view controller so that you can access the small view directly. We’ll call it layerView.
If you run the project, you should see a white square in the middle of a light gray background (see Figure 1.3). If you don’t see that, you may need to tweak the background colors of the window/view.
![Figure1.3 A white UIView on a gray background](http://ww4.sinaimg.cn/large/6cdd1b93gw1es9h9p0yt6j20hb08ddg8.jpg)
Figure1.3 A white UIView on a gray background

That’s not very exciting, so let’s add a splash of color. We’ll place a small blue square inside the white one.
We could achieve this effect by simply using another UIView and adding it as a subview to the one we’ve already created (either in code or with Interface Builder), but that wouldn’t really teach us anything about layers.
Instead, let’s create a CALayer and add it as a sublayer to the backing layer of our view. Although the layer property is exposed in the UIView class interface, the standard Xcode project templates for iOS apps do not include the Core Animation headers, so we cannot call any methods or access any properties of the layer until we add the appropriate framework to the project. To do that, first add the QuartzCore framework in the applicationtarget’s Build Phases tab (see Figure 1.4), and then import <QuartzCore/QuartzCore.h> in the view controller’s .m file.

![Figure 1.4 Adding the QuartzCore framework to the project](http://ww2.sinaimg.cn/large/6cdd1b93gw1es9hbl2mmqj20g708sq40.jpg)
Figure 1.4 Adding the QuartzCore framework to the project

After doing that, we can directly reference CALayer and its properties and methods in our code. In Listing 1.1, we create a new CALayer programmatically, set its backgroundColor property, and then add it as a sublayer to the layerView backing layer. (The code assumes that we created the view using Interface Builder and that we have already linked up the layerView outlet.) Figure 1.5 shows the result.
The CALayer backgroundColor property is of type CGColorRef, not UIColor like the UIView class’s backgroundColor, so we need to use the CGColor property of our UIColor object when setting the color. You can create a CGColor directly using Core Graphics methods if you prefer, but using UIColor saves you from having to manually release the color when you no longer need it.

Listing 1.1 Adding a Blue Sublayer to the View
	\#import "ViewController.h"	\#import <QuartzCore/QuartzCore.h>	@interface ViewController ()	@property (nonatomic, weak) IBOutlet UIView *layerView;	@end	@implementation ViewController	- (void)viewDidLoad {		[super viewDidLoad];		//create sublayer		CALayer *blueLayer = [CALayer layer];		blueLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
		blueLayer.backgroundColor = [UIColor blueColor].CGColor;		//add it to our view		[self.layerView.layer addSublayer:blueLayer]; }	@end![Figure1.5 AsmallblueCALayernestedinsideawhiteUIView](http://ww3.sinaimg.cn/large/6cdd1b93gw1es9hg4gtb4j20hn099wex.jpg)
Figure1.5 AsmallblueCALayernestedinsideawhiteUIView

A view has only one backing layer (created automatically) but can host an unlimited number of additional layers. As Listing 1.1 shows, you can explicitly create standalone layers and add them directly as sublayers of the backing layer of a view. Although it ispossible to add layers in this way, more often than not you will simply work with views and their backing layers and won’t need to manually create additional hosted layers.
On Mac OS, prior to version 10.8, a significant performance penalty was involved in using hierarchies of layer-backed views instead of standalone CALayer trees hosted inside a single view. But the lightweight UIView class in iOS barely has any negative impact on performance when working with layers. (In Mac OS 10.8, the performance of NSView is greatly improved, as well.)
The benefit of using a layer-backed view instead of a hosted CALayer is that while you still get access to all the low-level CALayer features, you don’t lose out on the high-level APIs (such as autoresizing, autolayout, and event handling) provided by the UIView class.
You might still want to use a hosted CALayer instead of a layer-backed UIView in a real- world application for a few reasons, however:
* You might be writing cross-platform code that will also need to work on a Mac.
* YoumightbeworkingwithmultipleCALayersubclasses(seeChapter6, “Specialized Layers”) and have no desire to create new UIView subclasses to host them all.
* You might be doing such performance-critical work that even the negligible overhead of maintaining the extra UIView object makes a measurable difference (although in that case, you’ll probably want to use something like OpenGL for your drawing anyway).
But these cases are rare, and in general, layer-backed views are a lot easier to work with than hosted layers.#Summary
This chapter explored the layer tree, a hierarchy of CALayer objects that exists in parallel beneath the UIView hierarchy that forms the iOS interface. We also created our own CALayer and added it to the layer tree as an experiment.
In Chapter 2, “The Backing Image,” we look at the CALayer backing image and at the properties that Core Animation provides for manipulating how it is displayed.
