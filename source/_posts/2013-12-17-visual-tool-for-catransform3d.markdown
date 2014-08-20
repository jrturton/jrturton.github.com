---
layout: post
title: "Visual tool for CATransform3D"
date: 2013-12-17 20:40
comments: false
categories: 
---

Understanding [CGAffineTransforms](https://developer.apple.com/library/mac/documentation/graphicsimaging/reference/CGAffineTransform/Reference/reference.html) is tricky. Understanding them in three dimensions, as a [CATransform3D](https://developer.apple.com/library/Mac/DOCUMENTATION/Cocoa/Reference/CoreAnimation_functions/Reference/reference.html), is even trickier. 

There are some [fantastic](http://ronnqvi.st/translate-rotate-translate) [resources](http://ronnqvi.st/the-math-behind-transforms) [available](http://markpospesel.wordpress.com/2012/05/23/anatomy-of-a-page-flip-animation/), but I'm a visual learner and I like to be able to mess with things to see what happens. 

Though it is great to understand the maths behind transforms[^1], that doesn't necessarily allow you to get the effect you're after just by making up some matrix values and trying them out. We need a little transform sandbox to play in.

<!--more-->

To that end, I've created a demo app, [available on GitHub](https://github.com/jrturton/3DTransformFun) which allows you to create any number of effects by stacking up transforms, adjusting the anchor point and tweaking the individual values of each transform in the stack:

{%img /images/transformsDemoScreenshot.png 'Screenshot of the transforms demo app'%}

A static screenshot doesn't really do it justice - you need to build the app and animate it to see the view dancing around the anchor point. I find it quite hypnotic. It is very useful for working out the transformations you need to apply to get a particular effect, rather than having to build your actual app each time. 

You can add transforms of any type (rotation, translation, scale, or manual), edit the individual parameters, re-order them in the stack and animate the whole thing to and from the identity transform. 

There were a few challenges faced when building the app:

* Making the anchor point indicator occupy the right position in 3D - the secret is to [use a CATransformLayer to hold all of your visible layers](http://www.cocoanetics.com/2012/08/cubed-coreanimation-conundrum/)
* Adjusting the position of the grid layer when updating the anchor point - once you've stopped using it as the main view's layer, you can't do the [frame resetting trick](http://stackoverflow.com/questions/1968017/changing-my-calayers-anchorpoint-moves-the-view)
* Learning that UIView block animations don't apply to animations on sublayers ([here](https://developer.apple.com/library/ios/documentation/windowsviews/conceptual/viewpg_iphoneos/AnimatingViews/AnimatingViews.html), via [here](http://khanlou.com/2012/10/mixing-uiview-and-calayer-animations/))

A lot of help also came from [iOS Core Animation: Advanced Techniques](http://www.informit.com/store/ios-core-animation-advanced-techniques-9780133440751) by [Nick Lockwood](https://twitter.com/nicklockwood), highly recommended for anyone interested in Core Animation.

Pull requests with improvements are most welcome. 

[^1]: And despite the excellent articles linked above, I only really understand it myself when I'm reading them. A day or so later, it's gone...
