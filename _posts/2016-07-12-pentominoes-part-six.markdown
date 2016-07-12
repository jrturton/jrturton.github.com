--- 
date: 2016-07-12
layout: post
title: "Pentominoes, part six: Gestures"
--- 

After some brief experimentation I decided that live views weren't good enough to use for playing around with an interactive board and gestures. Also, once you get to dealing with gestures and user interaction with something like this, you really need to be working on a device - things that feel fine with a mouse or trackpad and pointer can be no good at all when you're working on a device. 

In this post I'm going to talk about adding gesture recognisers and transferring to a full project.

<!--more-->

## The limits of playgrounds

This project has come a long way with playgrounds, but there are a few obstacles which make me think it can't go much further. I made and displayed a view controller within the playground, then decided to change to a full project, because of the following limitations: 

- Source code can't be shared between a standard Xcode project and a playground, so you can't do "live fiddling" in the playground then build the project, incorporating your changes - unless I've missed something. 
- Live view rendering in playgrounds doesn't look quite right - it seems to come out slightly blurry. 
- The layout life cycle of a view controller's view in a playground also didn't seem to come across all that clearly. I found myself adding code to work around the way the playground was working, which seemed a waste of time. 

I created a new single view Xcode project for the iPad, working in landscape mode only, and moved across the model and view files from the playground. 

There's a single view controller which takes a `board` and `tiles` property, and has a `boardView` and set of `tileViews` based off them. At the moment I'm just setting this up in the app delegate as I want to get the game mechanics working before I worry too much about setup and scoring and so on. 

In `viewDidLayoutSubviews()` I position the tiles evenly around the board. When you start the game, it looks like this:

![Pentominoes running on iPad](/images/pentominoes/Pentominoes_6_gameStart.png)

The code for that is fairly boring and possibly not what I'm going to end up with when this is all over, so I won't go into detail on that, you can look in the repo if you're interested. 

## Gesture recognisers

The main interaction the player will have is to drag tiles around the board. You do this with a pan gesture.

The view controller acts as the pan gesture's delegate. It isn't allowed to begin unless it begins at the current location of a tile:

```swift
public func gestureRecognizerShouldBegin(gestureRecognizer: UIGestureRecognizer) -> Bool {
    if gestureRecognizer != pan {
        return true
    }
    let location = gestureRecognizer.locationInView(view)
    if let hitTile = view.hitTest(location, withEvent: nil) as? TileView {
        activeTile = hitTile
        return true
    }
    return false
}
```

When this happens it sets the view controller's `activeTile` property, which in turn transitions that tile to the lifted state.

In the handler for the gesture, I move the tile around to follow the movement:

```swift
@IBAction func handlePan(sender: UIPanGestureRecognizer) {
    var fingerClearedLocation = sender.locationInView(view)
    fingerClearedLocation.y -= (activeTile?.bounds.height ?? 0.0) * 0.5
    switch sender.state {
    case .Began:
        UIView.animateWithDuration(0.1) { self.activeTile?.center = fingerClearedLocation }
    case .Changed:
        activeTile?.center = fingerClearedLocation
    case .Ended, .Cancelled:
        // TODO: put it down in a permissible place
        activeTile = nil
    default:
        break
    }
}
```

Another subtlety that you don't notice in the simulator - when you're dragging a tile around, your finger is covering most of the tile. In the code above I'm offsetting the location of the gesture so that you can see the whole tile and, more importantly, where it's going to go. 

With the current code, the tile will go wherever the user lets go - none of the rules I spent so long building are being taken into account yet. However, the game is almost playable right now! 

The only missing thing is the ability to rotate. The user is going to do this by tapping the screen while they are moving a tile. The tap gesture is handled like this:

```swift
@IBAction func handleTap(sender: UITapGestureRecognizer) {
    activeTile?.rotate(true)
}
```

Nothing happens if there isn't an active tile. 

There are a couple of subtleties to note when using taps together with pans. First, the tap is ignored if a pan is active, unless you implement the following delegate method: 

```swift
public func gestureRecognizer(gestureRecognizer: UIGestureRecognizer,
 shouldRecognizeSimultaneouslyWithGestureRecognizer otherGestureRecognizer: UIGestureRecognizer) -> Bool {
    return true
}
```

Second, unless you explicitly set a maximum number of touches on the pan (the default is infinity), the tap is also interpreted as being part of the pan. Since the location of a gesture is the average location of all the touches within that gesture, this means that the tile jumps to between your two fingers when you are panning and touching. Setting the maximum touches to 1 stops this. 

As it stands now the game is playable, but with a few problems. These will be addressed in upcoming posts: 

- The rule system needs to be incorporated so that the tiles can only be placed on the board where they are allowed to go.
- Picking up tiles that are close to each other seems a little unpredictable, because the entire tile area rather than the populated area is used for touches.
- It might be worth looking at a long press gesture to pick up tiles in this situation, so the user can tell which tile they are going to move when dealing with tiles on the board.

The repo as at the end of this post is at [this commit](https://github.com/jrturton/Pentominoes/commit/72b24c33a1824b6de23bf5bbee9e73d321a721d4).