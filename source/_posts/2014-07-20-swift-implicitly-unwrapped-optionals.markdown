---
layout: post
title: "Implicitly Unwrapped Optionals in Swift"
date: 2014-07-20 12:23
comments: false
categories: 
---
Implicitly unwrapped optionals don't make sense at first glance. It's optional, but you're always going to assume it contains a value? What's optional about that?

<!--more-->

From the Swift / iOS work I've done so far the principal use has been for properties that are always set, just not at initialization. This suits the lifecycle of Cocoa objects quite well - look at `UIViewController`, for example. The `view` property, and any subviews you might create during `loadView` or `viewDidLoad` don't exist until the view has been loaded, which does _not_ occur at the same time the view controller is initialized. 

If outlets were declared as simple `var` properties, a default value would need to be set or the initializer would have to set them, otherwise the project would not compile. Setting the property type to an Optional would remove this requirement, since Optionals get a default value of `.None`, but then you'd have to unwrap it all over your code:

```
var label : UILabel?
...
label!.text = "Unwrapped label"
```

This works! but! it! looks! terrible!

Making the property an implicitly unwrapped optional is functionally identical but much more readable:

```
var label : UILabel!
...
label.text = "Unwrapped label"
```

It's not just for UI components - anything you _always_ pass in to another object, but not at initialization (a managed object context, for example) is best handled as an implicitly unwrapped optional. Apple recommend this for outlets:

>"When you declare an outlet in Swift, you should make the type of the outlet an implicitly unwrapped optional. This way, you can let the storyboard connect the outlets at runtime, after initialization. When your class is initialized from a storyboard or xib file, you can assume that the outlet has been connected."

>_Excerpt From: Apple Inc. [“Using Swift with Cocoa and Objective-C.”](https://itun.es/gb/1u3-0.l)_

_**Note:** this behaviour changed between beta 3 (where outlets were "secret" implicitly unwrapped optionals) and beta 4 (where outlets became, well, explicit implicitly unwrapped optionals)_
 
If it's good enough for Apple, it's good enough for the rest of us. Disconnected outlets now lead to a new runtime error already [climbing up the rankings in Stack Overflow: `Can't unwrap Optional.None`](http://stackoverflow.com/q/24475889/852828). 

You can still check for a value in an implicitly unwrapped optional, if there's a chance it will not have a value:

```
if label != nil {
	label.text = "Yes, there's a label here"
}
```

Apple recommend that implicitly unwrapped optionals are only used where there isn't a chance that the value will be set to `nil` later on. The intended use case is when you are definitely setting a value, just not at initialization.

The other common place you see implicitly unwrapped optionals is when implementing or overriding a lot of Cocoa methods:

>Because Objective-C does not make any guarantees that an object is non–nil, Swift makes all classes in argument types and return types optional in imported Objective-C APIs. Before you use an Objective-C object, you should check to ensure that it is not missing.

>In some cases, you might be absolutely certain that an Objective-C method or property never returns a nil object reference. To make objects in this special scenario more convenient to work with, Swift imports object types as implicitly unwrapped optionals. Implicitly unwrapped optional types include all of the safety features of optional types. In addition, you can access the value directly without checking for nil or unwrapping it yourself. 

>Excerpt From: Apple Inc. [“Using Swift with Cocoa and Objective-C.”](https://itun.es/gb/1u3-0.l)

So when implementing table view delegate methods, for example, you'll get this:

```
func tableView(tableView: UITableView!, numberOfRowsInSection section: Int) -> Int {
```

The table view is passed in as an implicitly unwrapped optional. In this case you can be sure that a table view will actually be passed in, so you don't need to check, but in other cases you may have to ensure that a value exists.

This is the tradeoff of Optionals. We've got a unified representation of Nothingness, as discussed [here previously](http://commandshift.co.uk/blog/2014/06/11/understanding-optionals-in-swift/), but we lose the convenient safety (or, sometimes, silent bugginess) of messaging `nil`.

