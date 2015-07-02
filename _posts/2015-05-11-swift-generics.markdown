---
layout: post
title: "Swift Generics"
date: 2015-05-11 14:29:46 +0100
comments: false
categories: 
---

"Don't Repeat Yourself" is a solid bit of programming wisdom. It's so common it has been made into an acronym, DRY, which means that people get to make "jokes" about DRYing out their codebase. 

If you write a bit of code that does something, and elsewhere in your program you need to do the same or a very similar thing, don't copy and paste the bit of code. Make it accessible from both places, and make the minimum set of changes needed to make it reusable. 

["Boilerplate" code](http://en.wikipedia.org/wiki/Boilerplate_code) refers to code that has to be included regularly and without much alteration to achieve basic functionality. There are plenty of frequent, common operations that require a lot of boilerplate code. The boilerplate code obscures what you actually want to do and makes the program harder to read. A good example is inserting managed objects into a context. For each type of managed object, the code looks almost exactly the same, but different types are involved. 

I'm going to discuss some of the boilerplate code encountered in Core Data, and how you can use Swift generics to DRY out some of these operations. First, a few words about generics.

<!--more-->

## Generics in functions

Generics in functions allow code to be flexible about types at writing time, but strict at compile time. They work by having  type tokens for each non-specified type involved in the function. The type tokens can be completely generic, or have protocol restrictions on them. For example, for kicking off a [nice web service request](http://commandshift.co.uk/blog/2014/12/28/nice-web-services-swift-edition/), I have this function:

```swift
func startRequest<C : WebServiceConfiguration, R : WebServiceResponse>(configuration: C, completion:(response:R?, error:NSError?)->Void)
```

Don't panic! Here's a breakdown:

- `startRequest` - this is the name of the function
- `<C : WebServiceConfiguration, R: WebServiceResponse>` - these are the _type tokens_. Here we have two generic types, C and R. C has to conform to the `WebServiceConfiguration` protocol, and R has to conform to the `WebServiceResponse` protocol.
- `configuration: C` - the first parameter is of type C, which as we've noted above must conform to the `WebServiceConfiguration` protocol.
- `completion:(response:R?, error:NSError?)->Void` - the second paramater is a closure, which takes an optional R (conforming to `WebServiceResponse` and an optional error. 

The type tokens don't have to be single letters, though they are in nearly every example I've seen. You can use longer terms, but then the whole function definition can get quite long.

You call the function like so:

```swift
let loginConfiguration = LoginConfiguration(username: username, password:password)
startRequest(loginConfiguration) {
  (response : LoginResponse?, error) in
  // Do stuff...
}
```

The compiler can tell from this code that we're calling `startRequest` using a `LoginConfiguration` for the C type token, and a `LoginResponse` for the R type token. Clever stuff! 

`startRequest` is the _only_ function you ever call when kicking off a web request. There's no need for a separate function for each API endpoint!

## Generics in container types

As well as function arguments, generics are also used for container types - the most common example being the Optional enum, but also Swift's typed arrays and dictionaries. Those aren't as interesting for this discussion, though.

## Core Data - inserting objects

To insert a new entity in your context, the following code is required: 

```swift
let person = NSEntityDescription.insertNewObjectForEntityForName("Person", inManagedObjectContext: context) as! Person
person.name = "Bob"
...
```

Yes, it's only one line, but it's quite a long one, there's a constant string in there, and you need to force cast to a Person because `insertNewObject...` returns `AnyObject`. 

I find it's useful to think of how a function might look before I get around to implementing it. For inserting an object, I have the following requirements:

- We need a managed object context
- I don't want to have to write the class of the inserted object more than once
- It must be obvious what is happening
- No type casting required.

I'd like to do this:

```swift
let person = context.insert(Person)
```

First, get rid of the constant string with an extension on `NSManagedObject`:

```swift
extension NSManagedObject {    
  class var entityName : String {
    let components = NSStringFromClass(self).componentsSeparatedByString(".")
    return components[1]
  }
}
```

This assumes your entity names always match your class names. The processing is to remove the module name from the start. If you have any entities that don't follow this rule, you'd have to override this method in the appropriate `NSManagedObject` subclass.

Now, you can write the following extension for `NSManagedObjectContext`:

```swift
extension NSManagedObjectContext {
    
    func insert<T : NSManagedObject>(entity: T.Type) -> T {
        let entityName = entity.entityName
        return NSEntityDescription.insertNewObjectForEntityForName(entityName, inManagedObjectContext:self) as! T
    }
    
}
```

There's a small amount of magic happening here. The type token, T, has a specifier saying that the type used must be a subclass of `NSManagedObject`. The last line of the function is pretty much the same as the original insert code used above, using the forced cast. But in the function definition, I'm using `T.Type`, not `T`. This means the function is expecting the `entity` parameter to be a _type_, rather than an instance of an entity. 

Generics have allowed me to write a function where I pass in a class, which is guaranteed to return an instance of that class. 

As desired, this can be called like this:

```swift
let person = context.insert(Person)
```

But there's more. To support unit testing or development of an app, it is extremely useful to be able to quickly produce a set of sample data. For example, I'd like to create 10 `Person` objects like this:

```swift
context.createTestObjects(10) {
  (person: Person, index) in
  person.name = "Test Person \(index)"
}
```

That's pretty straightforward as well. As an extension on `NSManagedObjectContext`:

```swift
func createTestObjects<T: NSManagedObject>(count : Int, configuration : (object: T, index: Int) -> Void){
  for testNumber in 1...count {
    let object = self.insert(T.self)
    configuration(object: object, index: testNumber)
  }
}
```

Here, the type is inferred from the parameter list in the configuration closure. In the last app I made this was probably the most useful method in there, allowing me to create a reasonable set of test data almost instantly for unit testing and building the UI. 

## Core Data - fetching objects

Here's how you fetch objects from a context:

```swift
let request = NSFetchRequest(entityName: "Person")
var error : NSError? = nil
let people = context.executeFetchRequest(request, error: &error) as? [Person]
if let error = error {
  // Better do something about that error...
} else {
  // Do something with all those people....
}
```

The error checking, while not terribly useful for users of your app, is absolutely essential for you, the developer. But if you have to write all that boilerplate for every fetch request in your app, are you really going to be checking them all? 

First of all, there's that hardcoded string for the entity name. We've already solved that problem when inserting objects. The same applies again, in an `NSManagedObject` extension:

```swift
class func fetchRequest() -> NSFetchRequest {
  return NSFetchRequest(entityName:self.entityName)
}

class func fetchRequestWithKey(key: String, ascending: Bool = true) -> NSFetchRequest {
  let request = fetchRequest()
  request.sortDescriptors = [NSSortDescriptor(key: key, ascending: ascending)]
  return request
}
```

The second version allows you to add some sort descriptors by supplying the key and an ascending flag, which is often useful. 

The fetch operation itself can be made generic. The context performs the fetch so you can add the following extension to `NSManagedObjectContext`:

```swift
func fetchAll<T : NSManagedObject>(entity: T.Type, key:String? = nil, ascending:Bool = true) -> [T] {
  let fetchRequest : NSFetchRequest = {
    if let key = key {
      return entity.fetchRequestWithKey(key, ascending: ascending)
    } else {
      return entity.fetchRequest()
    }
  }()
  
  var error : NSError? = nil
  if let results = executeFetchRequest(fetchRequest, error: &error) as? [T] {
    return results
  }
  return [T]()
}
```

You can call this like so:

```swift
let people = context.fetchAll(Person)
```

Or, for ordering:

```swift
let people = context.fetchAll(Person.self, key:"name", ascending:true)
```

Why it has to be `Person.self` for the second example I am not sure. If you know why, please tell me! 

## Summary

- If you find yourself writing the same code, where only the types are different, consider generics.
- Generics are powered by type inference, and the types can be inferred from arguments passed in, the variable a function's return value is assigned to, or more complex ways such as the parameter list to a block.
- Generics can be useful for reducing boilerplate code, but always bear in mind future readers of your code (including yourself). Don't make existing APIs unrecognisable and force maintainers to learn a whole new one without good reason.
