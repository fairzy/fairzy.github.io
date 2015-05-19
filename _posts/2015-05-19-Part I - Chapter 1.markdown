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



###`CALayer`

	The CALayer class is conceptually very similar to UIView. Layers, like views, are rectangular objects that can be arranged into a hierarchical tree. Like views, they can contain content (such as an image, text, or a background color) and manage the position of their children (sublayers). They have methods and properties for performing animations and transforms. The only major feature of UIView that isn’t handled by CALayer is user interaction.

`CALayer`这个类在概念上与`UIView`非常类似，Layers也是可以被组成层级树的矩形对象，像Views一样，可以包含不同的内容（比如图片，文字，或者背景颜色）并且设置他们子Layer的坐标，他们有一些property可以用来执行动画和3d变换。CALayer与UIView最大的不同在于，CALayer不处理用户事件。





![图 1.2 layertree的层级结构（左边）与view的层级结构（右边）一一对应的](http://ww1.sinaimg.cn/large/6cdd1b93gw1es9h6jfsbsj20g20b974x.jpg)

图 1.2 layertree的层级结构（左边）与view的层级结构（右边）一一对应的


	It is actually these backing layers that are responsible for the display and animation of everything you see onscreen. UIView is simply a wrapper, providing iOS-specific functionality such as touch handling and high-level interfaces for some of Core Animation’s low-level functionality.

你在屏幕上看到的所有内容和动画，其实都是这些backing layer在负责。UIView只是一个简单的封装，提供了针对iOS的触摸事件支持，以及对底层动画的高级封装。








一定程度上，你说的的确没错。在简单的情况下，我们确实不需要直接跟CALayer打交道，因为苹果已经把强大的动画直接封装为简单的UIView接口提供给我们了。

但是，这样也少了很多灵活性，如果你想做点非常规的的事，或者评估没有暴露接口的事儿，你就没有偶选择了，只能去深入研究一下Core Animation的底层选项。
	* 3D transforms and positioning



* 3D变换
* 非矩形的区域
* 内容的Alpah通道蒙版
* 多步的，非线性的动画

我们将会在接下来的章节中讨论这些特性，现在，让我们看一下CALyer在app中可以怎样应用。




让我们创建一个简单的项目，并且操作一下layer的属性，在xcode里面建一个Single View的模板建一个iOS的项目。




Figure1.3 A white UIView on a gray background


	That’s not very exciting, so let’s add a splash of color. We’ll place a small blue square inside the white one.

这个不怎么激动人心啊，下面让我们来加点颜色吧，让我们在白色矩形中放一个蓝色的矩形吧。





![Figure 1.4 Adding the QuartzCore framework to the project](http://ww2.sinaimg.cn/large/6cdd1b93gw1es9hbl2mmqj20g708sq40.jpg)

Figure 1.4 Adding the QuartzCore framework to the project


	After doing that, we can directly reference CALayer and its properties and methods in our code. In Listing 1.1, we create a new CALayer programmatically, set its backgroundColor property, and then add it as a sublayer to the layerView backing layer. (The code assumes that we created the view using Interface Builder and that we have already linked up the layerView outlet.) Figure 1.5 shows the result.

完成这些之后，我们就可以直接访问CALayer的属性和方法了。在列表1.1中，我们用代码创建了一个新的CALayer，设置了它的背景颜色，并把它作为一个子layer添加到layerView的backing layer中（这里的代码假设你用的是IB并且已经吧IB里面的view连接到代码的IBOutlet了）。图1.5显示了运行结果。


CALyer的backgroundColor属性是CGColorRef类型，并不像UIView一样是UIColor，所以我们需要UIColor的CGColor属性。你也可以用Core Graphics的方法直接创建一个CGColor，但是用UIColor，你可以不用担心color的内存释放问题。


		blueLayer.backgroundColor = [UIColor blueColor].CGColor;


Figure1.5 A small blue CALayer nested inside a white UIView


	A view has only one backing layer (created automatically) but can host an unlimited number of additional layers. As Listing 1.1 shows, you can explicitly create standalone layers and add them directly as sublayers of the backing layer of a view. Although it ispossible to add layers in this way, more often than not you will simply work with views and their backing layers and won’t need to manually create additional hosted layers.

一个View只能有一个backing layer（自动创建的）但是可以有无限个子layer，就像在代码1.1中，我们显示的创建了一个单独的layer并把它直接添加到view的backing layer中，但通常，我们只是与view和他们的backing layer打交道，并不需要手动创建一些layer


在10.8之前的Mac OS中，用多个有backing layer的view去构成一个view层级结构比用一个view，然后view中有是一个layer tree的方式有严重的性能问题。但是在iOS中，UIView是轻量级的，用layers工作并不会对性能造成负面影响（在10.8的Mac OS版本中，NSView的这个性能问题已经极大的被解决了）。

用view的backing layer而不是单独建一个根layer的好处是，你可以在获得CALayer的底层功能时，也可以获取UIView的高级封装api（比如autoresizing，autolayout以及事件处理）。	



* 你可能需要用到CALyer的多个子类（参考第6章，“定制的Layer”）并且根本不需要创建UIView的子类来托管他们。
* 你可能再做一些对性能要求非常高的的工作，即使维护UIView的对象这样微不足道的工作都会产生可以测量的性能影响（尽管在那种情况下，你可能需要用OpenGL来绘制）。






在第2章，“底层图像”中，我们将会看一下CALayer的底层图像（backing image）和CA提供的操控它如何显示的一些属性。