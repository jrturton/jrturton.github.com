--- 
date: 2016-06-08
layout: post
title: "Pentominoes, part two: Board"
--- 

In [part one](http://commandshift.co.uk/blog/2016/06/06/pentominoes_part_one/) I went through the process of building the Tile model objects for my Pentominoes puzzle app. In this part I will talk about making the Board, and what I learned about protocols and default implementations in the process. 

<!--more-->

The board, like the tiles, can be represented with a two-dimensional array of `Bool`s. Also, like the tiles, it's going to be nice to be able to view the board as a string and access locations in there via subscripting. Hmmm. 

At this point I had three choices. 

I could reimplement the same logic in the `Board` and `Tile` classes, which is silly. I could make both classes inherit from a superclass, but that didn't feel right. Instead I decided to create a protocol.

## Protocols and default implementations 

```swift
public protocol PlayingGrid {
    var rows: [[Bool]] { get }
    subscript(row: Int) -> [Bool] { get }
}
```

The subscript implementation can be done in an extension: 

```swift
extension PlayingGrid {
    public subscript(row: Int) -> [Bool] {
        get {
            return rows[row]
        }
    }
}
```

The corresponding code can be removed from the `Tile` class, which is now declared like so:

```swift
public class Tile: PlayingGrid {
```

I then tried to make a default implementation of the `CustomStringConvertible` protocol method in an extension:

```swift
extension PlayingGrid: CustomStringConvertible {
```

This gives the error "Extension of protocol cannot have an inheritance clause". It turns out you can't do it like this. However, what I wanted is still possible. I had to define the inheritance in the main declaration:

```swift
public protocol PlayingGrid: CustomStringConvertible {
```

Then in the extension, it can be implemented. I rewrote the implementation so it didn't look so much like robot vomit:

```swift
extension Bool {
    var gridCharacter: String {
        return self ? "#" : "_"
    }
}

extension PlayingGrid where Self: CustomStringConvertible {
    public var description: String {
        let descriptions : [String] = rows.map { row in
            row.reduce("") { string, gridValue in
                string + gridValue.gridCharacter 
            }
        }
        return descriptions.joinWithSeparator("\n")
    }
}
```

I then had to add similar code to make `PlayingGrid` `CustomPlaygroundQuickLookable` as well. First, amend the protocol declaration:

```swift
public protocol PlayingGrid: CustomStringConvertible, CustomPlaygroundQuickLookable {
```

Then, amend the constraints on the extension (they need to be in the same extension, because the quick look depends on the description):

```swift
extension PlayingGrid where Self: protocol<CustomStringConvertible, CustomPlaygroundQuickLookable> {
```

(the `protocol< >` syntax was pointed out to me by the wonderful [@jessyMeow](https://twitter.com/jessyMeow))

Within the extension, I used the same implementation of `customPlaygroundQuickLook()` from `Tile`, removing the code from that class. 

Having done all that to split the shared functionality out from `Tile`, it was time to create `Board`. 

## Making the Board

According to the [Wikipedia entry](https://en.wikipedia.org/wiki/Pentomino) there are four different board sizes which can take all of the tiles with no gaps. It's not possible to represent this nicely in an enum, because the raw value of an enum can't be a tuple. Instead, a struct with static values works:

```swift
public struct Size {
    let height: Int
    let width: Int
    
    public static let SixByTen = Size(height: 6, width: 10)
    public static let FiveByTwelve = Size(height: 5, width: 12)
    public static let FourByFifteen = Size(height: 4, width: 15)
    public static let ThreeByTwenty = Size(height: 3, width: 20)
}
```

I use the height dimension first simply because that's how it is shown in the Wikipedia page and I wanted to use common nomenclature.

The board's initializer needs to take a `Size`, now we've defined it. To allow for positioning of any type or rotation of tile anywhere in the board, I will add four blocks of padding in each direction around the empty board. That way a tile can be placed anywhere on the "board" with a minimum of one of its squares actually on the playing surface. 

This is probably the sort of code that could be written as an unreadable one-liner in Swift, but that's not how I roll.

```swift
public init(size: Size) {
    // Extend by four "occupied" positions in every direction
    let paddingHorizontal = [Bool].init(count: 4, repeatedValue: true)
    let paddingVertical = [Bool].init(count: 8 + size.width, repeatedValue: true)
    let fullPaddingVertical = [[Bool]].init(count: 4, repeatedValue: paddingVertical)
    let emptyRow = [Bool].init(count: size.width, repeatedValue: false)
    rows = fullPaddingVertical
    for _ in 0..<size.height {
        rows += [paddingHorizontal + emptyRow + paddingHorizontal]
    }
    rows += fullPaddingVertical
}
```

I feel better about the code after these changes. I think its a sensible use of a protocol, I learned lots of things about default implementations and protocol constraints, and the actual `Board` and `Tile` implementations are currently very light. In the next part I will start to work on some game logic - placing tiles on the board. 


