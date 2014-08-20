---
layout: post
title: "Understanding NSComparisonResult"
date: 2013-08-28 21:03
comments: false
categories: 
---
Readers of this post will hopefully fall into one of two categories:

- "Well duh, this is obvious! Why are you writing about this?"
- "OH! That suddenly makes sense!"

_(Hopefully, nobody will fall into the third category, which is "I still don't understand!")_

If you're in the first category, congratulations, but you're not really the target audience.

<!--more-->

Here's an exerpt of the documentation for `NSDate`'s `compare:` method:

>Returns an NSComparisonResult value that indicates the temporal ordering of the receiver and another given date.

>`- (NSComparisonResult)compare:(NSDate *)anotherDate`

>Returns:


>If the receiver and `anotherDate` are exactly equal to each other, `NSOrderedSame`

>If the receiver is later in time than `anotherDate`, `NSOrderedDescending`

>If the receiver is earlier in time than `anotherDate`, `NSOrderedAscending`.

`NSOrderedSame` is pretty straightforward, but I've had a mental block over `NSOrderedDescending` / `NSOrderedAscending` for years, up to the point of writing categories on `NSDate` to return booleans for `isEarlierThan:` and so forth. I could understand it, but never _remember_ it, or be able to read code featuring it without it being a little speed bump in my understanding. 

No longer! Here's how I remember it:

{% img /images/compare.png %}

As you read the `compare:` method, imagine a bar placed over the text. If the value on the left is larger than the one on the right, the bar is at an angle, sloping downwards (**descending**) in the reading direction, so the method returns `NSOrderedDescending`, and _vice versa_. As I said at the start, it seems obvious, but it wasn't to me until I visualised it like this, and usually that means it might also help somebody else. If it did, you're welcome.

<small>On a related note, my campaign to have the nonsensical "forward slash" and "back slash" renamed to the infinitely more sensible "up slash" and "down slash" needs all the help it can get! #rationalslashnames</small>




