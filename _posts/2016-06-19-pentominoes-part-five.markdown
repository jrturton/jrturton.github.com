--- 
date: 2016-06-19
layout: post
title: "Pentominoes, part five: Some drawing"
--- 

It's time to move on from the character-based visualisations of the board and the tiles, and create some views. 

Each tile will be represented with a view, and the board will be a view. Placed tiles will be added as subviews of the board, which will simplify the drawing and positioning logic. 

<!--more-->

## Tiles

A `TileView` is initialised with a `Tile` model, a grid size (the edge of one of the squares that make up the view) and a colour. I originally had a method in here to create a path from the occupied squares, but moved this out to the `PlayingGrid` protocol since it ended up also applying to the board. 

The tile view has a shape layer derived from this path, and an optional drop shadow (to be shown when the player has "picked up" the tile. It looks like this:

![Lifted single tile](/images/pentominoes/Tile.png)

The tile view has a `rotate` method which will animate a rotation, then update the tile model and redraw after it is done. This keeps the model and view in sync without being too complicated, and it means I don't have to keep track of the rotated state of any views or the model. It's possible that won't work well with the UI (I might want to allow two or three rotations to be performed at once) but it will do for now. I'd rather get something working than overcomplicate before I'm actually at a specific stage in the project. The rotation method currently looks like this: 

```swift
func rotate(clockwise: Bool) {
    UIView.animateWithDuration(0.1, animations: {
        self.transform = CGAffineTransformMakeRotation(clockwise ? CGFloat(M_PI_2) : CGFloat(-M_PI_2))
        }, completion: { _ in
            self.transform = CGAffineTransformIdentity
            self.tile.rotate(clockwise)
            self.shapeLayer.path = self.tilePath()
    })
}
```

The `lifted` property will be toggled when the user picks up or places a tile. It affects the shadow of the shape layer in a property observer:

```swift
public var lifted: Bool = false {
    didSet {
        layer.shadowRadius = 5.0
        layer.shadowOffset = .zero
        layer.shadowColor = UIColor.blackColor().CGColor
        layer.shadowOpacity = lifted ? 0.5 : 0.0
    }
}
```

## The Board

A `BoardView` is initialised with a `Board` model and a grid size. It contains a shape layer which draws the empty board. This is the same path algorithm that I use for the tile, except the board draws empty squares, and the tile draws occupied squares. At the moment, it doesn't do anything else.

## Making the paths

This is one of those functions that probably be collapsed into a single-liner in Swift, but that's usually not a path I like to go down. It makes things unreadable and the intermediate steps are invisible as far as debugging is concerned. Who benefits from making code as short as possible? 

I went with this in the end. It makes each stage of the process pretty clear. It could perhaps be replaced entirely with a single for..in loop or `reduce`, but I like the fact that this structure shows several distinct steps. This means that if I decided to do additional things during the building of this path, or if I required just the `[CGRect]` output, it would be simple to split things out into separate functions.

This is an extension of `PlayingGrid`:

```swift
public func pathForSquares(occupied: Bool, gridSize: CGFloat) -> CGPath {
    
    let squaresForPath = squares().filter { $0.occupied == occupied }
    
    let rects : [CGRect] = squaresForPath.map { square in
        let originX = CGFloat(square.column) * gridSize
        let originY = CGFloat(square.row) * gridSize
        return CGRect(x: originX, y: originY, width: gridSize, height: gridSize)
    }
    
    let path : UIBezierPath = rects.reduce(UIBezierPath()) { path, rect in
        path.appendPath(UIBezierPath(rect: rect))
        return path
    }
    
    return path.CGPath
}
```

## A fast visualisation

I wanted to do a quick check that my board view and tile views were playing nicely with the logic I'd built in the earlier sections. This was a few lines of code to achieve in a playground:

```swift
let board = Board(size: .SixByTen)
let boardView = BoardView(board: board, gridSize: 30)
let shapes = (0..<11).map { Shape(rawValue:$0)! }
let tiles = shapes.map { Tile(shape: $0) }
for tile in tiles {
    for square in board.squares() {
        if board.canPositionTile(tile, atSquare: square) {
            board.positionTile(tile, atSquare: square)
            let tileView = TileView(tile: tile,color: randomColor(), gridSize: boardView.gridSize)
            boardView.addSubview(tileView)
            tileView.frame.origin = board.pointAtOriginOfSquare(square, gridSize: boardView.gridSize)
            break
        }
    }
}
```

Here I am just going through each possible tile and placing it in the first possible square on the board. 

Adding the `boardView` inline shows this:

![Board with tiles](/images/pentominoes/BoardWithTiles.png)

It's starting to look real! 

The code at the end of this post is at [this commit](https://github.com/jrturton/Pentominoes/commit/8864d0464d301ad98830634f9480c21b8bbabde7).