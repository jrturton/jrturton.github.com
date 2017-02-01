--- 
date: 2017-02-01
layout: post
title: "Embedding in scrollviews"
--- 

You've built a view controller in a storyboard. It started out quite simple, but you added bits here and there and the complexity grew. Now, a new requirement arrives. 

We need a text field.
We need another set of content at the bottom. 

All of a sudden, your layout won't fit on the screen any more. You need it to scroll. That's no problem, all you need to do is select the root view of the view controller, and choose **Embed in... > Scroll View**, right? 

Oh. That's not available when you're dealing with the root view. So how can you do it? How do you make a non-scrolling view controller scrollable?

<!--more-->

You **don't** drag a scroll view onto the view, so it's highlighted, then drop it - that will completely replace your view and all of its contents! 

You **can't** change the class of the root view in the identity inspector to a scroll view - not only does this not work, it also isn't a good idea, since you still want the root view to be a plain `UIView`. 

You want to go from this: 

**View** -> **Contents**

To this:

View -> Scrollview -> **View** -> **Contents**

You have tons of constraints and outlets linking the contents to the root view, and you don't want to have to rebuild or reconnect them. 

Luckily, there is a way to do it: 

- If you've used the top or bottom layout guides then you'll need to re-create those constraints against the top or bottom edges of the container instead. Hold alt after control-dragging to show the container instead of the layout guides in the add constraints menu.
- Make sure that you have a solid line of top-to-bottom and constraints so that the scroll view can work out its content height.
- Select the root view in the document outline, then drag it out of the view controller tree and drop it elsewhere in the scene, after the **Exit** segue point, for example. 
- The view should have maintained the same size, the contents and constraints will all be there, and the outlets will still be connected.
- The view controller will now be missing a root view. Drag in a UIView, which will fill the scene.
- Drag in a scroll view and pin it to all edges of the root view (don't use the layout guides, this is a very similar principle to the [previous article](http://commandshift.co.uk/blog/2017/01/09/scrollviews-and-stack-views/)
- Drag your original view back in so it's a subview of the scroll view. You'll see some red, don't worry.
- Pin the original view to all edges of the scroll view. At this point you'll still have some layout warnings.
- Pin the original view's width to that of the new root view (hat tip to [@calicoding](https://twitter.com/calicoding) for pointing out that nicer alternative to pinning to the root view's leading and trailing edges). All of your warnings should now be gone.

You're done - all of your original content is now safely embedded in a scroll view, with constraints and outlets intact.   
