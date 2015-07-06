--- 
date: 2015-07-06
layout: post
title: It's time to ditch your Autolayout helper
--- 

When iOS 6 came out, we were in the early stages of planning for a major project. We managed to convince the client that the new app should be iOS 6 only, and decided to go all-in on the new features. 

The two biggest new features, which we used almost everywhere in [the app](https://itunes.apple.com/gb/app/hl-live-for-ipad/id762449126?mt=8), were collection views and autolayout. 

The app was built almost entirely in code, meaning a lot of constraint creation. My [autolayout helper category](https://github.com/jrturton/UIView-Autolayout) was written to help build it. 

Now it's time for that category, and all the others (and there are _[loads](https://cocoapods.org/)_), to be deprecated. 

<!--more-->

Any "helper" category or wrapper introduces the following costs versus using the native API:

- Installing and maintaining a dependency
- Learning an additional API or DSL 
- Masking the native functionality and therefore obscuring some understanding of how things actually work
- An extra possible source of bugs when things aren't working 
- Vulnerability to changes made by the OS vendor or library maintainer that break your work

If the helper category's benefits offset these costs, then fine, go ahead. But if they don't, you should not be using it. 

As an example, my category uses left/right instead of leading/trailing. It wasn't an issue for me (and I had no idea of the significance when I was writing it) since the app wasn't due to be localised. If your app will be localised for RTL languages, you shouldn't be using my category. 

It's also now recommended to create constraints first and activate them in a set. My code doesn't do that. My code doesn't take layout margins into account either. Do any of the others? I don't know, and I don't care, because they are all redundant. 

## Why did we need layout helpers?
- Interface builder was awful for creating constraints, therefore it was easier in code
- The constraint API was verbose and common tasks took too much boilerplate.  

## Why don't we need them any more?
- Interface builder should now be your default option for creating your UI. Seriously. It's really good now, and I can't imagine making an adaptive interface using size classes in code. 
- Stack views should be your default option for layout anyway. 

With these changes alone, you'd be making constraints in code so rarely that the benefits of adding a layout helper aren't worth it. I haven't used my own category in the last couple of projects I've worked on. 

But  there's more. [Layout Anchors](https://developer.apple.com/library/prerelease/ios/documentation/AppKit/Reference/NSLayoutAnchor_ClassReference/) make many common constraint creation tasks much easier as well. 

## What now?

Helper categories and wrappers have their place. But don't use them mindlessly. Learn enough about what they are helping you with to know if you need them, and re-assess at new version releases if they are still relevant and if they are being updated to reflect the state of the art. 

Using and understanding the first-party API should always be your preferred option. 

Now, excuse me while I go and write a helper library for the new Contacts API. 
