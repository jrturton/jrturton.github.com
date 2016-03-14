---
layout: post
title: "Better snapshots"
date: 2016-03-13 21:00:03 +0100
comments: false
categories: 
---

In my talk at [RWDevCon 2016](http://rwdevcon.com) I described a way to handle custom view controller transitions in a way that didn't lead to messing up your view controllers and leaving yourself with a lot of horrible cleanup code. 

The secret to painless custom transitions is to do a lot of snapshots. In the talk I mentioned some utility methods for making snapshots, but there wasn't time in the session to cover the details. Instead, I've written about them here. 

<!--more-->

A sensible view controller transition doesn't move, fade or otherwise manipulate any of the view controller's views. That way lies madness. The correct way to handle them is:

- Create a canvas view, which fills the container view of the transition context
- Snapshot everything you're interested in moving
- Move the snapshots around, then animate them to their final positions
- Remove the canvas when you're done

This has numerous advantages:

- No cleanup code or resetting of view properties
- You can take advantage of autolayout in your view controllers, but move the snapshots around by animating the `center` property, which makes much more sense when doing transitions and means you don't need outlets to all your constraints
- Keeps the view controller very loosely coupled to the transition

Making a snapshot from a UIView is simple and fast:

```swift
let snapshot = view.snapshotViewAfterScreenUpdates(false)
```   

The resulting view has the same `bounds` as the original, but has a frame origin at zero, which is usually useless. 

For transitions, what is usually required is for the snapshot to be added to the canvas view, at the same position on the screen. This is done with an extension on `UIView`:

```swift
func snapshotView(view: UIView, afterUpdates: Bool) -> UIView {
    let snapshot = view.snapshotViewAfterScreenUpdates(afterUpdates)
    self.addSubview(snapshot)
    snapshot.frame = convertRect(view.bounds, fromView: view)
    return snapshot
}
```

You call the function like this:

```swift
let snapshot = canvas.snapshotView(view, afterUpdates: false)
```

For an array of views, another method in the extension: 

```swift
func snapshotViews(views: [UIView], afterUpdates: Bool) -> [UIView] {
    return views.map { snapshotView($0, afterUpdates: afterUpdates) }
}
```

Snapshotting is so common and so useful when doing transitions, and these extension methods make your transition code much more readable and obvious. 

The full source is here: 

<script src="https://gist.github.com/jrturton/76d71410d6dc7a643def.js"></script>