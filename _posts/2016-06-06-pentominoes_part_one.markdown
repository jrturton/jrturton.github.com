--- 
date: 2016-06-06
layout: post
title: "Pentominoes, part one: Tiles"
--- 
My daughter and I are members of [At-Bristol](https://www.at-bristol.org.uk), a most excellent interactive science centre in Bristol. On a recent visit she was captivated by a "Pentominoes" puzzle. Pentominoes are the twelve possible tile shapes you can make using five squares, joined at their edges. They are a little like tetris shapes, but with five squares instead of four.

<!--more-->

Here are the possible tile shapes and the two common notations for them:

<p><a href="https://commons.wikimedia.org/wiki/File:Pentomino_Naming_Conventions.svg#/media/File:Pentomino_Naming_Conventions.svg"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/69/Pentomino_Naming_Conventions.svg/1200px-Pentomino_Naming_Conventions.svg.png" alt="Pentomino Naming Conventions.svg"></a><br>By <a href="//commons.wikimedia.org/wiki/User:Nonenmac" title="User:Nonenmac">R. A. Nonenmacher</a> - <span class="int-own-work" lang="en">Own work</span>, <a href="http://www.gnu.org/copyleft/fdl.html" title="GNU Free Documentation License">GFDL</a>, https://commons.wikimedia.org/w/index.php?curid=4412149</p>

---

The puzzle consisted of wooden versions of the possible shapes and a six-by-ten frame to fit them in. Although my daughter didn't manage to solve the puzzle, she displayed extraordinary tenacity and technique, and I'm sure she would have got there given time. 

I promised to make her a version of the puzzle to play on her iPad instead. I've done a few little projects like this for her (hopefully one day, _with_ her) and thought this one might make an interesting "development diary" type of post. I've no doubt made some decisions you disagree with. I've no doubt made some decisions _I_ will disagree with, as I move through the project, but that's the point of this series - code doesn't spring perfectly formed onto the screen. It's written, examined, tested and re-written until it's good enough. I hope to document that process. 

Note that I said "good enough", not "perfect". You have to ship. I'm also hoping that the double commitments of writing about the process and the fact I promised her will help to drive this project along. 

## First thing: The tiles

I always like to get the data model and "business rules" defined at the start of a project. UI comes at the end for me, since it is so often driven by data. The first part to tackle was the tiles. 

Here are the rules about tiles: 

- The tiles can rotate on the z-axis in 90 degree increments. 
- The tile has a specific pattern of squares that cannot change, except via rotation
- There are 12 possible tile patterns, only one of each is permitted in the game

Which lead me to these code considerations:

Each `Tile` in the model represents a _real, physical_ object in the game, and we have the ability to change the internal state of the tile by rotating it. Therefore `Tile` should be a `class`, not a `struct`:

```swift
public class Tile {
}
```

Any possible rotation of the tile can fit into a five-by-five grid, so the tile will contain a two-dimensional matrix of `Bool` to represent the grid, where `true` represents the presence of a part of the tile, and `false` represents the absence:

```swift
private (set) public var rows: [[Bool]]
```

It's going to be handy to be able to access a particular location using a nice subscript notation, such as `tile[1][1]`:

```swift
public subscript(row: Int) -> [Bool] {
	get {
		return rows[row]
	}
}
```

`true` and `false` don't look very good when you're trying to debug or examine your work, so `Tile` should be `CustomStringConvertible` so that it can be `print`ed usefully in a playground. Here `#` will represent a filled space and `_` an empty one:

```swift
extension Tile: CustomStringConvertible {
    public var description: String {
        let descriptions: [String] = rows.map { $0.reduce("") { $0 + ($1 ? "#" : "_") } }
        return descriptions.joinWithSeparator("\n")
    }
}
```

The 12 possible tile patterns should be represented by an `enum`. Tiles should be created with `init(shape: O)`. I want to be able to create these in a loop, so I'm using an integer raw value:

```swift
public enum Shapes: Int {
    case O = 0
    case P = 1
    case Q = 2
    case R = 3
    case S = 4
    case T = 5
    case U = 6
    case V = 7
    case W = 8
    case X = 9
    case Y = 10
    case Z = 11
}
```

To create the two-dimensional array to represent the tiles but also to make the code nice and readable, I decided to do a reversal of the `description` method and have a `[String]` calculated variable which could be used to create the tiles:

```swift
var stringMap: [String] {
        switch self {
        case O:
            return [
                "__#__",
                "__#__",
                "__#__",
                "__#__",
                "__#__"
            ]
       ...
```

The initialiser then looks like this:

```swift
public init(shape: Shapes) {
    rows = shape.stringMap.map {
        return $0.characters.map { $0 == "#" }
    }
}
```

And of course, I was going to get this up and running in a playground first. Why on earth not? This meant I could instantly see if I'd screwed anything up: 

```swift
let tile = Tile(shape: .O)
print(tile)
```

Printing is a bit tedious, though, and when I add the `Tile` inline in the playground I get a fairly useless view with an outline list of unlabelled variables. That can be solved by making it `CustomPlaygroundQuickLookable`: 

```swift
extension Tile: CustomPlaygroundQuickLookable {
    public func customPlaygroundQuickLook() -> PlaygroundQuickLook {
        return PlaygroundQuickLook(reflecting: description)
    }
}
```

Now the tile shows up clearly inline in the playground!

The final part to address about tiles was the rotation action. I set about solving this complex problem the way all programmers do - I searched for "Rotate 2D array" on Stack Overflow, which led me [here](http://stackoverflow.com/a/8664879/852828), and then [here](http://stackoverflow.com/a/32922962/852828). Putting this information together gave me a rotation method (`transpose` was copied wholesale from Abizer's answer, so not reproduced here): 

```swift
public func rotate(clockwise: Bool) {  
    if !clockwise {
        reverseRows()
    }
    rows = transpose(rows)
    if clockwise {
        reverseRows()
    }
}
private func reverseRows() {
    rows = rows.map { $0.reverse() }
}
```

Thanks to the quick looking in the playground it was easy to check that this was working as intended. That was tiles all done. The [next part of the series](http://commandshift.co.uk/blog/2016/06/08/pentominoes-part-two/) will talk about the board.