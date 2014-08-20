---
layout: post
title: "Easy CGGradients"
date: 2012-11-25 21:30
comments: false
categories: code
---

Drawing your own view content can be much better than using images, particularly if you are dealing with universal applications. Once you take into account retina and non-retina, iPad and iPhone, 3.5 and 4 inch you can be managing a plethora of images, and if (when?) any changes are made to the design during development, that's a lot of files to regenerate and replace in your project.

Lots of common designs can be easily drawn in code, but the core graphics API for drawing gradients is a little bit impenetrable, particularly if you are used to Objective-C as opposed to C. 

<!--more-->

Here is a simple category method on `UIColor` to facilitate the creation of gradients. It is an instance method, returning a `CGGradient` from the receiver to the passed in `UIColor` object. The source is also available on [GitHub](https://github.com/jrturton/UIColor-EasyGradients).

**Header**

``` objc
@interface UIColor (EasyGradient)

-(CGGradientRef)newGradientToColor:(UIColor*)color;

@end

```
     
**Implementation**

``` objc

-(CGGradientRef)newGradientToColor:(UIColor*)color
{
    // Caller is responsible for releasing
    
    // Are we in the RGB colour space?
    BOOL selfOK,colorOK;
    
    CGFloat r1,r2,g1,g2,b1,b2,a1,a2;
    
    selfOK = [self getRed:&r1 green:&g1 blue:&b1 alpha:&a1];
    colorOK = [color getRed:&r2 green:&g2 blue:&b2 alpha:&a2];
    
    if (selfOK && colorOK)
    {
        CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
        size_t num_locations = 2;
        CGFloat locations[2] = {0.0,1.0};
        CGFloat components[8] = { r1,g1,b1,a1, r2,g2,b2,a2 };
        
        CGGradientRef gradient = CGGradientCreateWithColorComponents(colorSpace, components, locations, num_locations);
        CGColorSpaceRelease(colorSpace);
        return gradient;
    }
    
    // Try grayscale
    CGFloat w1,w2;
    selfOK = [self getWhite:&w1 alpha:&a1];
    colorOK = [color getWhite:&w2 alpha:&a2];
    if (selfOK && colorOK)
    {
        CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceGray();
        size_t num_locations = 2;
        CGFloat locations[2] = {0.0,1.0};
        CGFloat components[4] = { w1,a1, w2,a2 };
        
        CGGradientRef gradient = CGGradientCreateWithColorComponents(colorSpace, components, locations, num_locations);
        CGColorSpaceRelease(colorSpace);
        return gradient;
    }
    
    // otherwise, we can't do anything
    NSAssert(NO,@"Colors must be in RGB or Grayscale color space");
    return NULL;
}

```

Here's a snippet of it in action:

``` objc
- (void)drawRect:(CGRect)rect
{
    CGContextRef context = UIGraphicsGetCurrentContext();
    
    CGGradientRef monoGradient = [[UIColor colorWithWhite:0.0 alpha:1.0] newGradientToColor:[UIColor colorWithWhite:0.5 alpha:1.0]];
    
    CGContextAddEllipseInRect(context, CGRectInset(self.bounds,5.0,5.0));
    CGContextClip(context);
    CGContextDrawLinearGradient(context, monoGradient, CGPointMake(0.0, 0.0), CGPointMake(0.0, CGRectGetMaxY(self.bounds)), 0);
    
    CGGradientRelease(monoGradient);
    
    CGGradientRef colourGradient = [[UIColor redColor] newGradientToColor:[UIColor purpleColor]];
    
    CGContextAddEllipseInRect(context, CGRectInset(self.bounds,15.0,15.0));
    CGContextClip(context);
    CGContextDrawLinearGradient(context, colourGradient, CGPointMake(0.0, 0.0), CGPointMake(0.0, CGRectGetMaxY(self.bounds)), 0);
    CGGradientRelease(colourGradient);
}
```

Results:

{% img /images/2012_11_gradient_example.png %}

Beautiful!


