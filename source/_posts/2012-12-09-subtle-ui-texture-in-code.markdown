---
layout: post
title: "Subtle UI texture in code"
date: 2012-12-09 20:40
comments: false
categories: code
---

[Matt Gemmell has an interesting post](http://mattgemmell.com/2012/01/29/subtle-ui-texture-in-photoshop/)  on using Photoshop to create subtle noise textures for use on images and controls. With the advent of iOS 6, Apple added support for two key Core Image filter types, `CIColorMonochrome` and `CIRandomGenerator`, which allow noise textures to be generated using a few simple lines of code. Resizable images can also be configured to tile the central area instead of stretching it, meaning that noise textures can be used in them without artefacts.

In this post I will recreate the noise textured buttons from Gemmell's post, without the need for any external image software or image files. Any images for your project that can be generated in code should be, as they lead to smaller packages, simpler retheming and fewer files to manage. [I'm not alone in this opinion](http://blog.wilshipley.com/2005/07/pimp-my-code-part-3-gradient.html). 

<!--more-->

Creating an image in code _can_ take some time. The effect on your app's performance can be ameliorated by reducing the amount of times you do it. Images can be stored statically or cached using `NSCache` rather than be recreated every time or using an expensive `drawRect:` method. Generally, the best performance will be obtained by creating a `UIImage` object and treating it as one you'd loaded in from a file. 

Here, I am creating an image containing a noise texture. This texture is then drawn on top of a resizable button image. I've combined everything in one method for readability, but in a real use case you would probably have a category method on `UIImage` to return the noise texture, since you would probably use it on many of the images in your project. 

This code has external dependencies - you have to add the Core Image framework to your project, and I also make use of the [easy gradients code](http://commandshift.co.uk/blog/2012/11/25/easy-cggradients/) from an earlier post. Colour selection was made easy thanks to [Color Sense for Xcode](https://github.com/omz/ColorSense-for-Xcode). 

Here goes: 

``` obj-c

#import "UIColor+EasyGradients.h"
#import <CoreImage/CoreImage.h>

#define CORNER_RADIUS 10.0
#define HEIGHT 44.0
#define TILE_SIZE 50.0

-(UIImage*)buttonImage
{
    CGRect buttonRect = CGRectMake(0.0, 0.0, (CORNER_RADIUS * 2.0) + TILE_SIZE, HEIGHT);
    
    UIGraphicsBeginImageContextWithOptions(buttonRect.size, NO, 0);
    
    CGContextRef c = UIGraphicsGetCurrentContext();
    
    // Path used to define the button edge
    UIBezierPath *outline = [UIBezierPath bezierPathWithRoundedRect:buttonRect cornerRadius:CORNER_RADIUS];
    
    // Save the context as we are going to make modifications to it
    CGContextSaveGState(c);
    
    // Clip to the button area
    CGContextAddPath(c, outline.CGPath);
    CGContextClip(c);
    
    // Generate and draw the gradient
    CGGradientRef gradient = [[UIColor colorWithRed:0.997 green:0.866 blue:0.240 alpha:1.000] newGradientToColor:[UIColor colorWithRed:0.715 green:0.526 blue:0.148 alpha:1.000]];
    CGContextDrawLinearGradient(c, gradient, CGPointMake(0.0, 0.0), CGPointMake(0.0,HEIGHT), 0);
        [[UIColor colorWithWhite:0.360 alpha:1.000] set];
    CGGradientRelease(gradient);
    // Apply noise
    
    // Mono filter as the noise is coloured
    CIFilter *monoFilter = [CIFilter filterWithName:@"CIColorMonochrome"];
    // Noise filter
    CIFilter *noiseFilter = [CIFilter filterWithName:@"CIRandomGenerator"];
    // The mono filter takes the noise filter as input
    [monoFilter setValue:noiseFilter.outputImage forKey:kCIInputImageKey];
    // Create a core image image of the filter's output
    CIImage *noiseImage = [monoFilter outputImage];
    // Create a context to draw this in
    CIContext *noiseContext = [CIContext contextWithOptions:nil];
    
    // Make a core graphics image from the Core Image context. This is at pixel level, so we need to
    // take the scale into account.
    CGFloat scale = [UIScreen mainScreen].scale;
    CGRect noiseRect = CGRectApplyAffineTransform(buttonRect, CGAffineTransformMakeScale(scale, scale));
    CGImageRef noiseCGImage = [noiseContext createCGImage:noiseImage fromRect:noiseRect];
    
    // Draw the noise at 5% opacity
    CGContextSaveGState(c);
    CGContextSetAlpha(c, 0.05);
    CGContextDrawImage(c, buttonRect, noiseCGImage);
    CGImageRelease(noiseCGImage);
    CGContextRestoreGState(c);
    
    // Draw the button's border
    [[UIColor colorWithWhite:0.360 alpha:1.000] set];
    outline.lineWidth = 2.0;
    [outline stroke];
    CGContextRestoreGState(c);
    
    // Create a resizable image - need to tile the noise area otherwise we would get streaks. 
    UIImage *image = [UIGraphicsGetImageFromCurrentImageContext() resizableImageWithCapInsets:UIEdgeInsetsMake(0.0, CORNER_RADIUS, 0.0, CORNER_RADIUS) resizingMode:UIImageResizingModeTile];
    UIGraphicsEndImageContext();

    return image;
}
```

The height is fixed, because of the gradient, but the image can be tiled widthwise. 

Here are the results:

{% img /images/2012_12_textured_button.png %}

And zoomed in:

{% img /images/2012_12_textured_button_zoomed.png %}

