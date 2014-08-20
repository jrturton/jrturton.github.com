---
layout: post
title: "Bug Hunting"
date: 2013-11-17 20:07
comments: false
categories: 
---
The client had raised a bug. They'd marked it as high priority, as clients often do. If you take the following steps:

* Log in to the app
* Do some activities
* Switch to the settings app
* Set the date manually to yesterday
* Switch back to the app
* Log out
* Log back in again

The app would enter a strange, flickering hinterland for a few moments before crashing. The above steps might seem like a strange use case, but it was *interesting* and could possibly have real world relevance during daylight saving switchovers. 

In any case, once I can reproduce a bug, I find it very hard to resist fixing it. 

<!--more-->

Turning on Exception breakpoints is always the first step when fixing bugs. In fact, there aren't many cases where you *don't* want exception breakpoints turned on. This means you usually get the program stopping right at the problem instead of kicking you back to `main.m`. 

The crash came from a `dealloc` method, where an observer was being removed, leading to a segmentation fault. The object being deallocated was a view controller, and the call was not on the main thread, but during cleanup of a block from a networking call. 

A few Stack Overflow searches later and I'd convinced myself that the problem was down to not using weak references in my web service completion blocks, and that observers should probably be removed on the thread they were added on.

Sort a few of those out, re-run and get ready to bask in the glow of... Oh. Still not working, but the crash has moved to a _different_ thread, still in the same method. 

Debugging is a process of calling yourself an idiot for not fixing the problem until you call yourself an idiot for causing it in the first place. 

Stepping back from the thread issue, why are we even deallocating this object? It's supposed to be around the whole time for a logged in session. I'm an idiot. That's why I hadn't used weak references in the first place. I added a few logging breakpoints to work out where this view controller is being created or destroyed and re-run.

Logging breakpoints are another great tool when bug fixing. You can add them to a running app and get output straight away without having to rebuild, and they don't clutter up your version control system or get left in release code accidentally like NSLog statements.

    creating
    dealloc
    creating
    dealloc
    creating
    dealloc
    creating
    dealloc
    ...
   
Oh dear. That would also explain the flickering part of the flickering hinterland. A few more judiciously placed breakpoints (debugging completion block or delegate-based code is hard, because you never get a call stack) revealed that the app was logging itself out, then in, then out again, and so on. 

Why is it doing this? And why would it only do it if we've manually changed the date? The app only logs itself out if it has an invalid access token for the web services, and fails to obtain a new one. And the app logs itself in fine when it is launched, so what's going on?

It was time to fire up [Charles](http://www.charlesproxy.com/), an indispensable tool that I've only recently been introduced to. You can view all of your web requests, set breakpoints on them, edit the request or the response, record and re-play requests - basically, if you're doing anything that involves web services, you need Charles. I'm a total beginner at it but I work with some real experts. 

Charles gave me some confusing news. The login request wasn't happening, and yet the app was logging in, and making the rest of its requests just fine. Further breakpoints establish that the service is being called and a response is being returned, but without any network activity. This sounds like a caching issue, and indeed, the response that is coming back is the same every time, meaning the access token being used is no longer valid, causing the cycle of logging in and logging out again. The response comes back with (according to Charles) cache control of `private max-age 0`. 

There is a discussion to be had about whether this is the correct cache policy for this response to contain - a `no-cache` would probably be more appropriate. What appears to be happening is that, since the cached response was made before the date was changed, the age of the cached response is *less* than 0, so the rules are being followed. 

A swift bit of rearrangement of my web service classes (I'm using [AFNetworking](https://github.com/AFNetworking/AFNetworking), with a wrapper class per endpoint) gives me access to the `NSMutableURLRequest` being generated each time, and for the login service, I can now force the request to ignore any caching information. Hey presto! Fixed. On to the next high priority issue...
