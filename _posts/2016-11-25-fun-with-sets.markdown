--- 
date: 2016-11-25
layout: post
title: "Fun with Sets"
--- 

A Set is a collection of unique members. The uniqueness of those members, in a Swift `Set`, is determined by the `hashValue`. This means you can only store values in a set that conform to `Hashable`. 

To combine two sets, you use the `union` method. Consider the following groups of numbers: 

```swift
let someInts = Set([1, 2, 3])
let someMoreInts = Set([3, 4, 5])
let allTehInts = someInts.union(someMoreInts)
```

`allTehInts` will contain 1, 2, 3, 4 and 5. 3 was in both sets, but sets can only contain unique values, so it doesn't get included. 

_Which_ 3 is contained in `allTehInts`? The one from `someInts`, or the one from `someMoreInts`? THE ANSWER WILL SHOCK YOU! Or, confuse you, if you've just migrated to Swift 3.

<!--more-->

You may well be saying, "I don't care which one is included. 3 is 3", and you'd have a point. But in a recent project I was using sets for a slightly more complex purpose. 

To relieve the tedium of mapping JSON to Core Data models, there was code that creates a set of default "attribute mappings", which assume a 1:1 relationship between the entity attribute name and the field name from JSON. For each entity you can then specify additional mappings for when the names don't match up or you need to use a value transformer. The final set of mappings that gets used is a union of the default mappings and the specialised mappings. 

The hash value of the attribute mapping was based solely on the "remote key path" of the mapping - the name of the field in the JSON response. This makes sense, because you don't want to map the same remote field to multiple attributes. 

To demonstrate the principle without getting sidetracked, here is a simplified example using the reliable old `Person` struct: 

```swift
struct Person {
    let firstName: String
    let lastName: String
}

extension Person: Equatable {}

func == (lhs:Person, rhs: Person) -> Bool {
    return lhs.firstName == rhs.firstName
}

extension Person: Hashable {
    var hashValue: Int {
        return self.firstName.hashValue
    }
}
```

The `Person` has a first and last name, but the uniqueness is measured only on the first name. 

Here are three people with unimaginatively similar names: 

```swift
let person = Person(firstName: "Bob", lastName: "Bobson")
let person2 = Person(firstName: "Bob", lastName: "Jobson")
let person3 = Person(firstName: "Rob", lastName: "Bobson")
```

Let's put them into two sets: 

```swift
let set1 = Set([person, person3])
let set2 = Set([person2])
```

According to the rules above, `person`, Bob Bobson, and `person2`, Bob Jobson, are identical - they have the same first name. 

Create a single set which is a `union` of the two: 

```swift
let union = set1.union(set2)
```

In Swift 2.2, `person2` gets included in `union`, and `person` is dumped. Reverse the order: 

```swift
let union = set2.union(set1)
```

`person` is included in the final set this time. In the project, the final set of mappings was a union of the default and specific mappings, which meant that any specific mappings replaced the defaults. 

_This behaviour is reversed in Swift 3_. Members of the _first_ set are kept when performing a union operation. This means that most of the specific mappings are dropped when performing the union, because there is often a default mapping with the same name. 

A simple fix to reverse the order of the union, but it took an awful lot of head-scratching to find out what was happening and why. The Swift 3 documentation is specific about which members will be included in the event of a match, the earlier documentation is not. 