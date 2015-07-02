---
layout: post
title: "UIButton edge insets"
date: 2013-03-12 20:15
comments: false
categories: code
---

> Don't subclass or reimplement UIButton!

_(Paraphrased from some WWDC talk, I can't remember which one)_

This is good advice. Not only is UIButton a class cluster, so not really suitable for subclassing, it also does a lot for you, and with judicious use of the image view, background image view and (as of iOS6) attributed titles, you can create almost any effect you want.

I've recently been struggling with laying out a custom button featuring an image and a label, so if you've been baffled by `titleEdgeInsets`, `contentEdgeInsets` and `imageEdgeInsets`, read on!

<!--more-->

First we need to make our button. Obviously we'll ignore `UIButtonTypeRoundedRect`, which is presumably for debugging purposes since it is so ugly. 

Let's assume you have a nice background image for your button (perhaps drawn using clever code like [this](/blog/2012/12/09/subtle-ui-texture-in-code/)), and you're using `UIButtonTypeCustom`. As well as the background image, you can set another image and a title, which by default will be arranged side-by-side, with the image on the left. 

You might want to increase the space between the label and the image. You might want the image on the left. You might want the image in the centre, with the label underneath, and so on. You do all of this by editing three sets of `UIEdgeInsets` on the button. 

I'm not going to explain what they all are and how they work. I don't think it's a topic that is best understood by reading about it. You need to tweak and twiddle and see the effects straight away. To that end, I've created a [UIButton playground app](https://github.com/jrturton/ButtonInsets) which allows you to tweak the various sets of edge insets and see the results straight away. It doesn't feature a background image (feel free to add one yourself), since I didn't need one for the app I was working on. 

![Button insets screenshot](/images/buttonInsets.png)

It's about functionality, not looks! I hope this helps people struggling to understand edge insets, as I was. I now fire up this app every time I need to work out a button's internal layout. 

_As a slight diversion, I'd been reading a lot about [Reactive Cocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) and we'd been discussing at work about how or if we would find it useful. As a first investigation I wrote the buttons app twice, once using RAC and once with standard code. You may find the two approaches (both are contained in the project, you switch between the two using a `#define` statement in the prefix.pch file.)_ 
