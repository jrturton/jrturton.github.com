---
layout: post
title: "Enumerate Using Block"
date: 2012-12-06 20:57
comments: false
categories: code
---

`enumerateObjectsUsingBlock:` is often the best way of passing through the contents of a collection. Objective-C fast enumeration (`for (... in ...)`) uses it under the hood, so it is fast, and you also have the advantage of the index number (for arrays) whilst iterating. 

What [some people appear not to know](http://stackoverflow.com/questions/13423478/is-down-casting-within-block-parameters-a-bad-practice) is that you don't have to use `id` as the object parameter for the enumeration block. You don't have to call it `obj` either. 

<!--more-->

Instead of:

```objc

[myArray enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
    UIView *view = obj;
    if (indexOfViewToShow != idx)
        view.hidden = YES;
}];

```

It is perfectly allowable, and _much_ more readable, to do:

``` objc
[myArray enumerateObjectsUsingBlock:^(UIView *view, NSUInteger idx, BOOL *stop) {
    if (indexOfViewToShow != idx)
        view.hidden = YES;
}];
```

Of course, you have to be sure that everything in the array is a `UIView`, but this warning also applies to the first example. 

On a related note, a similar principle applies when coding `IBAction` methods, though this has become more obvious now you can create actions by control-dragging - `sender` doesn't have to be an `id`, it can be whatever class the originating object is.  