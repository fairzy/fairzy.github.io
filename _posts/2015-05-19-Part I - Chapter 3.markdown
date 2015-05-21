---
layout: post
title:  "Part I: The Layer Beneath - Chapter 3: Layer Geometry"
date:   2015-05-21 15:23:07
categories: Core Animation
---

> Let no one unversed in geometry enter here.
> 
> --Sign above the entrance to Plato's Academy

***


Chapter 2, “The Backing Image,” introduced the layer-backing image and the properties used to control its position and scaling within the layer bounds. In this chapter, we look at how the layer itself is positioned and sized with respect to its superlayer and siblings. We also explore how to manage your layer’s geometry and how it is affected by autoresizing and autolayout.

<br />

##Layout

UIView has three primary layout properties: frame, bounds, and center. CALayer has equivalents called frame, bounds, and position. Why they used “position” for layers and “center” for views will become clear, but they both represent the same value.
The frame represents the outer coordinates of the layer (that is, the space it occupies within its superlayer), the bounds property represents the inner coordinates (with {0, 0} typically equating to the top-left corner of the layer, although this is not always the case), and the center and position both represent the location of the anchorPoint relative to the superlayer. The anchorPoint property is explained later, but for now just think of it as the center of the layer. Figure 3.1 shows how these properties relate to one another.

The view’s frame, bounds, and center properties are actually just accessors (setter and getter methods) for the underlying layer equivalents. When you manipulate the view frame, you are really changing the frame of the underlying CALayer. You cannot change the view’s frame independently of its layer.

![Figure 3.1 UIView and CALayer coordinate systems (with example values)](http://ww1.sinaimg.cn/mw690/6cdd1b93gw1esc0jf5hauj20he0andg9.jpg)

Figure 3.1 UIView and CALayer coordinate systems (with example values)

The frame is not really a distinct property of the view or layer at all; it is a virtual property, computed from the bounds, position, and transform, and therefore changes when any of those properties are modified. Conversely, changing the frame may affect any or all of those values, as well.

You need to be mindful of this when you start working with transforms, because when a layer is rotated or scaled, its frame reflects the total axially aligned rectangular area occupied by the transformed layer within its parent, which means that the frame width and height may no longer match the bounds (see Figure 3.2).

![Figure 3.2 The effect that rotating a view or layer has on its frame property](http://ww3.sinaimg.cn/mw690/6cdd1b93gw1esc0jfacs8j20hk0bljs9.jpg)

Figure 3.2 The effect that rotating a view or layer has on its frame property

<br />

##anchorPoint

As mentioned earlier, both the view’s center property and the layer’s position property specify the location of the anchorPoint of the layer relative to its superlayer. The anchorPoint property of a layer controls how the layer’s frame is positioned relative to its position property. You can think of the anchorPoint as being the handle used to move the layer around.

By default, the anchorPoint is located in the center of the layer, so that the layer will be centered around that position, wherever it might be. The anchorPoint is not exposed in the UIView interface, which is why the view’s position property is just called “center.” But the anchorPoint of a layer can be moved. You could place it at the top left of the layer frame, for example, so that the layer’s contents would extend down and to the right of its position (see Figure 3.3) instead of being centered on it.

Like the contentsRect and contentsCenter properties covered in Chapter 2, the anchorPoint is specified in unit coordinates, meaning that its coordinates are relative to the dimensions of the layer. The top-left corner of the layer is {0, 0}, and the bottom-rightcorner is {1, 1}. The default (center) position is therefore {0.5, 0.5}. The anchorPoint can be placed outside of the layer bounds by specifying x or y values that are less than zero or greater than one.

Note in Figure 3.3 that when we change the anchorPoint, the position property does not change. Instead, the position remains fixed and the frame of the layer moves.

![Figure 3.3 The effect that changing the anchorPoint has on the frame](http://ww3.sinaimg.cn/mw690/6cdd1b93gw1esc0jfpuq6j20gy0h50tl.jpg)

Figure 3.3 The effect that changing the anchorPoint has on the frame

So, why would we want to change the anchorPoint? We can already place the frame wherever we want, so doesn’t changing the anchorPoint just cause confusion? To illustrate why this might be useful, let's try a practical example. Let’s build an analog clock face with moving hour, minute, and second hands.

The face and hands are constructed using four images (see Figure 3.4). To keep things simple, we’ll load and display these images in the traditional way, using four separate UIImageView instances (although we could do this using regular views by setting their backing layer contents images).

![Figure 3.4 The four images that make up the clock face and hands](http://ww2.sinaimg.cn/mw690/6cdd1b93gw1esc0jg1ipyj20h709at8z.jpg)

Figure 3.4 The four images that make up the clock face and hands

The clock components are arranged in Interface Builder (see Figure 3.5). The image views are nested inside another container view and have both autoresizing and autolayout disabled. This is because autoresizing acts on the view frame, and as demonstrated in Figure 3.2, the frame changes when the view is rotated, which will lead to layout glitches if a rotated view’s frame is resized.

We’ll use an NSTimer to update our clock, and make use of the view’s transform property to rotate the hands. (If you are not familiar with that property, don’t worry; we cover it in Chapter 5, “Transforms.”) Listing 3.1 shows the code for our clock.

![Figure 3.5 Laying out the clock views in Interface Builder](http://ww3.sinaimg.cn/mw690/6cdd1b93gw1esc0lq2c89j20ga09vwfn.jpg)

Figure 3.5 Laying out the clock views in Interface Builder

####Listing 3.1 Clock

***

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIImageView *hourHand;
    @property (nonatomic, weak) IBOutlet UIImageView *minuteHand;
    @property (nonatomic, weak) IBOutlet UIImageView *secondHand;
    @property (nonatomic, weak) NSTimer *timer;
    @end

    @implementation ViewController
    - (void)viewDidLoad {
        [super viewDidLoad];
        
        //start timer
        self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self
                                                    selector:@selector(tick) userInfo:nil
                                                     repeats:YES];//set initial hand positions
        [self tick];
    }
    - (void)tick {
        //convert time to hours, minutes and seconds
        NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
        NSUInteger units = NSHourCalendarUnit | NSMinuteCalendarUnit | NSSecondCalendarUnit;
        NSDateComponents *components = [calendar components:units
                                                   fromDate:[NSDate date]];
        //calculate hour hand angle
        CGFloat hoursAngle = (components.hour / 12.0) * M_PI * 2.0;
        //calculate minute hand angle
        CGFloat minsAngle = (components.minute / 60.0) * M_PI * 2.0;
        //calculate second hand angle
        CGFloat secsAngle = (components.second / 60.0) * M_PI * 2.0;
        //rotate hands
        self.hourHand.transform = CGAffineTransformMakeRotation(hoursAngle);
        self.minuteHand.transform = CGAffineTransformMakeRotation(minsAngle);
        self.secondHand.transform = CGAffineTransformMakeRotation(secsAngle);
    }
    @end

When we run the clock app, it looks a bit strange (see Figure 3.6). The reason for this is that the hand images are rotating around the center of the image, which is not where we would expect a clock’s hand to pivot.

You might think that this could be fixed by adjusting the position of the hand images in Interface Builder, but that won’t work because the images will not rotate correctly if they are not centered on the clock face.

![Figure 3.6 The clock face, with misaligned hands](http://ww1.sinaimg.cn/mw690/6cdd1b93gw1esc0lqgfpvj20hl08yaam.jpg)

Figure 3.6 The clock face, with misaligned hands

A solution that would work is to add additional transparent space at the bottom of all the images, but that makes the images larger than they need to be, and they will consume more memory as a result. That’s not very elegant.

A better solution is to make use of the anchorPoint property. Let’s add some additional lines of code to our -viewDidLoad method to offset the anchorPoint for each of our clock hands (see Listing 3.2). Figure 3.7 shows the correctly aligned hands.

####Listing3.2 ClockwithAdjustedanchorPointValues

***

    - (void)viewDidLoad {
        [super viewDidLoad];
        //adjust anchor points
        self.secondHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f); 
        self.minuteHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f); 
        self.hourHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f);
        //start timer
        ... 
    }

![Figure 3.7 The clock face, with correctly aligned hands](http://ww2.sinaimg.cn/mw690/6cdd1b93gw1esc0lqwlesj20hu08hq3i.jpg)

Figure 3.7 The clock face, with correctly aligned hands

<br />

##Coordinate Systems












Figure 3.8 The green view is positioned underneath the red one in the view hierarchy.

We would expect this drawing order to be reflected in the actual app, as well, but if we increase the zPosition of the green view (see Listing 3.3), we find that the order is reversed (see Figure 3.9). Note that we don’t need to increase it by much; views are infinitely thin, so even a 1-point increase in the zPosition brings the green view in front of the red one. A smaller value such as 0.1 or 0.0001 would also work, but be wary of using very tiny values because this can lead to visual glitches as a result of rounding errors in the floating-point calculations.

####Listing 3.3 Adjusting zPosition to Change the Display Order

***

	@interface ViewController ()
	@property (nonatomic, weak) IBOutlet UIView *redView;



Figure 3.9 The green view is drawn in front of the red view.

<br />

###Hit Testing






	@property (nonatomic, weak) CALayer *blueLayer;

	
		self.blueLayer.backgroundColor = [UIColor blueColor].CGColor;
	}
	
	
	
		{
			{
				otherButtonTitles:nil] show];


Figure 3.10 The touched layer is correctly identified.

The -hitTest: method also accepts a CGPoint; but instead of a BOOL, it returns either the layer itself or the deepest sublayer containing the point. This means that you do not need to transform and test the point against each sublayer in turn manually, as you do when using the -containsPoint: method. If the point lies outside of the outermost layer’s bounds, it returns nil. Listing 3.5 shows the code for determining the touched layer by using the -hitTest: method.

####Listing 3.5 Determining the Touched Layer Using hitTest:

***

	- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
	    
	    
	    {
	                                    message:nil
	                          cancelButtonTitle:@"OK"
	         show];
	    {
	                                    message:nil
	                                   delegate:nil
	                          cancelButtonTitle:@"OK"
	         show];
	}


<br/>









<br />



