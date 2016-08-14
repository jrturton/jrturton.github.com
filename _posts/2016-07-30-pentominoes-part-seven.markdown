--- 
date: 2016-07-30
layout: post
title: "Pentominoes, part seven: Dropping with intent"
--- 

I last left the project with the player being able to pick up and drop tiles on the board, but with no implementation of the game logic that I'd spent all that time building up. 

In this post I'm going to add the game logic in to the drag and drop action.

My goals are: 

- To show a highlighted "drop zone" on the board as the player moves a tile around. The drop zone will show them where the tile can be dropped
- To "snap" the tile into the drop zone if the drag is ended while the tile is in a permissible position
- Otherwise, to snap the tile back to its position at the edge of the board
- Sort out the messy pickup of tiles on a busy board

<!--more-->

## Where am I? 

The tile placing rules work around the `Square` in the top left of the tile - that's the place the `Board` uses to assess if a tile can go in a specific location. 

I need to do a lot of extra work in the `handlePan(_:)` method. 

When the `TileView` is being dragged around, the origin of the view is going to line up with the origin of the top left square in the tile. This can be converted into the coordinate system of the board like so:

```swift
let locationOnBoard = boardView.convertPoint(activeTile.bounds.origin, fromView: activeTile)
```

The `convertPoint / rect` family of functions trip a lot of people up, mostly because you need to remember that a view's `frame` and `center` are in the superview's coordinate system, which then needs to be the `fromView` that you use. It's my [5th highest voted Stack Overflow answer!](http://stackoverflow.com/questions/8465659/understand-convertrecttoview-convertrectfromview-convertpointtoview-and/8465817#8465817). If you use coordinates from the `bounds` instead, you can convert from the view itself, so it's a lot simpler.

In the first attempt at this, I just converted the point to a square on the board, checked that, and let the player drop it if it was OK. However, that didn't play very well - when placing tiles on an almost-full board, it was much nicer for the game to hunt around for nearby locations that could serve as drop zones, otherwise the player is holding a tile _almost_ over a perfectly good location and it isn't being highlighted as a drop.  

At that point the logic got more complicated than I was happy with inside a gesture handler, so I spun it out into a method on `Board`, which is probably where it should have been the whole time. 

It ended up looking like this: 

```swift
public func allowedDropLocation(tile: Tile, atPoint point: CGPoint, gridSize: CGFloat) -> Square? {
    let potentialSquare = squareAtPoint(point, gridSize: gridSize)
    var allowedDropLocation: Square?
    if canPositionTile(tile, atSquare: potentialSquare) {
        allowedDropLocation = potentialSquare
    } else {
        var distanceToDropPoint = CGFloat.max
        for square in squaresSurrounding(potentialSquare) {
            if canPositionTile(tile, atSquare: square) {
                let origin = pointAtOriginOfSquare(square, gridSize: gridSize)
                let xDistance = origin.x - point.x
                let yDistance = origin.y - point.y
                // No need to sqrt since we're just comparing
                let distance = (xDistance * xDistance) + (yDistance * yDistance)
                if distance < distanceToDropPoint {
                    distanceToDropPoint = distance
                    allowedDropLocation = square
                }
            }
        }
    }
    return allowedDropLocation
}
```

This did mean I had to `import CoreGraphics` into `Board`, making my "pure swift" model object not quite so pure, but honestly, who cares? It feels like the right place for this method to go. 

In brief, the method finds the square at the correct location, checks if that's a permissible drop, if not, it gets a 3x3 grid of squares centred on that square and checks each one of those, picking the one whose centre is closest to the tile's location. 

This could possibly do with some tweaking - obviously it favours placing the tile at the "actual" square even when the drop location is really near the edge: 

![tile being dropped near the edge of its drop zone](/images/pentominoes/DropLocation.png)

But I think I'll play a while and see if it's a problem. 

`squareAtPoint` is a new function on `PlayingGrid` that returns a `Square` given a particular grid size and point:

```swift
public func squareAtPoint(point: CGPoint, gridSize: CGFloat) -> Square {
    let row = Int(floor(point.y / gridSize))
    let column = Int(floor(point.x / gridSize))
    return Square(row: row, column: column)
}
```

## Showing the drop zone

The drop zone is going to be the shape of the tile, drawn on the board. I already have a method to get a `CGPath` out from a grid, so I add a `CAShapeLayer` to `BoardView`. Because we're dealing with locations at the top left of tiles, the `anchorPoint` of the layer is set to (0, 0), so that I can set its `position` directly corresponding to the square that I'm planning to drop it on. All of this configuration can be done as part of the property declaration, like this:

```swift
private let highlightLayer: CAShapeLayer = {
    $0.anchorPoint = CGPoint(x: 0, y: 0)
    $0.fillColor = UIColor(red: 0.0, green: 1.0, blue: 0.0, alpha: 0.25).CGColor
    $0.strokeColor = UIColor.darkGrayColor().CGColor
    $0.lineWidth = 4.0
    $0.hidden = true
    return $0
}(CAShapeLayer())
```

I'm a fan of this style - it prevents having a lot of setup code in the initializer of the view. 

I add a calculated property to `BoardView` so the path can be updated: 

```swift
var dropPath: CGPath? {
    set {
        let origin = highlightLayer.position
        CATransaction.begin()
        CATransaction.setDisableActions(true)
        highlightLayer.path = newValue
        highlightLayer.position = origin
        CATransaction.commit()
    }
    get {
        return highlightLayer.path
    }
}
```

This is called from the `activeTile` `didSet` property observer in the view controller:

```swift
boardView.dropPath = activeTile?.tile.pathForSquares(true, gridSize: gridSize)
```

In the early draft of this code I had a lot of extra state that I was resetting at different points in the gesture handling, until I realised it made more sense just to tie it to the `activeTile` property. 

The last part of showing the drop zone is to place the highlight layer in the correct place and make it visible. This method on `BoardView` takes care of that:

```swift
func showDropPathAtOrigin(origin: CGPoint?) {
    CATransaction.begin()
    CATransaction.setDisableActions(true)
    if let origin = origin {
        highlightLayer.position = origin
        highlightLayer.hidden = false
    } else {
        highlightLayer.hidden = true
    }
    CATransaction.commit()
}
```

If no point is passed in, the layer is hidden, otherwise it is moved. Notice that here and in the `dropPath` setter, I'm using a `CATransaction` to prevent implicit animations happening. Without that code, the path and position changes would be animated, which isn't the behaviour I'm after.

Back in the gesture handler, I can call the `allowedDropLocation` method discussed earlier, find the origin of the square, and use that to position or hide the drop layer:

```swift
if let allowedDropLocation = board.allowedDropLocation(activeTile.tile, atPoint:locationOnBoard, gridSize: gridSize) {
    let squareOrigin = board.pointAtOriginOfSquare(allowedDropLocation, gridSize: gridSize)
    boardView.showDropPathAtOrigin(squareOrigin)
} else {
    boardView.showDropPathAtOrigin(nil)
}
```

## Dropping the tile

Currently when the player drops a tile, it stays where it is. Now that I know if the tile can fit on the board at its current position, I can take this into account when the gesture ends:

```swift
let locationOnBoard = boardView.convertPoint(activeTile.bounds.origin, fromView: activeTile)
let allowedDropLocation = board.allowedDropLocation(activeTile.tile, atPoint:locationOnBoard, gridSize: gridSize)

self.activeTile = nil

if let allowedDropLocation = allowedDropLocation {
    board.positionTile(activeTile.tile, atSquare: allowedDropLocation)
    boardView.addSubviewPreservingLocation(activeTile)
    UIView.animateWithDuration(0.1) {
        activeTile.frame.origin = self.board.pointAtOriginOfSquare(allowedDropLocation, gridSize: self.gridSize)
    }
} else {
    UIView.animateWithDuration(0.25) {
       self.positionTiles()
    }
}
``` 

I repeat the logic for confirming the drop location - this seemed better than having a property for the allowed drop location and having to reset it all the time (which is what I originally had). If there's a valid location, then the tile is added to the board at the correct position, and the tile view is made a subview of the board. I wrote a utility method to move a view to a new superview, whilst keeping the same location on the screen. 

There is then a short animation - either to snap the tile into place on the board, or to return it to "home" around the edge. `positionTiles` is the tile placing code that was in `layoutSubviews`, which has been moved to its own function. 

The `activeTile` property is set to nil before this happens, because that is used when the tiles are being positioned.

## Fixing tile pickup

Tile pickup is currently done by hit testing, which means that the order of subviews drives the tile selection. Each tile is a square much larger than the occupied tiles, as you can see in this screenshot from [Reveal](http://revealapp.com) (get Reveal, it's the best):

![tile views overlapping](/images/pentominoes/Overlap.png)

Each tile is a 5x5 grid of squares. As you can see, the V tile (with the orange background) is almost completely covered by the S tile, with the red background. Most attempts to pick up the V will result in the S being picked up instead. 

This is only a problem for tiles placed on the board, so I can override `hitTest(_: withEvent:)` on `BoardView` to fix it. I can get the appropriate square using the same method used when handling the pan gesture, see if there's a tile at that square, and if so, return the appropriate tile view, otherwise use the super implementation:

```swift
public override func hitTest(point: CGPoint, withEvent event: UIEvent?) -> UIView? {
    
    let square = board.squareAtPoint(point, gridSize: gridSize)
    if
        let tile = board.tileAtSquare(square),
        let tileView = (tileViews.filter{ $0.tile.shape == tile.shape }).first {
        return tileView
    }
    
    return super.hitTest(point, withEvent: event)
}
```

To get this work I needed a way to check equality on tiles - this meant adding a  property to hold the `Shape` that the tile was initialized with. 

That's all for this time - the current state of the code is [here](https://github.com/jrturton/Pentominoes/commit/44f9e6b55ff3cd4716888c898d7127a448115049). The game is now fully working - but it's not very jazzy. In the next post I'll be adding some "juice" - little touches to make it more fun.  
