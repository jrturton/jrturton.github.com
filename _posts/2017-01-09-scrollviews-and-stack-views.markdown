--- 
date: 2017-01-09
layout: post
title: "Scrollviews and stack views"
--- 

File under: _Every time I do this, I forget how to do it, so I'm writing it down_. 

It's a common occurrence. You have some content in a stack view, and you want it to be scrollable. Maybe the content in the stack can change or you want keyboard avoidance or whatever. How do you set that up in a storyboard without either IB moaning at you, or the views not coming out right? 

<!--more-->

Here's the view hierarchy you need: 

1. The root view of the view controller
2. Your scroll view
3. Your stack view

The scroll view needs to be pinned to the leading and trailing margins, with a space of zero. It should also be pinned to the top and bottom of the root view (not the layout guides - you'll need to hold alt down while choosing constraints to get this instead). 

This will make your scroll view fill the entire scene. If you have a navigation bar or tab bart then it will sort the insets for you if the option "Adjust scroll view insets" is checked against the view controller (it is by default). This will let your content move behind the bars. 

At this point your scroll view will have a size, but it won't have a content size. The content size comes from the constraints between the scroll view and its subviews. 

You pin the top, left, bottom and right margins of the stack view to the scroll view. This tells the scroll view to make its content the same size as the stack. This is fine as far as the height goes (for a vertical stack view, the height is derived from the height of the arranged subviews) but it will have no idea about the width, and will be complaining at you.

To solve that, you need to also pin the leading and trailing edges of the stack view to the superview, which will give it something concrete to decide its width against. 

So, here's the summary of what you need (for a vertical stack view):

1. **Root view**
2. **Scroll view**, pinned to all edges of the **root view** (_not_ the layout guides)
3. **Stack view**, pinned to all edges of the **scroll view**, and leading and trailing edges pinned to the **root view**

You'll need to have content in the stack view for interface builder to cope with all this, too, but that's about the size of it.


