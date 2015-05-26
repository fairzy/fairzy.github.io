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

Part I covered just about everything that Core Animation can do, apart from animation. Animation is a 
pretty significant part of the Core Animation framework. In this chapter, we take a look at how it 
works. Specifically, we explore implicit animations, which are animations that the framework performs 
automatically (unless you tell it not to).

##Transactions

Core Animation is built on the assumption that everything you do onscreen will (or at least may) be animated. Animation is not something that you enable in Core Animation. Animations have to be explicitly turned off; otherwise, they happen all the time.

Whenever you change an animatable property of a CALayer, the change is not reflected immediately onscreen. Instead, the layer property animates smoothly from the previous value to the new one. You don’t have to do anything to make this happen; it’s the default behavior.

This might seem a bit too good to be true, so let’s demonstrate it with an example: We’ll take the blue square project from Chapter 1, “The Layer Tree,” and add a button that will set the layer to a random color. Listing 7.1 shows the code for this. Tap the button and you will see that the color changes smoothly instead of jumping to its new value (see Figure 7.1).

####Listing 7.1 Randomizing the Layer Color 

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIView *layerView; 
    @property (nonatomic, weak) IBOutlet CALayer *colorLayer;
    @end

    @implementation ViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        //create sublayer
        self.colorLayer = [CALayer layer];
        self.colorLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f); 
        self.colorLayer.backgroundColor = [UIColor blueColor].CGColor;
        //add it to our view
        [self.layerView.layer addSublayer:self.colorLayer]; 
    }

    - (IBAction)changeColor {
        //randomize the layer background color
        CGFloat red = arc4random() / (CGFloat)INT_MAX;
        CGFloat green = arc4random() / (CGFloat)INT_MAX;
        CGFloat blue = arc4random() / (CGFloat)INT_MAX; 
        self.colorLayer.backgroundColor = [UIColor colorWithRed:red 
                                                          green:green 
                                                           blue:blue
                                                          alpha:1.0].CGColor;
    }
    @end

![](http://ww1.sinaimg.cn/mw690/6cdd1b93gw1eshsunejxaj20n80biab0.jpg)

Figure 7.1 Adding a button to change the layer color

This kind of animation is known as implicit animation. It is implicit because we are not specifying what kind of animation we want to happen; we just change a property, and Core Animation decides how and when to animate it. Core Animation also supports explicit animation, which is covered in the next chapter.

When you change a property, how does Core Animation determine the type and duration of the animation that it will perform? The duration of the animation is specified by the settings for the current transaction, and the animation type is controlled by layer actions.

Transactions are the mechanism that Core Animation uses to encapsulate a particular set of property animations. Any animatable layer properties that are changed within a given transaction will not change immediately, but instead will begin to animate to their new value as soon as that transaction is committed.

Transactions are managed using the CATransaction class. The CATransaction class has a peculiar design in that it does not represent a single transaction as you might expect from the name, but rather it manages a stack of transactions without giving you direct access to them. CATransaction has no properties or instance methods, and you can’t create a transaction using +alloc and -init as normal. Instead, you use the class methods +begin and +commit to push a new transaction onto the stack or pop the current one, respectively.

Any layer property change that can be animated will be added to the topmost transaction in the stack. You can set the duration of the current transaction’s animations by using the +setAnimationDuration: method, or you can find out the current duration using the +animationDuration method. (The default is 0.25 seconds.)Core Animation automatically begins a new transaction with each iteration of the run loop. (The run loop is where iOS gathers user input, handles any outstanding timer or network events, and then eventually redraws the screen.) Even if you do not explicitly begin a transaction using `[CATransaction begin]`, any property changes that you make within a given run loop iteration will be grouped together and then animated over a 0.25-second period.

Armed with this knowledge, we can easily change the duration of our color animation. It would be sufficient to change the animation duration of the current (default) transaction by using the +setAnimationDuration: method, but we will start a new transaction first so that changing the duration doesn’t have any unexpected side effects. Changing the duration of the current transaction might possibly affect other animations that are incidentally happening at the same time (such as screen rotation), so it is always a good idea to push a new transaction explicitly before adjusting the animation settings.

Listing 7.2 shows the modified code. If you run the app, you will notice that the color fade happens much more slowly than before.

####Listing 7.2 Controlling Animation Duration Using CATransaction

    - (IBAction)changeColor {
        //begin a new transaction
        [CATransaction begin];
        //set the animation duration to 1 second
        [CATransaction setAnimationDuration:1.0];
        //randomize the layer background color
        CGFloat red = arc4random() / (CGFloat)INT_MAX;
        CGFloat green = arc4random() / (CGFloat)INT_MAX;
        CGFloat blue = arc4random() / (CGFloat)INT_MAX; 
        self.colorLayer.backgroundColor = [UIColor colorWithRed:red 
                                                          green:green 
                                                           blue:blue
        alpha:1.0].CGColor;
        //commit the transaction
        [CATransaction commit]; 
    }

If you’ve ever done any animation work using the UIView animation methods, this pattern should look familiar. UIView has two methods, `+beginAnimations:context:` and `+commitAnimations`, that work in a similar way to the `+begin` and `+commitmethods` on CATransaction. Any view or layer properties you change between calls to `+beginAnimations:context:` and `+commitAnimations` will be animated automatically because what those UIView animation methods are actually doing is setting up a CATransaction.

In iOS 4, Apple added a new block-based animation method to UIView, `+animateWithDuration:animations:`. This is syntactically a bit cleaner than having separate methods to begin and end a block of property animations, but really it’s just doing the same thing behind the scenes.

The CATransaction +begin and +commit methods are called internally by the `+animateWithDuration:animations:` method, with the body of the animations block executed in between them so that any property changes that you make inside the block will be encapsulated by the transaction. This has the benefit of avoiding any risk of mismatched `+begin` and `+commit` calls due to developer error.

##Completion Blocks

The UIView block-based animation allows you to supply a completion block to be called when the animation has finished. This same feature is available when using the CATransaction interface by calling the +setCompletionBlock: method. Let’s adapt our example again so that it performs an action when the color change has completed. We’ll attach a completion block and use it to trigger a second animation that spins the layer 90 degrees each time the color has changed. Listing 7.3 shows the code, and Figure 7.2 shows the result.

####Listing 7.3 Adding a Callback When the Color Animation Completes

    - (IBAction)changeColor {
        //begin a new transaction
        [CATransaction begin];
        //set the animation duration to 1 second
        [CATransaction setAnimationDuration:1.0]; 
        //add the spin animation on completion
        [CATransaction setCompletionBlock:^{
            //rotate the layer 90 degrees
            CGAffineTransform transform = self.colorLayer.affineTransform; 
            transform = CGAffineTransformRotate(transform, M_PI_2); 
            self.colorLayer.affineTransform = transform;
        }];
        //randomize the layer background color
        CGFloat red = arc4random() / (CGFloat)INT_MAX;
        CGFloat green = arc4random() / (CGFloat)INT_MAX;
        CGFloat blue = arc4random() / (CGFloat)INT_MAX; 
        self.colorLayer.backgroundColor = [UIColor colorWithRed:red
                                                          green:green 
                                                           blue:blue
                                                          alpha:1.0].CGColor;
        //commit the transaction
        [CATransaction commit]; 
    }

![](http://ww2.sinaimg.cn/mw690/6cdd1b93gw1eshtv37pt2j20n00bljse.jpg)

Figure 7.2 A rotation animation applied after the color fade has finished

Notice that our rotation animation is much faster than our color fade animation. That’s because the completion block that applies the rotation animation is executed after the color fade animation’s transaction has been committed and popped off the stack. It is, therefore, using the default transaction, with the default animation duration of 0.25 seconds.

##Layer Actions

Now let’s try an experiment: Instead of animating a standalone sublayer, we’ll try directly animating the backing layer of our view. Listing 7.4 shows an adapted version of the codefrom Listing 7.2 that removes the colorLayer and sets the layerView backing layer’s background color directly.

####Listing 7.4 Setting the Property of the Backing Layer Directly 

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIView *layerView; 
    @end

    @implementation ViewController
    - (void)viewDidLoad {
        [super viewDidLoad];
        //set the color of our layerView backing layer directly
        self.layerView.layer.backgroundColor = [UIColor blueColor].CGColor; 
    }
    - (IBAction)changeColor {
        //begin a new transaction
        [CATransaction begin];
        //set the animation duration to 1 second
        [CATransaction setAnimationDuration:1.0];
        //randomize the layer background color
        CGFloat red = arc4random() / (CGFloat)INT_MAX;
        CGFloat green = arc4random() / (CGFloat)INT_MAX;
        CGFloat blue = arc4random() / (CGFloat)INT_MAX; 
        self.layerView.layer.backgroundColor = [UIColor colorWithRed:red
                                                               green:green 
                                                                blue:blue
        alpha:1.0].CGColor;
        //commit the transaction
        [CATransaction commit]; 
    }

If you run the project, you’ll notice that the color snaps to its new value immediately when the button is pressed instead of animating smoothly as before. What’s going on? The implicit animation seems to have been disabled for the UIView backing layer.

Come to think of it, we’d probably have noticed if UIView properties always animated automatically whenever we modified them. So, if UIKit is built on top of Core Animation (which always animates everything by default), how come implict animations are disabled by default in UIKit?

We know that Core Animation normally animates any property change of a CALayer (provided it can be animated) and that UIView somehow turns this behavior off for its backing layer. To understand how it does that, we need to understand how implicit animations are implemented in the first place.

The animations that CALayer automatically applies when properties are changed are called actions. When a property of a CALayer is modified, it calls its -actionForKey: method, passing the name of the property in question. What happens next is quite nicely documented in the header file for CALayer, but it essentially boils down to this:

1. The layer first checks whether it has a delegate and if the delegate implements the -actionForLayer:forKey method specified in the CALayerDelegate protocol. If it does, it will call it and return the result.
2. If there is no delegate, or the delegate does not implement -actionForLayer:forKey, the layer checks in its actions dictionary, which contains a mapping of property names to actions.
3. If the actions dictionary does not contain an entry for the property in question, the layer searches inside its style dictionary hierarchy for any actions that match the property name.
4. Finally, if it fails to find a suitable action anywhere in the style hierarchy, the layer will fall back to calling the -defaultActionForKey: method, which defines standard actions for known properties.
The result of this exhaustive search will be that -actionForKey: either returns nil (in which case, no animation will take place and the property value will change immediately) or an object that conforms to the CAAction protocol, which CALayer will then use to animate between the previous and current property values.

And that explains how UIKit disables implicit animations: Every UIView acts as the delegate for its backing layer and provides an implementation for the -actionForLayer:forKey method. When not inside an animation block, UIView returns nil for all layer actions, but within the scope of an animation block it returns non- nil values. We can demonstrate this with a simple experiment (see Listing 7.5).


