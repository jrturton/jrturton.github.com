---
layout: post
title: "Creating individual layout constraints"
date: 2013-02-20 20:16
comments: false
categories: code
---

Other posts in the Autolayout series:

- [Autolayout](/blog/2013/01/13/autolayout/)
- [Autolayout in interface builder](/blog/2014/03/10/autolayout-in-interface-builder-xcode-5-dot-1/)
- [Visual format language for autolayout](/blog/2013/01/31/visual-format-language-for-autolayout/)

In the previous two posts in this series we've talked about creating constraints in interface builder and in code using the visual format language. In this final part I'm going to discuss creating individual constraints, and also some category methods on `UIView` that can make creating layouts in code a lot simpler, and to make your layout code much more readable.  

Creating a single constraint is done using this method:

{%codeblock lang:objc %}

+ (id)constraintWithItem:(id)view1 
                attribute:(NSLayoutAttribute)attr1 
                relatedBy:(NSLayoutRelation)relation 
                   toItem:(id)view2 
                attribute:(NSLayoutAttribute)attr2 
               multiplier:(CGFloat)multiplier 
                 constant:(CGFloat)c

{%endcodeblock%}

Even writing the definition takes several lines, so you can see why this method is not recommended as a way to build your entire layout. However in some cases this method is the only way to build the layout you need, particularly since it exposes the multiplier and constant properties, which are not readily available for the other methods of constraint creation. 

<!--more-->

First, let's have the usual discussion of each argument in the main method and try to explain what it is all for. Remember that constraints are usually expressed as:

    attribute1 == multiplier x attribute2 + constant
    
`==` can also be `>=` or `<=`, and in some cases, `multiplier` and `attribute2` are irrelevant, so we are just looking at:

    attribute1 == constant
    
With that in mind, let's go through the method. It returns a single instance of `NSLayoutConstraint`, rather than the array of constraints returned by the visual format language method. 

- `view1`

The view that you want to be affected by the constraint. 

- `attr1`

The attribute of `view1` that you want to be controlled by the constraint, such as the left edge, the center X position, the width and so forth. This is a single value from the `NSLayoutAttribute` enum. 

- `relation`

The relationship between the attribute and the right hand side of the constraint equation. Either `NSLayoutRelationEqual`, `NSLayoutRelationLessThanOrEqual` or `NSLayoutRelationGreaterThanOrEqual`. 

- `view2`

The view that has the attribute that you want to use to define the right hand side of the constraint equation. If you are just setting a value to a constant, use `nil` for this argument.

- `attr2`

The attribute of `view2` that you want to use to define the right hand side of the constraint equation. If you are just setting a value to a constant, use `NSLayoutAttributeNotAnAttribute` for this argument. 

- `multiplier`

The value by which `attr2` should be multiplied to give the value for `attr1`. 

- `constant`

The value to add to `attr2` (after the multiplier has been used) to give the value for `attr1`.
    
So far, so much rewording of the existing documentation. Let's look at some concrete examples. 

### Fix a view to a specific width

{%codeblock lang:objc %}

[view addConstraint:[NSLayoutConstraint constraintWithItem:view 
	attribute:NSLayoutAttributeWidth 
	relatedBy:NSLayoutRelationEqual 
	toItem:nil 
	attribute:NSLayoutAttributeNotAnAttribute 
	multiplier:1.0 
	constant:200.0]];

{%endcodeblock%} 

Sets the width of `view` to 200.0 points.

### Make a view half the width of its superview

{%codeblock lang:objc %}

[view addConstraint:[NSLayoutConstraint constraintWithItem:view
	attribute:NSLayoutAttributeWidth 
	relatedBy:NSLayoutRelationEqual 
	toItem:superview 
	attribute:NSLayoutAttributeWidth 
	multiplier:0.5 
	constant:0]];

{%endcodeblock%} 

### Pin a view's edge to the edge of another view

{%codeblock lang:objc %}

[view addConstraint:[NSLayoutConstraint constraintWithItem:view
	attribute:NSLayoutAttributeLeft 
	relatedBy:NSLayoutRelationEqual 
	toItem:view2 
	attribute:NSLayoutAttributeRight 
	multiplier:1.0 
	constant:0.0]];

{%endcodeblock%} 

### Centre a view on an axis within its superview

{%codeblock lang:objc %}

[view addConstraint:[NSLayoutConstraint constraintWithItem:view
	attribute:NSLayoutAttributeCenterX 
	relatedBy:NSLayoutRelationEqual 
	toItem:superview 
	attribute:NSLayoutAttributeCenterX 
	multiplier:1.0 
	constant:0.0]];

{%endcodeblock%} 


It's all pretty simple, but pretty verbose too. All of the invocations look very similar, so when reading back the code it is hard to see the intent without having to parse it manually. We can do better.

##NSLayoutConstraint creation - the missing methods

### Fix a view to a specific size

{%codeblock lang:objc %}
[view constrainToSize:CGSizeMake(200.0,100.0)];
{%endcodeblock%}

A size of 0.0 in either dimension means that no constraint will be created.

### Pin a view to its superview's edges

Very often you want to pin a view to one or more of the edges of its superview, sometimes with an inset. That's at least two lines of VFL, or more if you're creating the constraints individually. 

{%codeblock lang:objc %}
[view pinToSuperviewEdges:JRTViewPinLeftEdge | JRTViewPinRightEdge inset:0.0];
{%endcodeblock%}

_(A new bitmask had to be introduced here to allow combining edges, and to give the option of pinning all edges with a single value)_

### Pin a view to the edge of another view

{%codeblock lang:objc %}
[view pinEdge:NSLayoutAttributeLeft toEdge:NSLayoutAttributeRight ofView:view2 inset:0.0];
{%endcodeblock%}

### Centre a view within its superview

{%codeblock lang:objc %}
[view centerInContainerOnAxis:NSLayoutAttributeCenterX];
{%endcodeblock%}

or

{%codeblock lang:objc %}
[view centerInView:superview];
{%endcodeblock%}

## A note on the approach

I've chosen to use a category on UIView to achieve things here since it gives a more direct expression of intent, and it also leads to more concise code. These methods have all been used by me in production code, and were added as I felt a requirement for them. 

A category on `NSLayoutConstraint` would be an alternative, with the following advantages and disadvantages: 

Advantages:

- More consistent with the existing API
- Gives access to the constraints as they are created (if you wanted to store them in properties for later adjustment or bulk removal / application)

Disadvantages:

- More verbose code (`addConstraint:` and `addConstraints:` all over the place)
- Would need to pass in view references to all the methods, making them even longer
- Not as immediately readable (in my opinion)

The only significant advantage is the second one, but I've found that the times I want to store a specific constraint are rare enough that I just use the long-winded method to make it in that case. 

The category code is available on [GitHub](https://github.com/jrturton/UIView-Autolayout), and you can also get hold of it via [CocoaPods](http://www.cocoapods.org/?q=autolayout).





 
