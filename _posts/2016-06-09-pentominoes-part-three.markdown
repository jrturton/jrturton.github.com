--- 
date: 2016-06-09
layout: post
title: "Pentominoes, part three: Placing the first tile"
--- 

It's time to think about some game logic. The first thing a player will do is try to place a `Tile` on the `Board`. How can I tell if the move should be allowed? 

To solve this problem I ended up creating a `SequenceType` and `GeneratorType`, which is the main focus of this part. 

<!--more-->

To see if a tile can go in a certain position on a board, I need to check each occupied square of the tile's grid, and each square that matches on the board's grid, and see if the spaces are all free. 

## Making a for...in loop 

I _could_ (and, indeed, did) do this by having an outer loop of rows and an inner loop of columns, but it felt a bit messy, and part of the point of this project is to look at some of the interesting things Swift permits you to do. A nicer way to work would be to be able to go something like:

```swift
for square in tile.squares {
	if square.occupied {
		... check the board at the relevant square
	}
}
```

The first step is to make a `struct` which can represent a square on a `PlayingGrid`:

```swift
public struct Square {
    let row: Int
    let column: Int
    let occupied: Bool?
}
```

`occupied` is an optional because sometimes I will want to create a Square and use it to query a `PlayingGrid`. 

To allow a for...in loop to work, you need two things: a `SequenceType`, and a `GeneratorType`. 

The sequence is the thing that is returned from, for example, `tile.squares` in the code above. Things like arrays also conform to `SequenceType`, which is how you can do for...in loops on them. 

`SequenceType` has a single function you have to implement. Once you have that function, you can `map`, `reduce`, `forEach` and many others. The function is `generate()`, and that has to return a `GeneratorType`. 

## GeneratorType

`GeneratorType` is a simple protocol. It has one function to implement, `next()`, which returns whatever type your sequence is going to be made of. Swift can infer that from the way you declare the function. 

In this case the generator is going to be a class, as it will need to hold some mutable state to track the last element that was generated. Here's the setup:

```swift
public class GridSquareGenerator: GeneratorType {
    var currentRow: Int = 0
    var currentColumn: Int = -1
    
    let grid: PlayingGrid
    
    public init(grid: PlayingGrid) {
        self.grid = grid
    }
}
```

And here is the all-important `next()` method:

```swift
public func next() -> Square? {
    guard currentRow < grid.rows.count else { return nil }
    
    currentColumn += 1
    
    if currentColumn == grid[currentRow].count {
        currentColumn = 0
        currentRow += 1
    }
    if currentRow < grid.rows.count {
        return Square(row: currentRow, column: currentColumn, occupied: grid[currentRow][currentColumn])
    } else {
        return nil
    }
}
```

This moves along the columns and down the rows, creating and returning the squares. Generators can take many forms, you can even have ones that are theoretically infinite (for example, ones that generate mathematical series). It all depends on the state you track and the implementation of `next()`.

## SequenceType

Creating the sequence is simple: 

```swift
public class GridSquareSequence: SequenceType {
    let grid: PlayingGrid
    
    public init(grid: PlayingGrid) {
        self.grid = grid
    }
    
    public func generate() -> GridSquareGenerator {
        return GridSquareGenerator(grid: grid)
    }
}
```

To make the sequence of squares of a `PlayingGrid` accessible, declare it like so:

```swift
extension PlayingGrid {
    public func squares() -> GridSquareSequence {
        return GridSquareSequence(grid: self)
    }
}
```

## Placing a tile

With all that mess hidden away inside the `PlayingGrid` protocol, it is now quite nice to add placement checking logic to the board:

```swift
extension Board {
    public func canPositionTile(tile: Tile, atSquare: Square) -> Bool {
        
        for tileSquare in tile.squares() {
            let boardSquare = tileSquare.offsetBy(atSquare)
            if !squareWithinBoard(boardSquare) {
                return false
            }
            if tileSquare.occupied == true && squareOccupied(boardSquare) {
                return false
            }
        }
        return true
    }
}
```

I added a couple of utility functions to make this more readable, not shown here. 

At this point, the blog posts have caught up to where I am in development. So from now on I'll be giving commit links to the repo for the state at the end of each post. Here's the [first](https://github.com/jrturton/Pentominoes/commit/b95d9b64753550e9e165abb8e987bc6c3dd1d991).