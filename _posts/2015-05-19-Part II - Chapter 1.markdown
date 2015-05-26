---
layout: post
title:  "Part II: Setting Things In Motion - Chapter 7: Implicit Animations"
date:   2015-05-21 15:23:07
categories: Core Animation
---

> Do what I mean, not what I say.
>
> --Edna Krabappel, The Simpsons

***

Part I covered just about everything that Core Animation can do, apart from animation. Animation is a pretty significant part of the Core Animation framework. In this chapter, we take a look at how it works. Specifically, we explore implicit animations, which are animations that the framework performs automatically (unless you tell it not to).

第一章我们几乎涵盖了 Core Animation 能做的事儿，除了动画。动画是Core Animation 的非常重要的部分。这一章，我们来看一下动画是如何工作的，我们会

##Transactions

Core Animation is built on the assumption that everything you do onscreen will (or at least may) be animated. Animation is not something that you enable in Core Animation. Animations have to be explicitly turned off; otherwise, they happen all the time.

Whenever you change an animatable property of a CALayer, the change is not reflected immediately onscreen. Instead, the layer property animates smoothly from the previous value to the new one. You don’t have to do anything to make this happen; it’s the default behavior.

This might seem a bit too good to be true, so let’s demonstrate it with an example: We’ll take the blue square project from Chapter 1, “The Layer Tree,” and add a button that will set the layer to a random color. Listing 7.1 shows the code for this. Tap the button and you will see that the color changes smoothly instead of jumping to its new value (see Figure 7.1).

