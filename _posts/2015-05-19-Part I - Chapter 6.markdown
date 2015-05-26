---
layout: post
title:  "Part I: The Layer Beneath - Chapter 6: Specialized Layers"
date:   2015-05-21 15:23:07
categories: Core Animation
---

> Specialization is a feature of every complex organization.
>
> --Catharine R. Stimpson

***

Up to this point, we have been working with the CALayer class, and we have seen that it has some useful image drawing and transformation capabilities. But Core Animation layers can be used for more than just images and colors. This chapter explores some of the other layer classes that you can use to extend Core Animation’s drawing capabilities.

到现在，我们已经跟CALayer类愉快的玩耍过了，已经学习了它的一些有用的图片绘制和变换能力。但是layers可不只是用来展示图片和颜色，这一章我们将会探索一下具有其他能力的layer类。

<br />

##CAShapeLayer

In Chapter 4, “Visual Effects,” you learned how to use CGPath to create arbitrarily shaped shadows without using images. It would be neat if we could create arbitrarily shaped layers in the same way.

在第4章，“Visual Effects”中，我们学习了如何使用CGPath不使用图片来创建任意形状的阴影。如果我们能用相同方式创建任意形状的layer就好了。

CAShapeLayer is a layer subclass that draws itself using vector graphics instead of a bitmap image. You specify attributes such as color and line thickness, define the desired shape using a CGPath, and CAShapeLayer renders it automatically. Of course, you could use Core Graphics to draw a path directly into the contents of an ordinary CALayer (as in Chapter 2, “The Backing Image”), but there are several advantages to using CAShapeLayer instead:

CAShapeLayer是一个子类，它使用图形向量而不是图片来绘制内容。你指定颜色和线条宽度，用CGPath定义想要的形状，CAShapeLayer就可以自动渲染这些内容。当然你月可以用Core Graphics来直接把内容绘制到一个普通的CALayer上（就像第二章：“The Backing Image”中展示的），但是用CAShapeLayer有几种好处：

* It’s fast — CAShapeLayer uses hardware-accelerated drawing and is much faster than using Core Graphics to draw an image.
* It’s memory efficient — A CAShapeLayer does not have to create a backing image like an ordinary CALayer does, so no matter how large it gets, it won’t consume much memory.
* It doesn't get clipped to the layer bounds — A CAShapeLayer can happily draw outside of its bounds. Your path will not get clipped like it does when you draw into a regular CALayer using Core Graphics (as you saw in Chapter 2).
* There's no pixelation — When you transform a CAShapeLayer by scaling it up or moving it closer to the camera with a 3D perspective transform, it does not become pixelated like an ordinary layer’s backing image would.

* CAShapeLayer更快 - 它使用硬件加速来绘制内容，这样就比用Core Graphics绘制图片快多了。
* 更节省内存 - 一个CAShapeLayer不像普通CALayer一样创建backing image，所以不管多大尺寸，它都不会消耗太多内存。
* 不像普通layer一样，会被layer的边界裁剪，（就像第二章看到的）
* 不会模糊，当你把一个CAShapeLayer放大或者把它用3D透视变换像camera移动时，他不会像使用image作为backing layer的普通layer一样变得像素模糊。

###Creating a CGPath

CAShapeLayer can be used to draw any shape that can be represented by a CGPath. The shape doesn’t have to be closed, and the path doesn’t have to be unbroken, so you can actually draw several distinct shapes in a single layer. You can control the strokeColor and fillColor of the path, along with other properties such as lineWidth (line thickness, in points), lineCap (how the ends of lines look), and lineJoin (how the joints between lines look); but you can only set these properties once, at the layer level. If you want to draw multiple shapes with different colors or styles, you have to use a separate layer for each shape.

Listing 6.1 shows the code for a simple stick figure drawing, rendered using a single CAShapeLayer. The CAShapeLayer path property is defined as a CGPathRef, but we’ve created the path using the UIBezierPath helper class, which saves us from having to worry about manually releasing the CGPath. Figure 6.1 shows the result. It’s not exactly a Rembrandt, but you get the idea!

####Listing 6.1 Drawing a Stick Figure Using CAShapeLayer 

***

    #import "DrawingView.h"
    #import <QuartzCore/QuartzCore.h>
    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIView *containerView; 
    @end

    @implementation ViewController
    - (void)viewDidLoad {
        [super viewDidLoad];
        //create path
        UIBezierPath *path = [[UIBezierPath alloc] init]; 
        [path moveToPoint:CGPointMake(175, 100)];
        [path addArcWithCenter:CGPointMake(150, 100) 
                        radius:25
                    startAngle:0 endAngle:2*M_PI
                     clockwise:YES];
        [path moveToPoint:CGPointMake(150, 125)];
        [path addLineToPoint:CGPointMake(150, 175)]; 
        [path addLineToPoint:CGPointMake(125, 225)]; 
        [path moveToPoint:CGPointMake(150, 175)]; 
        [path addLineToPoint:CGPointMake(175, 225)]; 
        [path moveToPoint:CGPointMake(100, 150)]; 
        [path addLineToPoint:CGPointMake(200, 150)];
        //create shape layer
        CAShapeLayer *shapeLayer = [CAShapeLayer layer]; 
        shapeLayer.strokeColor = [UIColor redColor].CGColor; 
        shapeLayer.fillColor = [UIColor clearColor].CGColor; 
        shapeLayer.lineWidth = 5;
        shapeLayer.lineJoin = kCALineJoinRound; 
        shapeLayer.lineCap = kCALineCapRound; 
        shapeLayer.path = path.CGPath;
        //add it to our view
        [self.containerView.layer addSublayer:shapeLayer]; 
    }
    @end

![](http://ww1.sinaimg.cn/mw690/6cdd1b93gw1esglp2raxlj20gh07wjru.jpg)

Figure 6.1 A simple stick figure displayed using CAShapeLayer

###Rounded Corners, Redux

Chapter 2 mentioned that CAShapeLayer provides an alternative way to create a view with rounded corners, as opposed to using the CALayer cornerRadius property. Although using a CAShapeLayer is a bit more work, it has the advantage that it allows us to specify the radius of each corner independently.
We could create a rounded rectangle path manually using individual straight lines and arcs, but UIBezierPath actually has some convenience constructors for creating rounded rectangles automatically. The following code snippet produces a path with three rounded corners and one sharp:

    //define path parameters
    CGRect rect = CGRectMake(50, 50, 100, 100); 
    CGSize radii = CGSizeMake(20, 20); 
    UIRectCorner corners = UIRectCornerTopRight | UIRectCornerBottomRight | UIRectCornerBottomLeft;
    //create path
    UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:rect byRoundingCorners:corners cornerRadii:radii];

We can use a CAShapeLayer with this path to create a view with mixed sharp and rounded corners. If we want to clip the view’s contents to this shape, we can use our CAShapeLayer as the mask property of the view’s backing layer instead of adding it as a sublayer. (See Chapter 4, “Visual Effects,” for a full explanation of layer masks.)

##CATextLayer

A user interface cannot be constructed from images alone. A well-designed icon can do a great job of conveying the purpose of a button or control, but sooner or later you’re going to need a good old-fashioned text label.

If you want to display text in a layer, there is nothing stopping you from using the layer delegate to draw a string directly into the layer contents with Core Graphics (which is essentially how UILabel works). This is a cumbersome approach if you are working directly with layers, though, instead of layer-backed views. You would need to create a class that can act as the layer delegate for each layer that displays text, and then write logic to determine which layer should display which string, not to mention keeping track of the different fonts, colors, and so on.

Fortunately, this is unnecessary. Core Animation provides a subclass of CALayer called CATextLayer that encapsulates most of the string drawing features of UILabel in layer form and adds a few extra features for good measure.

CATextLayer also renders much faster than UILabel. It’s a little-known fact that on iOS6 and earlier, UILabel actually uses WebKit to do its text drawing, which carries a significant performance overhead when you are drawing a lot of text. CATextLayer uses Core Text and is significantly faster.

Let’s try displaying some text using a CATextLayer. Listing 6.2 shows the code to set up and display a CATextLayer, and Figure 6.2 shows the result.

####Listing 6.2 Implementing a Text Label Using CATextLayer 

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIView *labelView; 
    @end

    @implementation ViewController
    - (void)viewDidLoad {
        [super viewDidLoad];
        //create a text layer
        CATextLayer *textLayer = [CATextLayer layer]; 
        textLayer.frame = self.labelView.bounds; 
        [self.labelView.layer addSublayer:textLayer];
        //set text attributes
        textLayer.foregroundColor = [UIColor blackColor].CGColor; 
        textLayer.alignmentMode = kCAAlignmentJustified; 
        textLayer.wrapped = YES;
        //choose a font
        UIFont *font = [UIFont systemFontOfSize:15];
        //set layer font
        CFStringRef fontName = (__bridge CFStringRef)font.fontName; 
        CGFontRef fontRef = CGFontCreateWithFontName(fontName); 
        textLayer.font = fontRef;
        textLayer.fontSize = font.pointSize; CGFontRelease(fontRef);
        //choose some text
        NSString *text = @"Lorem ipsum dolor sit amet, consectetur adipiscing \ 
        elit. Quisque massa arcu, eleifend vel varius in, facilisis pulvinar \ 
        leo. Nunc quis nunc at mauris pharetra condimentum ut ac neque. Nunc \
        elementum, libero ut porttitor dictum, diam odio congue lacus, vel \ 
        fringilla sapien diam at purus. Etiam suscipit pretium nunc sit amet \ 
        lobortis";
        //set layer text
        textLayer.string = text; 
    }
    @end

![](http://ww3.sinaimg.cn/mw690/6cdd1b93gw1esglscqu5gj20jk09wwfh.jpg)

Figure 6.2 A plain text label implemented using CATextLayer

If you look at this text closely, you’ll see that something is a bit odd; the text is pixelated. That’s because it’s not being rendered at Retina resolution. Chapter 2 mentioned the contentsScale property, which is used to determine the resolution at which the layer contents are rendered. The contentsScale property always defaults to 1.0 instead of the screen scale factor. If we want Retina-quality text, we have to set the contentsScale of our CATextLayer to match the screen scale using the following line of code:

    textLayer.contentsScale = [UIScreen mainScreen].scale; 

This solves the pixelation problem (see Figure 6.3).

![](http://ww3.sinaimg.cn/mw690/6cdd1b93gw1esglta6tljj20ji0azmyc.jpg)

Figure 6.3 The effect of setting the contentsScale to match the screen

The CATextLayer font property is not a UIFont, it’s a CFTypeRef. This allows you to specify the font using either a CGFontRef or a CTFontRef (a Core Text font), depending on your requirements. The font size is also set independently using the fontSize property, because CTFontRef and CGFontRef do not encapsulate the point size like UIFont does. The example shows how to convert from a UIFont to a CGFontRef.

Also, the CATextLayer string property is not an NSString as you might expect, but is typed as id. This is to allow you the option of using an NSAttributedString instead of an NSString to specify the text (NSAttributedString is not a subclass of NSString). Attributed strings are the mechanism that iOS uses for rendering styled text. They specify style runs, which are specific ranges of the string to which metadata such as font, color, bold, italic, and so forth are attached.

###Rich Text

In iOS 6, Apple added direct support for attributed strings to UILabel and to other UIKit text views. This is a handy feature that makes attributed strings much easier to work with, but CATextLayer has supported attributed strings since its introduction in iOS 3.2; so if you still need to support earlier iOS versions with your app, CATextLayer is a great way to add simple rich text labels to your interface without having to deal with the complexity of Core Text or the hassle of using a UIWebView.

Let’s modify the example to use an NSAttributedString (see Listing 6.3). On iOS 6 and above we could use the new NSTextAttributeName constants to set up our stringattributes, but because the point of the exercise is to demonstrate that this feature also works on iOS 5 and below, we’ve used the Core Text equivalents instead. This means that you’ll need to add the Core Text framework to your project; otherwise, the compiler won’t recognize the attribute constants.

Figure 6.4 shows the result. (Note the red, underlined text.)

####Listing 6.3 Implementing a Rich Text Label Using NSAttributedString

    #import "DrawingView.h"
    #import <QuartzCore/QuartzCore.h> 
    #import <CoreText/CoreText.h>

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIView *labelView; 
    @end

    @implementation ViewController
    - (void)viewDidLoad {
        [super viewDidLoad];
        //create a text layer
        CATextLayer *textLayer = [CATextLayer layer]; 
        textLayer.frame = self.labelView.bounds; 
        textLayer.contentsScale = [UIScreen mainScreen].scale; 
        [self.labelView.layer addSublayer:textLayer];
        //set text attributes
        textLayer.alignmentMode = kCAAlignmentJustified; 
        textLayer.wrapped = YES;
        //choose a font
        UIFont *font = [UIFont systemFontOfSize:15];
        //choose some text
        NSString *text = @"Lorem ipsum dolor sit amet, consectetur adipiscing \ 
        elit. Quisque massa arcu, eleifend vel varius in, facilisis pulvinar \ 
        leo. Nunc quis nunc at mauris pharetra condimentum ut ac neque. Nunc \ 
        elementum, libero ut porttitor dictum, diam odio congue lacus, vel \ 
        fringilla sapien diam at purus. Etiam suscipit pretium nunc sit amet \ 
        lobortis";
        //create attributed string
        NSMutableAttributedString *string = nil;
        string = [[NSMutableAttributedString alloc] initWithString:text];
        //convert UIFont to a CTFont
        CFStringRef fontName = (__bridge CFStringRef)font.fontName; 
        CGFloat fontSize = font.pointSize;
        CTFontRef fontRef = CTFontCreateWithName(fontName, fontSize, NULL);
        //set text attributes
        NSDictionary *attribs = @{
            (__bridge id)kCTForegroundColorAttributeName:(__bridge id)[UIColor blackColor].CGColor, 
            (__bridge id)kCTFontAttributeName: (__bridge id)fontRef
        };
        [string setAttributes:attribs range:NSMakeRange(0, [text length])]; 
        attribs = @{
            (__bridge id)kCTForegroundColorAttributeName: (__bridge id)[UIColor redColor].CGColor, 
            (__bridge id)kCTUnderlineStyleAttributeName: @(kCTUnderlineStyleSingle),
            (__bridge id)kCTFontAttributeName: (__bridge id)fontRef
        };
        [string setAttributes:attribs range:NSMakeRange(6, 5)];
        //release the CTFont we created earlier
        CFRelease(fontRef);
        //set layer text
        textLayer.string = string; 
    }
    @end

![](http://ww2.sinaimg.cn/mw690/6cdd1b93gw1esglx1lx3sj20jn09tt9t.jpg)

Figure 6.4 A rich text label implemented using CATextLayer

###Leading and Kerning

It’s worth mentioning that the leading (line spacing) and kerning (spacing between letters) for text rendered using CATextLayer is not completely identical to that of the string rendering used by UILabel due to the different drawing implementations (Core Text and WebKit, respectively).
The extent of the discrepancy varies (depending on the specific font and characters used) and is generally fairly minor, but you should keep this mind if you are trying to exactly match appearance between regular labels and a CATextLayer.

###A `UILabel` Replacement

We’ve established that CATextLayer has performance benefits over UILabel, as well as some additional layout options and support for rich text on iOS 5. But it’s fairly cumbersome to use by comparison to a regular label. If we want to make a truly usable replacement for UILabel, we should be able to create our labels in Interface Builder, and they should behave as much as possible like regular views.
We could subclass UILabel and override its methods to display the text in a CATextLayer that we’ve added as a sublayer, but we’d still have the redundant empty backing image created by the presence of UILabel -drawRect: method. And because CALayer doesn’t support autoresizing or autolayout, a sublayer wouldn’t track the size of the view bounds automatically, so we would need to manually update the sublayer bounds every time the view is resized.What we really want is a UILabel subclass that actually uses a CATextLayer as its backing layer, then it would automatically resize with the view and there would be no redundant backing image to worry about.

As we discussed in Chapter 1, “The Layer Tree,” every UIView is backed by an instance of CALayer. That layer is automatically created and managed by the view, so how can we substitute a different layer type? We can’t replace the layer once it has been created, but if we subclass UIView, we can override the +layerClass method to return a different layer subclass at creation time. UIView calls the +layerClass method during its initialization, and uses the class it returns to create its backing layer.

Listing 6.4 shows the code for a UILabel subclass called LayerLabel that draws its text using a CATextLayer instead of the using the slower –drawRect: approach that an ordinary UILabel uses. LayerLabel instances can either be created program- matically or via Interface Builder by adding an ordinary label to the view and setting its class to LayerLabel.

####Listing 6.4 LayerLabel, a UILabel Subclass That Uses CATextLayer

    #import "LayerLabel.h"
    #import <QuartzCore/QuartzCore.h> 

    @implementation LayerLabel

    + (Class)layerClass {
        //this makes our label create a CATextLayer 
        //instead of a regular CALayer for its backing layer 
        return [CATextLayer class];
    }

    - (CATextLayer *)textLayer 
    {
        return (CATextLayer *)self.layer; 
    }

    - (void)setUp {
        //set defaults from UILabel settings
        self.text = self.text; self.textColor = self.textColor;
         self.font = self.font;
        //we should really derive these from the UILabel settings too 
        //but that's complicated, so for now we'll just hard-code them 
        [self textLayer].alignmentMode = kCAAlignmentJustified;

        [self textLayer].wrapped = YES; [self.layer display];
    }

    - (id)initWithFrame:(CGRect)frame 
    {
        //called when creating label programmatically
        if (self = [super initWithFrame:frame]) 
        {
            [self setUp]; 
        }
        return self; 
    }

    - (void)awakeFromNib {
        //called when creating label using Interface Builder
        [self setUp]; 
    }
    - (void)setText:(NSString *)text {
        super.text = text;
        //set layer text
        [self textLayer].string = text; 
    }
    - (void)setTextColor:(UIColor *)textColor {
        super.textColor = textColor;
        //set layer text color
        [self textLayer].foregroundColor = textColor.CGColor; 
    }
    - (void)setFont:(UIFont *)font {
        super.font = font;
        //set layer font
        CFStringRef fontName = (__bridge CFStringRef)font.fontName; 
        CGFontRef fontRef = CGFontCreateWithFontName(fontName); 
        [self textLayer].font = fontRef;
        [self textLayer].fontSize = font.pointSize;CGFontRelease(fontRef); 
    }
    @end

If you run the sample code, you’ll notice that the text isn’t pixelated even though we aren’t setting the contentsScale anywhere. Another benefit of implementing CATextLayer as a backing layer is that its contentsScale is automatically set by the view.

In this simple example, we’ve only implemented a few of the styling and layout properties of UILabel, but with a bit more work we could create a LayerLabel class that supports the full functionality of UILabel and more (you will find several such classes already available as open source projects online).
If you only intend to support iOS 6 and above, a CATextLayer-based label may be of limited use. But in general, using +layerClass to create views backed by different layer types is a clean and reusable way to utilize CALayer subclasses in your apps.

<br />

##CATransformLayer

When constructing complex objects in 3D, it is convenient to be able to organize the individual elements hierarchically. For example, suppose you were making an arm: You would want the hand to be a child of the wrist, which would be a child of the forearm, which would be a child of the elbow, which would be a child of the upper arm, which would be a child of the shoulder, and so on.

The reason for this is that it allows you to move each section independently. Pivoting the elbow would move the lower arm and hand but not the shoulder. Core Animation layers easily allow for this kind of hierarchical arrangement in 2D, but in 3D it’s not possible because each layer flattens its children into a single plane (as explained in Chapter 5, “Transforms”).

CATransformLayer solves this problem. A CATransformLayer is unlike a regular CALayer in that it cannot display any content of its own; it exists only to host a transform that can be applied to its sublayers. CATransformLayer does not flatten its sublayers, so it can be used to construct a hierarchical 3D structure, such as our arm example.

Creating an arm programmatically would require rather a lot of code, so we’ll demonstrate this with something a bit simpler: In the cube example in Chapter 5, we worked around the layer-flattening problem by rotating the camera instead of the cube by using the sublayerTransform of the containing layer. This is a neat trick, but only works for a single object. If our scene contained two cubes, we would not be able to rotate them independently using this technique.

So, let’s try it using a CATransformLayer instead. The first problem to address is that we constructed our cube in Chapter 5 using views rather than standalone layers. We cannot place a view-backing layer inside another layer that is not itself a view-backing layer without messing up the view hierarchy. We could create a new UIView subclass backed by a CATransformLayer (using the +layerClass method), but to keep things simple for our example, let’s just re-create the cube using standalone layers instead of views. This means we can’t display buttons and labels on our cube faces like we did in Chapter 5, but we don’t need to do that right now.

Listing 6.5 contains the code. We position each cube face using the same basic logic we used in Chapter 5. But instead of adding the cube faces directly to the container view’s backing layer as we did before, we place them inside a CATransformLayer to create a standalone cube object, and then place two such cubes into our container. We’ve colored the cube faces randomly so as to make it easier to distinguish them without labels or lighting. Figure 6.5 shows the result.

####Listing 6.5 Assembling a 3D Layer Hierarchy Using CATransformLayer 

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIView *containerView; @end
    @implementation ViewController

    - (CALayer *)faceWithTransform:(CATransform3D)transform {
        //create cube face layer
        CALayer *face = [CALayer layer];
        face.frame = CGRectMake(-50, -50, 100, 100);
        //apply a random color
        CGFloat red = (rand() / (double)INT_MAX); 
        CGFloat green = (rand() / (double)INT_MAX); 
        CGFloat blue = (rand() / (double)INT_MAX); 
        face.backgroundColor = [UIColor colorWithRed:red
                                               green:green blue:blue
                                               alpha:1.0].CGColor;
        //apply the transform and return
        face.transform = transform;
        return face;
    }
    - (CALayer *)cubeWithTransform:(CATransform3D)transform {
        //create cube layer
        CATransformLayer *cube = [CATransformLayer layer];
        //add cube face 1
        CATransform3D ct = CATransform3DMakeTranslation(0, 0, 50); 
        [cube addSublayer:[self faceWithTransform:ct]];
        //add cube face 2
        ct = CATransform3DMakeTranslation(50, 0, 0); 
        ct = CATransform3DRotate(ct, M_PI_2, 0, 1, 0); 
        [cube addSublayer:[self faceWithTransform:ct]];
        //add cube face 3
        ct = CATransform3DMakeTranslation(0, -50, 0); 
        ct = CATransform3DRotate(ct, M_PI_2, 1, 0, 0); 
        [cube addSublayer:[self faceWithTransform:ct]];
        //add cube face 4
        ct = CATransform3DMakeTranslation(0, 50, 0); 
        ct = CATransform3DRotate(ct, -M_PI_2, 1, 0, 0); 
        [cube addSublayer:[self faceWithTransform:ct]];
        //add cube face 5
        ct = CATransform3DMakeTranslation(-50, 0, 0); 
        ct = CATransform3DRotate(ct, -M_PI_2, 0, 1, 0); 
        [cube addSublayer:[self faceWithTransform:ct]];
        //add cube face 6
        ct = CATransform3DMakeTranslation(0, 0, -50); 
        ct = CATransform3DRotate(ct, M_PI, 0, 1, 0); 
        [cube addSublayer:[self faceWithTransform:ct]];
        //center the cube layer within the container
        CGSize containerSize = self.containerView.bounds.size; 
        cube.position = CGPointMake(containerSize.width / 2.0,
        containerSize.height / 2.0);
        //apply the transform and return
        cube.transform = transform;
        return cube; 
    }
    - (void)viewDidLoad {
        [super viewDidLoad];
        //set up the perspective transform
        CATransform3D pt = CATransform3DIdentity; 
        pt.m34 = -1.0 / 500.0; 
        self.containerView.layer.sublayerTransform = pt;
        //set up the transform for cube 1 and add it
        CATransform3D c1t = CATransform3DIdentity; 
        c1t = CATransform3DTranslate(c1t, -100, 0, 0); 
        CALayer *cube1 = [self cubeWithTransform:c1t]; 
        [self.containerView.layer addSublayer:cube1];
        //set up the transform for cube 2 and add it
        CATransform3D c2t = CATransform3DIdentity;
        c2t = CATransform3DTranslate(c2t, 100, 0, 0); 
        c2t = CATransform3DRotate(c2t, -M_PI_4, 1, 0, 0); 
        c2t = CATransform3DRotate(c2t, -M_PI_4, 0, 1, 0); 
        CALayer *cube2 = [self cubeWithTransform:c2t]; 
        [self.containerView.layer addSublayer:cube2];
    }
    @end

![](http://ww1.sinaimg.cn/mw690/6cdd1b93gw1esgmt03l10j20gi084aal.jpg)

Figure 6.5 Two cubes with shared perspective but different transforms applied

##CAGradientLayer

CAGradientLayer is used to generate a smooth gradient between two or more colors. It’s possible to replicate the appearance of a CAGradientLayer using Core Graphics to draw into an ordinary layer’s backing image, but the advantage of using a CAGradientLayer instead is that the drawing is hardware accelerated.

###Basic Gradients

We’ll start with a simple diagonal gradient from red to blue (see Listing 6.6). The gradient colors are specified using the colors property, which is an array. The colors array expects values of type CGColorRef (which is not an NSObject derivative), so we need to use the bridging trick that we first saw in Chapter 2 to keep the compiler happy.

CAGradientLayer also has startPoint and endPoint properties that define the direction of the gradient. These are specified in unit coordinates, not points, so the top-left corner of the layer is specified with {0, 0} and the bottom-right corner is {1, 1}. The resulting gradient is shown in Figure 6.6.

####Listing 6.6 A Simple Two-Color Diagonal Gradient 

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIView *containerView; 
    @end

    @implementation ViewController
    - (void)viewDidLoad {
        [super viewDidLoad];
        //create gradient layer and add it to our container view
        CAGradientLayer *gradientLayer = [CAGradientLayer layer]; 
        gradientLayer.frame = self.containerView.bounds; 
        [self.containerView.layer addSublayer:gradientLayer];
        //set gradient colors
        gradientLayer.colors = @[(__bridge id)[UIColor redColor].CGColor, 
                                (__bridge id)[UIColor blueColor].CGColor];
        //set gradient start and end points
        gradientLayer.startPoint = CGPointMake(0, 0); 
        gradientLayer.endPoint = CGPointMake(1, 1);
    }
    @end

![](http://ww1.sinaimg.cn/mw690/6cdd1b93gw1esgmvf86dxj20gs08edgc.jpg)

Figure 6.6 A two-color diagonal gradient using CAGradientLayer

###Multipart Gradients

The colors array can contain as many colors as you like, so it is simple to create a multipart gradient such as a rainbow. By default, the colors in the gradient will be evenly spaced, but we can adjust the spacing using the locations property.

The locations property is an array of floating-point values (boxed as NSNumber objects). These values define the positions for each distinct color in the colors array, and are specified in unit coordinates, with 0.0 representing the start of the gradient and 1.0 representing the end.

It is not obligatory to supply a locations array, but if you do, you must ensure that the number of locations matches the number of colors or you’ll get a blank gradient.

Listing 6.7 shows a modified version of the diagonal gradient code from Listing 6.6. We now have a three-part gradient from red to yellow to green. A locations array has been specified with the values 0.0, 0.25, and 0.5, which causes the gradient to be squashed up toward the top-left corner of the view (see Figure 6.7).

####Listing 6.7 Using the locations Array to Offset a Gradient

    - (void)viewDidLoad {
        [super viewDidLoad];
        //create gradient layer and add it to our container view
        CAGradientLayer *gradientLayer = [CAGradientLayer layer]; 
        gradientLayer.frame = self.containerView.bounds; 
        [self.containerView.layer addSublayer:gradientLayer];
        //set gradient colors
        gradientLayer.colors = @[(__bridge id)[UIColor redColor].CGColor, (__bridge id)[UIColor yellowColor].CGColor, (__bridge id)[UIColor greenColor].CGColor];
        //set locations
        gradientLayer.locations = @[@0.0, @0.25, @0.5];
        //set gradient start and end points
        gradientLayer.startPoint = CGPointMake(0, 0);
        gradientLayer.endPoint = CGPointMake(1, 1); 
    }

![](http://ww1.sinaimg.cn/mw690/6cdd1b93gw1esgmy2d9ylj20h208eaal.jpg)

Figure 6.7 A three-color gradient, offset to the top left using the locations array

<br />

##CAReplicatorLayer

The CAReplicatorLayer class is designed to efficiently generate collections of similar layers. It works by drawing one or more duplicate copies of each of its sublayers, applying a different transform to each duplicate. This is probably easier to demonstrate than to explain, so let’s construct an example.

###Repeating Layers

In Listing 6.8, we create a small white square layer in the middle of the screen, then turn it into a ring of ten layers using CAReplicatorLayer. The instanceCount property specifies how many times the layer should be repeated. The instanceTransform applies a CATransform3D (in this case, a translation and rotation that moves the layer to the next point in the circle).

The transform is applied incrementally, with each instance positioned relative to the previous one. This is why the duplicates don’t all end up in the same place. Figure 6.8 shows the result.

Listing 6.8 Repeating Layers Using CAReplicatorLayer 

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIView *containerView; 
    @end

    @implementation ViewController
    - (void)viewDidLoad {
        [super viewDidLoad];
        //create a replicator layer and add it to our view
        CAReplicatorLayer *replicator = [CAReplicatorLayer layer]; 
        replicator.frame = self.containerView.bounds; 
        [self.containerView.layer addSublayer:replicator];
        //configure the replicator
        replicator.instanceCount = 10;
        //apply a transform for each instance
        CATransform3D transform = CATransform3DIdentity;
        transform = CATransform3DTranslate(transform, 0, 200, 0); 
        transform = CATransform3DRotate(transform, M_PI / 5.0, 0, 0, 1);
        transform = CATransform3DTranslate(transform, 0, -200, 0); 
        replicator.instanceTransform = transform;
        //apply a color shift for each instance
        replicator.instanceBlueOffset = -0.1; 
        replicator.instanceGreenOffset = -0.1;
        //create a sublayer and place it inside the replicator
        CALayer *layer = [CALayer layer];
        layer.frame = CGRectMake(100.0f, 100.0f, 100.0f, 100.0f); 
        layer.backgroundColor = [UIColor whiteColor].CGColor; 
        [replicator addSublayer:layer];
    }
    @end

![](http://ww4.sinaimg.cn/mw690/6cdd1b93gw1esgmzvklp2j20gr086gm6.jpg)

Figure 6.8 A ring of layers, created using CAReplicatorLayer

Note how the color of the layers changes as they are repeated: This was achieved using the instanceBlueOffset and instanceGreenOffset properties. By decreasing the intensity of the blue and green color components for each repetition, we’ve caused the layers to color-shift toward red.

This replication effect may look cool, but what are the practical applications? CAReplicatorLayer is useful for special effects such as drawing a contrail behind a missile in a game, or a particle explosion (although iOS 5 introduced CAEmitterLayer,which is more suited to creating arbitrary particle effects). There is another more practical use however: reflections.

###Reflections

By using CAReplicatorLayer to apply a transform with a negative scale factor to a single duplicate layer, you can create a mirror image of the contents of a given view (or an entire view hierarchy), creating a real-time “reflection” effect.

Let’s try implementing this idea in the form of a reusable UIView subclass called ReflectionView that will automatically generate a reflection of its contents. The code to create this is very simple (see Listing 6.9), and actually using the ReflectionView is even simpler; we can just drop an instance of our ReflectionView into Interface Builder (see Figure 6.9), and it will generate a reflection of its subviews at runtime without the need to add any setup code to the view controller (see Figure 6.10).

####Listing 6.9 Automatically Drawing a Reflection with CAReplicatorLayer 

    #import "ReflectionView.h"
    #import <QuartzCore/QuartzCore.h> 

    @implementation ReflectionView

    + (Class)layerClass {
        return [CAReplicatorLayer class]; 
    }

    - (void)setUp {
        //configure replicator
        CAReplicatorLayer *layer = (CAReplicatorLayer *)self.layer; 
        layer.instanceCount = 2;
        //move reflection instance below original and flip vertically
        CATransform3D transform = CATransform3DIdentity;
        CGFloat verticalOffset = self.bounds.size.height + 2;
        transform = CATransform3DTranslate(transform, 0, verticalOffset, 0); 
        transform = CATransform3DScale(transform, 1, -1, 0); 
        layer.instanceTransform = transform;
        //reduce alpha of reflection layer
        layer.instanceAlphaOffset = -0.6; 
    }
    - (id)initWithFrame:(CGRect)frame {
        //this is called when view is created in code
        if ((self = [super initWithFrame:frame])) {
            [self setUp]; 
        }
        return self; 
    }
    - (void)awakeFromNib {
        //this is called when view is created from a nib
        [self setUp]; 
    }
    @end

![](http://ww2.sinaimg.cn/mw690/6cdd1b93gw1esgn2nl0xpj20g209yjsi.jpg)

Figure 6.9 Using a ReflectionView in Interface Builder

![](http://ww2.sinaimg.cn/mw690/6cdd1b93gw1esgn38wbxlj20gh08j0t8.jpg)

Figure6.10 TheReflectionViewautomaticallycreatesareflectionatruntime

A more complete, open source implementation of the ReflectionView class, complete with an adjustable gradient fade effect (implemented using CAGradientLayer and layer masking), can be found at https://github.com/nicklockwood/ReflectionView.

<br />

##CAScrollLayer

For an untransformed layer, the size of a layer’s bounds will match the size of its frame. The frame is calculated automatically from the bounds, so changing either property will update the other.

But what if you want to display only a small part of a larger layer? For example, you might have a large image that you want the user to be able to scroll around, or a long list of data or text. In a typical iOS application, you might use a UITableView or UIScrollView, for this, but what’s the equivalent when working with standalone layers?

In Chapter 2, we explored the use of the layer contentsRect property, which is a good solution for displaying only a small part of a larger image in a layer. It’s not a very good solution if your layer contains sublayers, though, as you would need to manually recalculate and update all the sublayer positions every time you wanted to “scroll” the visible area.

That’s where CAScrollLayer comes in. CAScrollLayer has a -scrollToPoint: method that automatically adjusts the origin of the bounds so that the layer contents appear to scroll. Note, however, that that is all it does. As discussed earlier, Core Animation does not handle user input, so CAScrollLayer is not responsible for turning touch events into a scrolling action, nor does it render scrollbars orimplement any other iOS-specific behavior such as scroll bouncing (when a view elastically snaps back into place after scrolling past its bounds).

Let’s use a CAScrollLayer to create a very basic UIScrollView replacement. We’ll create a custom UIView that uses a CAScrollLayer as its backing layer, and then use UIPanGestureRecognizer to implement the touch handling. The code is shown in Listing 6.10. Figure 6.11 shows the ScrollView being used to pan around a UIImageView that is larger than the ScrollView frame.

Listing 6.10 Implementing a Scrolling View Using CAScrollLayer 

    #import "ScrollView.h"
    #import <QuartzCore/QuartzCore.h> 

    @implementation ScrollView
    + (Class)layerClass {
        return [CAScrollLayer class]; 
    }
    - (void)setUp {
        //enable clipping
        self.layer.masksToBounds = YES;
        //attach pan gesture recognizer
        UIPanGestureRecognizer *recognizer = nil; 
        recognizer = [[UIPanGestureRecognizer alloc]
        initWithTarget:self action:@selector(pan:)]; 
        [self addGestureRecognizer:recognizer];
    }
    - (id)initWithFrame:(CGRect)frame {
        //this is called when view is created in code
        if ((self = [super initWithFrame:frame])) {
            [self setUp]; 
        }
        return self; 
    }
    - (void)awakeFromNib {
        //this is called when view is created from a nib
        [self setUp]; 
    }
    - (void)pan:(UIPanGestureRecognizer *)recognizer {
        //get the offset by subtracting the pan gesture 
        //translation from the current bounds origin 
        CGPoint offset = self.bounds.origin;
        offset.x -= [recognizer translationInView:self].x; 
        offset.y -= [recognizer translationInView:self].y;
        //scroll the layer
        [(CAScrollLayer *)self.layer scrollToPoint:offset];
        //reset the pan gesture translation
        [recognizer setTranslation:CGPointZero inView:self]; 
    }
    @end

![](http://ww1.sinaimg.cn/mw690/6cdd1b93gw1esgn5uigdvj20gw086q3g.jpg)

Figure6.11 UsingCAScrollLayertocreateamakeshiftscrollview

Unlike UIScrollView, our custom ScrollView class doesn’t implement any sort of bounds checking. It’s quite possible to scroll the layer contents right off the edge of the view and keep going indefinitely. CAScrollLayer has no equivalent to theUIScrollView contentSize property, and so has no concept of the total scrollable area—all that is really happening when you scroll a CAScrollLayer is that it adjusts its bounds origin to the value you specify. It doesn’t adjust the bounds size at all because it doesn’t need to; contents can overflow the bounds without consequence.

The astute among you may wonder then what the point of using a CAScrollLayer is at all, as you could simply use a regular CALayer and adjust the bounds origin yourself. The truth is that there isn’t much point really. UIScrollView doesn’t use a CAScrollLayer, in fact, and simply implements scrolling by manipulating the layer bounds directly.

CAScrollLayer does have one potentially useful feature, though. If you look in the CAScrollLayer header file, you will notice that it includes a category that extends CALayer with a couple of extra methods and a property:

    - (void)scrollPoint:(CGPoint)p;
    - (void)scrollRectToVisible:(CGRect)r; 
    @property(readonly) CGRect visibleRect;

You might assume from their names that these methods add scrolling functionality to every CALayer instance, but in fact they are just utility methods to be used with layers that are placed inside of a CAScrollLayer. The scrollPoint: method searches up through the layer tree to find the first available CAScrollLayer and then scrolls it so that the specified point is visible. The scrollRectToVisible: method does the same for a rect. The visibleRect property determines the portion of the layer (if any) that is currently visible within the containing CAScrollLayer.

It would be relatively straightforward to implement these methods yourself, but CAScrollLayer saves you the trouble, and so (just about) justifies its existence when it comes to implementing layer scrolling.

<br />

##CATiledLayer

Sometimes you will find that you need to draw a really huge image. A typical example might be a photograph taken with a high-megapixel camera or a detailed map of the surface of the Earth. iOS applications usually run on quite memory-constrained devices, so loading an entire such image into memory is generally a bad idea. Loading large images may also be quite slow, and doing so in the conventional way (by calling the UIImage -imageNamed: or -imageWithContentsOfFile: methods on the main thread) is likely to freeze your interface for a while, or at least cause animations to stutter.

There is also an upper limit on the size of image that can be efficiently drawn on iOS. All images displayed onscreen have to eventually be converted to an OpenGL texture, and OpenGL has a maximum texture size (usually 2048×2048 or 4096×4096, depending on the device model). If you try to display an image that is larger than can fit into a single texture, even if the image is already resident in RAM, you can expect to see some very poorperformance indeed, as Core Animation is forced to process the image using the CPU instead of the much faster GPU. (See Chapter 12, “Tuning for Speed,” and Chapter 13, “Efficient Drawing,” for a more detailed explanation of software versus hardware drawing.)
CATiledLayer offers a solution for loading large images that solves the performance problems with large images by splitting the image up into multiple small tiles and loading them individually as needed. Let’s test this out with an example.

###Tile Cutting

We’ll start with a fairly large image—in this case, a 2048×2048 image of a snowman. To get any benefit from CATiledLayer, we’ll need to slice this into several smaller images. You can do this programmatically, but if you load the entire image and then cut it up at runtime, you will lose most of the loading performance advantage that CATiledLayer is designed to provide. Ideally, you want to do this as a preprocessing step instead.

Listing 6.11 shows the code for a simple Mac OS command-line app that can cut an image into tiles and save them as individual files for use with a CATiledLayer.

####Listing 6.11 A Terminal App for Slicing an Image into Tiles 

    #import <AppKit/AppKit.h>

    int main(int argc, const char * argv[]) {
        @autoreleasepool
        {
            //handle incorrect arguments
            if (argc < 2) {
                NSLog(@"TileCutter arguments: inputfile");
                return 0; 
            }

            //input file
            NSString *inputFile = [NSString stringWithCString:argv[1] encoding:NSUTF8StringEncoding];
            //tile size
            CGFloat tileSize = 256; //output path
            NSString *outputPath = [inputFile stringByDeletingPathExtension];
            //load image
            NSImage *image = [[NSImage alloc] initWithContentsOfFile:inputFile];
            NSSize size = [image size];
            NSArray *representations = [image representations]; 
            if ([representations count])
            {
                NSBitmapImageRep *representation = representations[0]; 
                size.width = [representation pixelsWide];
                size.height = [representation pixelsHigh];
            }
            NSRect rect = NSMakeRect(0.0, 0.0, size.width, size.height); 
            CGImageRef imageRef = [image CGImageForProposedRect:&rect
                                    context:NULL hints:nil];
            //calculate rows and columns
            NSInteger rows = ceil(size.height / tileSize); 
            NSInteger cols = ceil(size.width / tileSize);
            //generate tiles
            for (int y = 0; y < rows; ++y) {
                for (int x = 0; x < cols; ++x) {
                    //extract tile image
                    CGRect tileRect = CGRectMake(x*tileSize, y*tileSize, tileSize, tileSize);
                    CGImageRef tileImage = CGImageCreateWithImageInRect(imageRef, tileRect);
                    //convert to jpeg data
                    NSBitmapImageRep *imageRep = [[NSBitmapImageRep alloc] initWithCGImage:tileImage];
                    NSData *data = [imageRep representationUsingType: NSJPEGFileType properties:nil];
                    CGImageRelease(tileImage);
                    //save file
                    NSString *path = [outputPath stringByAppendingFormat: @"_%02i_%02i.jpg", x, y];
                    [data writeToFile:path atomically:NO]; 
                }
            } 
        }
        return 0; 
    }

We will use this to convert our 2048×2048 snowman image into 64 individual 256×256 tiles. (256×256 is the default tile size for CATiledLayer, although this can be changed using the tileSize property.) Our app expects to receive the path to the input image file as the first command-line parameter. We could hard-code this path argument in the build scheme so that we can run it from within Xcode, but that’s not very useful if we want to use a different image in future. Instead, we’ll build the app and save it somewhere sensible, and then invoke it from the Terminal, like this:

    > path/to/TileCutterApp path/to/Snowman.jpg

The app is very basic, but could easily be extended to support additional arguments such as tile size, or to export images in formats other than JPEG. The result of running it is a sequence of 64 new images, named as follows:

    Snowman_00_00.jpg 
    Snowman_00_01.jpg
    Snowman_00_02.jpg 
    ... 
    Snowman_07_07.jpg

Now that we have the tile images, we need to set up an iOS application to consume them. CATiledLayer integrates nicely with UIScrollView, so for the purposes of this example, we’ll place the CATiledLayer inside a UIScrollView. Other than setting the layer and scrollview bounds to match our total image size, all we really need to do is implement the -drawLayer:inContext: method, which is called by CATiledLayer whenever it needs to load a new tile image.

Listing 6.12 shows the code, and Figure 6.12 shows the result.

Listing 6.12 ASimpleScrollingCATiledLayerImplementation 

    #import "ViewController.h"
    #import <QuartzCore/QuartzCore.h>

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIScrollView *scrollView; 
    @end

    @implementation ViewController
    - (void)viewDidLoad {
        [super viewDidLoad];
        //add the tiled layer
        CATiledLayer *tileLayer = [CATiledLayer layer];
        tileLayer.frame = CGRectMake(0, 0, 2048, 2048); 
        tileLayer.delegate = self; 
        [self.scrollView.layer addSublayer:tileLayer];
        //configure the scroll view
        self.scrollView.contentSize = tileLayer.frame.size;
        //draw layer
        [tileLayer setNeedsDisplay]; 
    }
    - (void)drawLayer:(CATiledLayer *)layer inContext:(CGContextRef)ctx {
        //determine tile coordinate
        CGRect bounds = CGContextGetClipBoundingBox(ctx);
        NSInteger x = floor(bounds.origin.x / layer.tileSize.width); 
        NSInteger y = floor(bounds.origin.y / layer.tileSize.height);
        //load tile image
        NSString *imageName = [NSString stringWithFormat: @"Snowman_%02i_%02i, x, y];
        NSString *imagePath = [[NSBundle mainBundle] pathForResource:imageName ofType:@"jpg"];
        UIImage *tileImage = [UIImage imageWithContentsOfFile:imagePath];
        //draw tile
        UIGraphicsPushContext(ctx); 
        [tileImage drawInRect:bounds]; 
        UIGraphicsPopContext();
    }
    @end

![](http://ww3.sinaimg.cn/mw690/6cdd1b93gw1esgnipsb3hj20gt08b74v.jpg)

Figure6.12 ScrollingaCATiledLayerusingUIScrollView

As you scroll around the image, you will notice the tiles fading in as CATiledLayer loads them. This is the default behavior of CATiledLayer. (You might have seen this effect before in the [pre-iOS 6] Apple Maps app.) You can vary the fade duration or disable it altogether using the fadeDuration property.

CATiledLayer (unlike most UIKit and Core Animation methods) supports multi- threaded drawing. The -drawLayer:inContext: method may be called concurrently on multiple threads at the same time, so be careful to ensure that any drawing code you implement within that method is thread-safe.

###Retina Tiles

You might also have noticed that the tile images are not being displayed at Retina resolution. To render the CATiledLayer at the device’s native resolution, we need to set the layer contentsScale to match the UIScreen scale, as follows:

    tileLayer.contentsScale = [UIScreen mainScreen].scale;

Interestingly, the tileSize is specified in pixels, not points, so by increasing the contentsScale, we automatically halve the default tile size (it now equates to 128×128 points onscreen instead of 256×256). Therefore, we don’t need to update the tile size manually or supply a separate set of tiles at Retina resolution. We will need to adjust the tile rendering code slightly to accommodate the change in scale, however:

    //determine tile coordinate
    CGRect bounds = CGContextGetClipBoundingBox(ctx); 
    CGFloat scale = [UIScreen mainScreen].scale;
    NSInteger x = floor(bounds.origin.x / layer.tileSize.width * scale); 
    NSInteger y = floor(bounds.origin.y / layer.tileSize.height * scale);

Correcting the scale in this way also means that our snowman image will be rendered at half the size on Retina devices (at a total size of only 1024×1024 points instead of 2048×2048 as before). This usually doesn’t matter for the types of images normally displayed with CATiledLayer (such photographs and maps, which are designed to be zoomed and viewed at various scales), but it’s worth bearing in mind.

<br />

##CAEmitterLayer

In iOS 5, Apple introduced a new CALayer subclass called CAEmitterLayer. CAEmitterLayer is a high-performance particle engine designed to let you create real- time particle animations such as smoke, fire, rain, and so on.

CAEmitterLayer acts as a container for a collection of CAEmitterCell instances that define a particle effect. You will create one or more CAEmitterCell objects as templates for the different particle types, and the CAEmitterLayer is responsible for instantiating a stream of particles based on these templates.

A CAEmitterCell is similar to a CALayer: It has a contents property that can be set using a CGImage, as well as dozens of configurable properties that control the appearance and behavior of the particles. We won’t attempt to describe each and every property in detail, but they are well documented in the CAEmitterCell class header file.

Let’s try an example: We’ll create a fiery explosion effect by emitting particles in a circle with varying velocity and alpha. Listing 6.13 contains the code for generating the explosion. You can see the result in Figure 6.13.

####Listing 6.13 Creating an Explosion Effect with CAEmitterLayer 

    #import "ViewController.h"
    #import <QuartzCore/QuartzCore.h>

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIView *containerView; 
    @end

    @implementation ViewController
    - (void)viewDidLoad {
        [super viewDidLoad];
        //create particle emitter layer
        CAEmitterLayer *emitter = [CAEmitterLayer layer]; 
        emitter.frame = self.containerView.bounds; 
        [self.containerView.layer addSublayer:emitter];
        //configure emitter
        emitter.renderMode = kCAEmitterLayerAdditive; 
        emitter.emitterPosition = CGPointMake(emitter.frame.size.width / 2.0,
        emitter.frame.size.height / 2.0);
        //create a particle template
        CAEmitterCell *cell = [[CAEmitterCell alloc] init];
        cell.contents = (__bridge id)[UIImage imageNamed:@"Spark.png"].CGImage; 
        cell.birthRate = 150;
        cell.lifetime = 5.0;
        cell.color = [UIColor colorWithRed:1
                            green:0.5 blue:0.1
                            alpha:1.0].CGColor;
        cell.alphaSpeed = -0.4; cell.velocity = 50; 
        cell.velocityRange = 50; cell.emissionRange = M_PI * 2.0;
        //add particle template to emitter
        emitter.emitterCells = @[cell]; 
    }
    @end

![](http://ww2.sinaimg.cn/mw690/6cdd1b93gw1esgnm2yr78j20h208k752.jpg)

Figure 6.13 A fiery explosion effect

The properties of CAEmitterCell generally break down into three categories:

* A starting value for a particular attribute of the particle. For example, the color property specifies a blend color that will be multiplied by the colors in the contents image. In our explosion project, we’ve set this to an orangey color.
* A range by which a value will vary from particle to particle. For example, the emissionRange property is set to 2π in our project, indicating that particles can be emitted in any direction within a 360-degree radius. By specifying a smaller value, we could create a conical funnel for our particles.
* A change over time for a particular value. For example, in the explosion project, we set alphaSpeed to -0.4, meaning that the alpha value of the particle will reduce by 0.4 every second, creating a fadeout effect for the particles as they travel away from the emitter.

The properties of the CAEmitterLayer itself control the position and general shape of the entire particle system. Some properties such as birthRate, lifetime, and velocity duplicate values that are specified on the CAEmitterCell. These act as multipliers so that you can speed up or amplify the entire particle system using a single value. Other notable properties include the following:

* preservesDepth, which controls whether a 3D particle system is flattened into a single layer (the default) or can intermingle with other layers in the 3D space of its container layer.
* renderMode, which controls how the particle images are blended visually. You may have noted in our example that we set this to kCAEmitterLayerAdditive, which has the effect of combining the brightness of overlapping particles so that they appear to glow. If we were to leave this as the default value of kCAEmitterLayerUnordered, the result would be a lot less pleasing (see Figure 6.14).

<br />

##CAEAGLLayer

When it comes to high-performance graphics on iOS, the last word is OpenGL. It should probably also be the last resort, at least for nongaming applications, because it’s phenomenally complicated to use compared to the Core Animation and UIKit frameworks.

OpenGL provides the underpinning for Core Animation. It is a low-level C API that communicates directly with the graphics hardware on the iPhone and iPad, with minimal abstraction. OpenGL has no notion of a hierarchy of objects or layers; it simply deals with triangles. In OpenGL everything is made of triangles that are positioned in 3D space and have colors and textures associated with them. This approach is extremely flexible and powerful, but it’s a lot of work to replicate something like the iOS user interface from scratch using OpenGL.

To get good performance with Core Animation, you need to determine what sort of content you are drawing (vector shapes, bitmaps, particles, text, and so on) and then select an appropriate layer type to represent that content. Only some types of content have beenoptimized in Core Animation; so if the thing you want to draw isn’t a good match for any of the standard layer classes, you will struggle to achieve good performance.

Because OpenGL makes no assumptions about your content, it can be blazingly fast. With OpenGL, you can draw anything you like provided you know how to write the necessary geometry and shader logic. This makes it a popular choice for games (where Core Animation’s limited repertoire of optimized content types doesn’t always meet the requirements), but it’s generally overkill for applications with a conventional interface.

In iOS 5, Apple introduced a new framework called GLKit that takes away some of the complexity of setting up an OpenGL drawing context by providing a UIView subclass called GLKView that handles most of the setup and drawing for you. Prior to that it was necessary to do all the low-level configuration of the various OpenGL drawing buffers yourself using CAEAGLLayer, which is a CALayer subclass designed for displaying arbitrary OpenGL graphics.

It is rare that you will need to manually set up a CAEAGLLayer any more (as opposed to just using a GLKView), but let’s give it a go for old time’s sake. Specifically, we'll set up an OpenGL ES 2.0 context, which is the standard for all modern iOS devices.

Although it is possible to do this entirely without using GLKit, it involves a lot of additional work in the form of setting up vertex and fragment shaders, which are self- contained programs that are written in a C-like language called GLSL, and are loaded at runtime into the graphics hardware. 

Writing GLSL code is not really related to the task of setting up an EAGLLayer, so we’ll use the GLKBaseEffect class to abstract away the shader logic for us. Everything else, we’ll do the old-fashioned way.

Before we begin, you need to add the GLKit and OpenGLES frameworks to your project. You should then be able to implement the code in Listing 6.14, which does the bare minimum necessary to set up a GAEAGLLayer with an OpenGL ES 2.0 drawing context and render a colored triangle (see Figure 6.15).

####Listing 6.14 Drawing a Triangle Using CAEAGLLayer

    #import "ViewController.h" 
    #import <QuartzCore/QuartzCore.h> 
    #import <GLKit/GLKit.h>

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIView *glView; 
    @property (nonatomic, strong) EAGLContext *glContext; 
    @property (nonatomic, strong) CAEAGLLayer *glLayer; 
    @property (nonatomic, assign) GLuint framebuffer; 
    @property (nonatomic, assign) GLuint colorRenderbuffer; 
    @property (nonatomic, assign) GLint framebufferWidth; 
    @property (nonatomic, assign) GLint framebufferHeight; 
    @property (nonatomic, strong) GLKBaseEffect *effect;
    @end

    @implementation ViewController

    - (void)setUpBuffers {
        //set up frame buffer
        glGenFramebuffers(1, &_framebuffer); 
        glBindFramebuffer(GL_FRAMEBUFFER, _framebuffer);
        //set up color render buffer
        glGenRenderbuffers(1, &_colorRenderbuffer); 
        glBindRenderbuffer(GL_RENDERBUFFER, _colorRenderbuffer); 
        glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0,GL_RENDERBUFFER, _colorRenderbuffer); 
        [self.glContext renderbufferStorage:GL_RENDERBUFFER
                                fromDrawable:self.glLayer]; 
        glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_WIDTH,&_framebufferWidth); 
        glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_HEIGHT, &_framebufferHeight);
        //check success
        if (glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE) {
            NSLog(@"Failed to make complete framebuffer object: %i", glCheckFramebufferStatus(GL_FRAMEBUFFER));
        } 
    }

    - (void)tearDownBuffers {
        if (_framebuffer) {
            //delete framebuffer
            glDeleteFramebuffers(1, &_framebuffer);
            _framebuffer = 0; 
        }
        if (_colorRenderbuffer) {
            //delete color render buffer
            glDeleteRenderbuffers(1, &_colorRenderbuffer);
            _colorRenderbuffer = 0; 
        }
    }

    - (void)drawFrame {
        //bind framebuffer & set viewport
        glBindFramebuffer(GL_FRAMEBUFFER, _framebuffer); 
        glViewport(0, 0, _framebufferWidth, _framebufferHeight);
        //bind shader program
        [self.effect prepareToDraw];
        //clear the screen
        glClear(GL_COLOR_BUFFER_BIT); glClearColor(0.0, 0.0, 0.0, 1.0);
        //set up vertices
        GLfloat vertices[] = {
            -0.5f, -0.5f, -1.0f, 0.0f, 0.5f, -1.0f, 0.5f, -0.5f, -1.0f,
        };
        //set up colors
        GLfloat colors[] = {
            0.0f, 0.0f, 1.0f, 1.0f, 0.0f, 1.0f, 0.0f, 1.0f, 1.0f, 0.0f, 0.0f, 1.0f,
        };
        //draw triangle
        glEnableVertexAttribArray(GLKVertexAttribPosition); 
        glEnableVertexAttribArray(GLKVertexAttribColor); 
        glVertexAttribPointer(GLKVertexAttribPosition, 3, GL_FLOAT, GL_FALSE, 0, vertices); 
        glVertexAttribPointer(GLKVertexAttribColor, 4, GL_FLOAT, GL_FALSE, 0, colors); 
        glDrawArrays(GL_TRIANGLES, 0, 3);
        //present render buffer
        glBindRenderbuffer(GL_RENDERBUFFER, _colorRenderbuffer);
        [self.glContext presentRenderbuffer:GL_RENDERBUFFER]; 
    }
    - (void)viewDidLoad
    {
        [super viewDidLoad];
        //set up context
        self.glContext = [[EAGLContext alloc] initWithAPI: kEAGLRenderingAPIOpenGLES2];
        [EAGLContext setCurrentContext:self.glContext];
        //set up layer
        self.glLayer = [CAEAGLLayer layer];
        self.glLayer.frame = self.glView.bounds;
        [self.glView.layer addSublayer:self.glLayer]; 
        self.glLayer.drawableProperties = @{kEAGLDrawablePropertyRetainedBacking:
        @NO, kEAGLDrawablePropertyColorFormat: kEAGLColorFormatRGBA8};
        //set up base effect
        self.effect = [[GLKBaseEffect alloc] init]; //set up buffers
        [self setUpBuffers];
        //draw frame
        [self drawFrame]; 
    }

    - (void)viewDidUnload {
        [self tearDownBuffers];
        [super viewDidUnload]; 
    }

    - (void)dealloc {
        [self tearDownBuffers];
        [EAGLContext setCurrentContext:nil]; 
    }
    @end

![](http://ww4.sinaimg.cn/mw690/6cdd1b93gw1esgqjg69utj20gz084t96.jpg)

Figure6.15 AsingletrianglerenderedinaCAEAGLLayerusingOpenGL

In a real OpenGL application, we would probably want to call the -drawFrame method 60 times per second using an NSTimer or CADisplayLink (see Chapter 11, “Timer- Based Animation”), and we would separate out geometry generation from drawing so that we aren’t regenerating our triangle’s vertices every frame (and also so that we can draw something other than a single triangle), but this should be enough to demonstrate the principle.

<br/>

##AVPlayerLayer

The last layer type we will look at is AVPlayerLayer. Although not part of the Core Animation framework (the A V prefix is a bit of a giveaway), AVPlayerLayer is an example of another framework (in this case, AVFoundation) tightly integrating with Core Animation by providing a CALayer subclass to display a custom content type.

AVPlayerLayer is used for playing video on iOS. It is the underlying implementation used by high-level APIs such as MPMoviePlayer, and provides lower-level control over the display of video. Usage of AVPlayerLayer is actually pretty straightforward: You can either create a layer with a video player already attached using the +playerLayerWithPlayer: method, or you can create the layer first and attach an AVPlayer instance using the player property.

Before we begin, we need to add the AVFoundation framework to our project, because it’s not included in the default project template. After you’ve done that, see Listing 6.15 for the code to create a simple movie player. Figure 6.16 shows the movie player in action.

####Listing 6.15 Playing a Video with AVPlayerLayer

    #import "ViewController.h"
    #import <QuartzCore/QuartzCore.h> 
    #import <AVFoundation/AVFoundation.h>

    @interface ViewController ()
    @property (nonatomic, weak) IBOutlet UIView *containerView; 
    @end

    @implementation ViewController
    - (void)viewDidLoad {
        [super viewDidLoad];
        //get video URL
        NSURL *URL = [[NSBundle mainBundle] URLForResource:@"Ship" withExtension:@"mp4"];
        //create player and player layer
        AVPlayer *player = [AVPlayer playerWithURL:URL];
        AVPlayerLayer *playerLayer = [AVPlayerLayer playerLayerWithPlayer:player];
        //set player layer frame and attach it to our view
        playerLayer.frame = self.containerView.bounds; 
        [self.containerView.layer addSublayer:playerLayer];
        //play the video
        [player play]; 
    }
    @end

![](http://ww1.sinaimg.cn/mw690/6cdd1b93gw1esgqkxqid9j20gs08kaaj.jpg)

Figure 6.16 Still-frame from a video playing inside an AVPlayerLayer

Although we’re creating the AVPlayerLayer programmatically, we are still adding it to a container view instead of directly inside the main view for the controller. This is so that we can use ordinary autolayout constraints to center the layer; otherwise, we would have to reposition it programmatically every time the device is rotated due to Core Animation not supporting autoresizing or autolayout (see Chapter 3, “Layer Geometry,” for details).

Of course, because AVPlayerLayer is a subclass of CALayer, it inherits all of its cool features. We’re not restricted to playing our video in a simple rectangle; with the few extra lines of code in Listing 6.16, we can rotate our video in 3D and add rounded corners, a colored border, mask, drop shadow, and so on (see Figure 6.17).

####Listing 6.16 Adding a Transform, Border, and Corner Radius to the Video

    - (void)viewDidLoad {
        ...
        //set player layer frame and attach it to our view
        playerLayer.frame = self.containerView.bounds; 
        [self.containerView.layer addSublayer:playerLayer];
        //transform layer
        CATransform3D transform = CATransform3DIdentity; 
        transform.m34 = -1.0 / 500.0;
        transform = CATransform3DRotate(transform, M_PI_4, 1, 1, 0); 
        playerLayer.transform = transform;
        //add rounded corners and border
        playerLayer.masksToBounds = YES; 
        playerLayer.cornerRadius = 20.0; 
        playerLayer.borderColor = [UIColor redColor].CGColor; 
        playerLayer.borderWidth = 5.0;
        //play the video
        [player play]; 
    }

![](http://ww3.sinaimg.cn/mw690/6cdd1b93gw1esgqmbiibkj20gm08emxp.jpg)

Figure 6.17 AVPlayerLayer rotated in 3D and displaying a border and corner radius

<br />

##Summary

This chapter provided an overview of the many specialized layer types and the effects that can be achieved by using them. We have only scratched the surface in many cases; classes such as CATiledLayer or CAEmitterLayer could fill a chapter on their own. However, the key point to remember is that CALayer is a jack-of-all-trades, and is not optimized for every possible drawing scenario. To get the best performance out of Core Animation, you need to choose the right tool for the job, and hopefully you have been inspired to dig deeper into the various CALayer subclasses and their capabilities.We touched on animation a little bit in this chapter with the CAEmitterLayer and AVPlayerLayer classes. In Part II, we dive into animation properly, starting with implicit animations.

