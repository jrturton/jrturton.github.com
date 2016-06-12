--- 
date: 2016-06-12
layout: post
title: "Pentominoes, part four: Updating the board"
--- 

So far my `Board` model is smart enough to tell if a `Tile` can be placed in a specific location. Now I need to think about what happens when the player actually places the tile. How does the board update its model? What needs to happen here? 

<!--more-->

The `rows` property of the board needs to change so that the squares now occupied by the tile are marked as such. Simply updating the relevant positions in the array isn't going to be good enough, though. The user may pick up a tile again - how will I know which tile it was, and which squares to mark as unoccupied again? 

There are many possible solutions to this problem. I decided to do it like this:

- Store the `rows` configuration of an empty board when the board is initialised
- Create a new struct which holds a tile and its location on the board
- Have a private array of these structs to record placed tiles
- Have a property observer on this array which updates `rows` by adding all placed tiles onto the empty board.

I toyed with the idea of making `rows` a calculated variable but that seemed like it would add a lot of processing overhead (the `rows` property is going to be accessed quite often) without making things much simpler. The only concern with the current model is that if a placed tile is rotated, this will not be reflected in the board's model. However, a user will not be able to rotate a placed tile - they will have to remove it from the board to do that. 

The empty board is just a constant `[[Bool]]` which is assigned during init. Here is the struct and property:

```swift
private struct PlacedTile {
    let square: Square
    let tile: Tile
}

private var placedTiles = [PlacedTile]() {
    didSet {
        updateRows()
    }
}
```

A property observer on an array is fired whenever an object is added or removed. This is the function that updates the board:

```swift
private func updateRows() {
    rows = emptyBoard
    for placedTile in placedTiles {
        let occupiedSquares = placedTile.tile.squares().filter { $0.occupied == true }
        for tileSquare in occupiedSquares {
            let boardLocation = tileSquare.offsetBy(placedTile.square)
            rows[boardLocation.row][boardLocation.column] = true
        }
    }
}
```

Pretty simple - for each occupied square in a placed tile, mark the relevant board square as occupied.

Now to place or remove a tile:

```swift
public func positionTile(tile: Tile, atSquare square: Square) -> Bool {
    if !canPositionTile(tile, atSquare: square) {
        return false
    }
    placedTiles.append(PlacedTile(square: square, tile: tile))
    return true
}

public func removeTile(tile: Tile) -> Tile? {
	if let index = placedTiles.indexOf( { $0.tile === tile } ){
        placedTiles.removeAtIndex(index)
        return tile
    }
    return nil
}
```

I'm planning to use these return values to inform the UI once I get to that stage. 

While thinking about the updated board model and user interaction, it seemed that I was going to need a method to find the tile at a given board square. This will be used when processing touches on the board, and managed to give me some Bonus Swift Features™ to cram into this article after all. I was wondering if I'd get any. 

Here's the method I wrote:

```swift
public func tileAtSquare(square: Square) -> Tile? {
	for placedTile in placedTiles {
		let locationInTile = square.offsetBy(-placedTile.square)
		if placedTile.tile.squareWithinGrid(locationInTile) {
			let occupiedSquares = placedTile.tile.squares().filter { $0.occupied == true }
			for tileSquare in occupiedSquares {
				if tileSquare == locationInTile {
					return placedTile.tile
				}
			}
		}
	}
	return nil
}
```

This translates the board location to a tile location, checks that is anywhere within the tile, then checks if an occupied square in the tile matches up. 

The first bonus Swift feature is here: `square.offsetBy(-placedTile.square)`. To translate from a board location to a tile location, I need to do the reverse of the offset function. Rather than define a new function, since I could not think of a non-confusing pair of names for them, I instead defined the unary minus operator to work for a `Square`: 

```swift
public prefix func -(square: Square) -> Square {
    return Square(row: -square.row, column: -square.column)
}
```

I deliberately do not return a `Square` with an occupied property - anything returned here is for location calculations only, and not part of the board model. 

In a similar vein, for the second bonus Swift feature I defined `==` for `Square` to ignore the occupied property:

```swift
public func ==(left: Square, right: Square) -> Bool {
    return left.row == right.row && left.column == right.column
}
```

At the end of this stage of the project I have these feelings:

- The model and logic is pretty close to done
- I'm starting to feel itchy about the lack of unit tests. Playgrounds were good for quick visualisations and checks, but I can see some opportunities to refactor and I don't want to do that without the backup of tests.
- I'm starting to think about the views that are going to be involved - I would like to continue in playgrounds to use Live Views for those quick visualisation wins again. 

The next post will either be me migrating into an Xcode project and adding some tests, or creating some views in the playground, or a bit of both. It'd be nice to get a project going but it's also really good to be looking at things in a live view as you are writing code, and it would be interesting to see how far I can take that. 

[Here's the link](https://github.com/jrturton/Pentominoes/commit/fb3c3630019118bed6dae73f83815d71eb38839e) to the code as of the end of this post. 