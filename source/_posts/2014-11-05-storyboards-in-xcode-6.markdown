---
layout: post
title: "Storyboards in Xcode 6"
date: 2014-11-05 19:33:19 +0000
comments: false
categories: 
---

I've liked the idea of storyboards since their introduction, and even used them a couple of times, but I've always gone back to laying out views in code, mostly using [my autolayout category](https://github.com/jrturton/UIView-Autolayout).

It's clear that Apple would *prefer* us to be using interface builder and storyboards, so every time a new version of Xcode comes out I give it a try to see if I'm ready to move on. Here are the results for Xcode 6, after I've spent the last few weeks building a universal app for iOS 7 and 8 using storyboards. 

<!--more-->

## The problem with storyboards

These are my most important reasons for not using storyboards up to and including Xcode 5.

1. **Splitting configuration of the layout and appearance over multiple files**: there was always a lot you couldn't implement in the storyboard, particularly for custom view classes, so I always ended up with a long `viewDidLoad` method to finish off all the parts that couldn't be done in interface builder. This then meant that there were multiple places to check when trying to investigate or change something. 
2. **Constraints**: I always use Autolayout. Constraint editing in interface builder started off terrible and has improved with each version of Xcode. 
3. **iPad and iPhone versions**: there was a *lot* of duplication if you were making a universal app using storyboards.  Even if your view controllers had the same elements with different layouts, that was still two files, two sets of outlets to connect, two sets of fonts and colours and sizes to set...
4. **git dirtying**: The maddening habit Xcode has of marking a storyboard file as changed just by opening it. 
5. **Randomly destructive editing**: OK, it's not *random*, but here I'm talking about the fact that, due to the nature of a GUI editor, you don't see all the changes that are made when you alter something. Delete a view? A bunch of constraints disappear. Change a class? Some outlets may get disconnected. When you delete a line of code, all you've deleted *is a line of code*. 
6. **No theming**: There's no way to reuse things like colours, border widths, corner radii or padding constants. More importantly, there's no way to easily respond to the whims of your designer and swap out all the occurrences of Sunset Orange to the infinitely more pleasing Sunrise Orange, or to re-skin an app with a totally different colour scheme by swapping out a JSON file containing definitions of all your semantically named colours (you do that, right?). 

## The Xcode 6 scorecard

1. **Splitting implementation of the layout and appearance over multiple files**: `IBInspectable` has gone a long way towards fixing this problem. It's essentially a more attractive, less "stringly"-typed version of the user-defined runtime attributes window. You get a nicely-named attribute in the inspector with a drop-down, text box, colour picker and so on depending on the attribute type. What's missing is support for enums or some other way of giving a custom set of options to choose from. 
2. **Constraints**: The constraint editor is now pretty good. Judicious use of container views and, most importantly, **naming your views** makes finding the right constraint easy, and when you select a view all of its constraints are accessible from the size inspector. Best of all, the Preview Assistant shows you live updates of your layout for as many simulated devices and orientations as you can fit on your screen. This is priceless, especially for universal apps.  
3. **iPad and iPhone versions**: Size classes and the preview assistant solve this problem *extremely* well. Device-specific storyboards are created at compile time, allowing backwards compatibility with iOS7. Between size classes you can install or uninstall views and constraints, or set different constants on constraints. It's possible to make radically different layouts for each device or orientation without too much work - again, container views help a lot with this. Adaptive segues are also very good. 
4. **git dirtying**: *sigh*
5. **Randomly destructive editing**: This still happens, but I'm not sure it can ever be solved. It might be more of a problem with the user than the software. 
6. **No theming**: There's no straightforward solution to this, but I was very pleased to discover that you can create categories with `IBInspectable` properties and these will be detected by interface builder. This mean, for example, you could have a `BOOL` property on a `UIView` category for a particular set of layer styles, and in interface builder, set this to `YES`. When the view is loaded, the setter is called. At this point you can configure the view according to your styles and hey presto - centralised theming is back! This is where enum support for inspectable properties would really shine - currently I'm using an inspectable string property combined with a dictionary and a lot of assert statements to access my precious semantic colours. 

## Conclusion

Enough of my objections have been overcome. More importantly, I've got to the end of a storyboard-using project and I don't wish that I'd done it without storyboards! There were still a few niggles, but nothing serious. My next project will also be storyboard-driven. I haven't even covered `IBDesignable`, since I didn't use it in this project, but a colleague has been using it and is very impressed.



