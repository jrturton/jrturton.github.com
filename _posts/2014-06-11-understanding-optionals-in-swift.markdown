---
layout: post
title: "Understanding Optionals in Swift"
date: 2014-06-11 21:46
comments: false
categories: 
---

## What are they for?
In Objective-C, you can safely send a message to `nil`, which will return something treated as `nil`, or `NO`, or `0` (this variety is part of the reason Swift has optionals, of which more later). This is a useful and powerful feature, but it only applies to Objective-C objects. There's no universal way to deal with absent or no-value types such as integers, floats or Booleans. Things like `NSNotFound`, `NSIntegerMax`,` -1` or `0` are used to represent "no value" in these cases. 

Optionals are Swift's way of unifying the representation of Nothingness. By using them, we lose some of the ease and flexibility of nil messaging, but gain compile-time checking, safety and a consistent way of dealing with the same problem, regardless of variable type. 

<!--more-->

## How do they work?
Here's an optional integer declaration:

```swift
var perhapsInt: Int?
```

Here's the important thing you need to understand:

**An optional declaration doesn't say "this is an Int, which is optional". It says, "this is an Optional, which may or may not hold an Int"**

Go back and read that again.

**An optional declaration doesn't say "this is an Int, which is optional". It says, "this is an Optional, which may or may not hold an Int"**

(I wrote it twice, in case you didn't read the middle instruction)

You're not dealing with a specially declared `Int` variable. An Optional is a type all of its own, actually one of Swift's new super-powered enums. It has two possible values, `None` and `Some(T)`, where `T` is an associated value of the correct type.

If there is no value, the optional has the value of `None`. If there is a value, the optional has the value of `Some` and an _associated_ value of the `Int`, or whatever other type was assigned. It's this structure that allows any type to be represented by an Optional, and it separates the representation of Nothing from the representation of any value of Something. If you've ever implemented a separate `BOOL` to track if a value has actually been set or if it's just in a default state, Optionals are your friend. 

Once you understand that Optionals are a distinct type, then the process of "unwrapping" values and all the compile time checks finally make sense. When something requires an `Int`, you can't pass it a different type. You either force unwrap the optional using the `!` operator, or you can pull it out into a temporary constant using optional binding.

## How do I use them?

An Optional is declared by using a `?` after the type:

```swift
var perhapsInt : Int?
```

This is automatically set to a `.None` value.

You can check if an Optional contains a value by comparing it to `nil`:

```swift
var perhapsInt: Int?
perhapsInt = 1
if perhapsInt != nil {
  let intString = String(perhapsInt!)
  println(intString)
}
```

Removing the `!` is a compile time error, because you're not unwrapping the Optional. Unwrapping the Optional when it has a `None` value is a run-time error:

```swift
var perhapsInt: Int?
let intString = String(perhapsInt!)
```

Instead of checking and then force unwrapping the Optional, you can combine the two with what's known as _Optional Binding_:

```swift
var perhapsInt: Int?
perhapsInt = 1
if let actualInt = perhapsInt {
  println(actualInt)
}
```

You can also use the (slightly more compact) _nil coalescing operator_ to supply a default value if the optional doesn't have one:

```swift
var perhapsInt: Int?
let definiteInt = perhapsInt ?? 2
println(definiteInt) // prints 2
perhapsInt = 3
let anotherInt = perhapsInt ?? 4
println(anotherInt) // prints 3
```

You can set an optional back to `.None` by assigning `nil` :

```swift
perhapsInt = nil
```

This is all covered in depth in Apple's language guide. The key thing to understand (which isn't really covered until the end, and who reads to the end of books nowadays?) is the nature of an Optional and the fact that it is **a separate type, rather than a special annotation of a standard type**. Suddenly, this initially mind-blowing code from the Intermediate Swift video (session 403), makes sense:

```swift
func stateFromPlist(list: Dictionary<String, AnyObject>) -> State? {
	switch (list["name"], list["population"], list["abbr"]) { 
		case (.Some(let listName as NSString),
		.Some(let pop as NSNumber),
		.Some(let abbr as NSString))
		where abbr.length == 2:
			return State(name: listName, population: pop, abbr: abbr)
		default:
			return nil
	}
}
```

What is happening here is the following:

```swift
func stateFromPlist(list: Dictionary<String, AnyObject>) -> State?
```

Here's the function declaration. It takes a `Dictionary`, with `String` Keys and `AnyObject` values. It returns an Optional `State`.

```swift
switch (list["name"], list["population"], list["abbr"])
```

Here the dictionary is unpacked into a tuple containing three Optionals - dictionary subscripts return Optionals, since the dictionary might not contain a value. Because switch statements are superpowered in Swift, you can then match on a complex set of criteria:

```swift
case (.Some(let listName as NSString),
        .Some(let pop as NSNumber),
        .Some(let abbr as NSString))
```

Remember, Optionals are enums. Here, the switch statement is checking for the `.Some` case and associated value for each part of the tuple. If a value is there, you can use your knowledge of the structure to extract the strings and numbers from the optionals into constants of the appropriate type. If any of the values are missing, the case will not be executed.

```swift
where abbr.length == 2:
```	

The authors are simply showing off at this point. Still within the case statement, having extracted values from optionals, they are showing that you can add _further_ logic to each case. 

Only if all of the entries from the dictionary contain values, and if the abbreviation is two characters long, will a valid State object be returned. Otherwise, the dictionary is rejected as invalid and `nil` (meaning Optional value `.None`) will be returned.

This one function shows what great promise there is in Swift. The equivalent Objective-C code would have been something like the following, and would become immensely more complex with additional validation or processing rules:

```objc
-(State*)stateFromPlist:(NSDictionary*)plist
{
	NSString *name = plist[@"name"];
	NSNumber *population = plist[@"population"];
	NSString *abbreviation = plist[@"abbr"];
	
	if (name == [NSNull null] || population == [NSNull null] || abbreviation == [NSNull null])
		return nil;
	
	if (name && population && abbreviation && [abbreviation length] == 2)
		return [State stateWithName:name population:population abbreviation:abbreviation];
	else
		return nil;
}
```

I don't know about you, but having written my fair share of code like the above, I'm looking forward to a Swift world.
