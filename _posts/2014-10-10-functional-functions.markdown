---
layout: post
title: "Functional Functions"
date: 2014-10-10 19:33:28 +0100
comments: true
categories: 
---

*Functional programming for people who know nothing about functional programming by someone who knows next to nothing about functional programming : a series*

The introduction of Swift has brought with it a lot of talk about **functional programming**. I'd previously only heard the term whilst investigating things like Reactive Cocoa, and had not investigated too deeply because the documentation for that was _so_ full of arcane new terms[^1], but now it sounds dangerously like becoming mainstream.

[^1]: Futures? Promises? Signals?

As I've previously mentioned, I learnt to code on _the streets_, not some fancy computer school. It looks like I'm going to have to go back out there and work out what this new[^notnew] menace is.

<!--more-->

[^notnew]: Actually, invented at the same time as OOP, so not new at all

## What sort of programming am I doing now?

In everyday Objective-C, I don't write many functions, even when I probably should due to compiler optimisation and blah blah blah computer school. I mostly write methods. So am I currently doing *methodical programming*? Sounds nice, but although I am rather methodical, it's not a real term. In addition, the difference between functions and methods in Objective-C isn't applicable to other languages, so that's not it. For the rest of this article I'll just use the term "function", which you can think of as a method or a function in Objective-C. Note that, in Swift, there's only `func`. 

I'm definitely doing _Object-Oriented Programming_. By this I mean that I work with _objects_ that hold _data_ and present an _interface_ for other objects to interact with. 

Is functional programming a replacement for object-oriented programming? Can I write an iOS app using functional programming? I'm pretty sure the answer to these questions is "not entirely".

## What is functional programming?

[This quote](http://blog.sigfpe.com/2006/08/you-could-have-invented-monads-and.html) summed it up nicely for me:

>In a pure functional language, however, a function can only read what is supplied to it in its arguments and the only way it can have an effect on the world is through the values it returns.

This contrasts with OOP, where often you'd change or access an object's underlying state by calling functions defined on it. This is probably best illustrated with an example, of Genuine Production Code from my current project:

```objc
-(NSString *)stringFromMinutes:(NSInteger)minutes
{
    NSInteger hoursPart = minutes / 60;
    NSInteger minutesPart = minutes % 60;

    NSString *representation = nil;
    switch (self.recordingMode){
        case JRTHoursRecordingMinutes:
            representation = [NSString stringWithFormat:@"%d:%02d",hoursPart,minutesPart];
            break;
        case JRTHoursRecordingDecimal:
        {
            CGFloat fraction = minutesPart / 60.0f;
            representation = [NSString stringWithFormat:@"%.2f",hoursPart+fraction];
        }
            break;
    }

    return representation;
}
```

And, functionally:

```objc
-(NSString *)stringFromMinutes:(NSInteger)minutes recordingMode:(JRTRecordingMode)recordingMode
{
    NSInteger hoursPart = minutes / 60;
    NSInteger minutesPart = minutes % 60;

    NSString *representation = nil;
    switch (recordingMode){
        case JRTHoursRecordingMinutes:
            representation = [NSString stringWithFormat:@"%d:%02d",hoursPart,minutesPart];
            break;
        case JRTHoursRecordingDecimal:
        {
            CGFloat fraction = minutesPart / 60.0f;
            representation = [NSString stringWithFormat:@"%.2f",hoursPart+fraction];
        }
            break;
    }

    return representation;
}
```

The difference is small, but note that the functional version of the code stands completely alone:

- It doesn't depend on anything that isn't passed in
- It doesn't affect anything except the return value (though this is the case for both methods here)
- It doesn't even need an *instance*. There is no reference to instance variables. The functional version could be a class method or a standalone function, whereas the non-functional version has to be an instance method.  
- When called with the same parameters, it will always give the same results [^2]. 

The non-functional version might return a different value if other things were happening in your program. What if the recording mode was changed by some other  method? What if the method was being used to process a huge array of items on a background thread, would you then need to worry about synchronisation on the `self` or `recordingMode` values, or the value changing part way through?

[^2]: This is a simplified description of the term known as [idempotence](http://en.m.wikipedia.org/wiki/Idempotence). Functions that are idempotent are also [deterministic](http://en.m.wikipedia.org/wiki/Deterministic_algorithm). Functional programming comes with a lot of terminological baggage. 

It's entirely possible that you're already writing code in a functional style. It certainly lends itself well to test-driven development. Testing the functional version of the method above is trivial, for the non-functional version you need to ensure that the `self` instance is correctly configured. 

## First class functions

Another key part of functional programming is the idea that functions themselves can be treated just like other variables. You may have heard of this as functions being _first-class citizens_ in a particular language. 

This means you can pass a function as an argument to another function, or that a function can return a function. 

You're probably doing this already as well, particularly the first one. Blocks in Objective-C are functions, and can be passed around and stored just like variables. If you've ever passed in a completion block or done an animation using blocks, you've treated a function as a first-class citizen. 

In our example above, the function could be given to a view to use to convert model data into a user-facing version, like a lightweight version of `NSNumberFormatter`. 

## Functions as the unit of information 

In object-oriented programming, the object is what everything is built around. Objects hold information (often in the form of *other* objects) and do work with that information. In functional programming, particularly "pure" functional programming[^maths],  functions do all the work, and everybody tries their best *not* to hold any information, particularly not any information that can be changed by someone else when you're not looking. **Shared mutable state** is the bogeyman that functional programming is trying to avoid. 

[^maths]: Mathematicians are heavily into it, and mathematicians are heavily into pure things

## OK, I think I get the picture, why do I care?

Well, all the [cool](http://robnapier.net/llama-calculus) [kids](http://www.objc.io/books/) are talking about it, and you want to get involved, but that kind of thinking is why you can now ask Siri "What does the fox say?"[^fox]. There are concrete reasons why functional programming can benefit *you*:

[^fox]: Go ahead, I'll wait. Ask a few times. 

- Remember the bogeyman? Shared mutable state? Think back over the last ten bugs you introduced[^Introduced]. How many of them involved a property getting set to something unexpected, causing trouble down the line when something else depended on it? 

[^Introduced]: Or fixed, if I'm being generous

- What about your unit tests? Do you have a `setUp` method dozens of lines long to stand up an object and its data for testing? Have you not bothered with unit tests because they seem too complicated? A functional approach breaks your logic down into usable blocks with no dependencies, simplifying your unit tests. 

## Writing an iOS app using functional programming 

At this point, I wouldn't recommend it. Cocoa Touch is an object-oriented framework. You can't avoid a certain amount of object-oriented code. But we're here to make apps, not to worry about the academic purity of our code base. You *can* introduce functional elements to your existing code, most effectively at the model and business logic layer. Take a look at the app you're currently working on, and see if you can identify methods that rely on shared mutable state. Can these be rewritten? What benefits does this introduce to your codebase?

## What next?

This post has only scratched the surface, I realise. But read the subtitle again. This is as deep as I've scratched so far. Now that I've got the idea of functions as standalone entities, the next step for me is to explore what further possibilities await, particularly in the area of passing functions around. If I can figure it out, I'll post again.

