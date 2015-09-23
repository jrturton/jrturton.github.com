--- 
date: 2015-09-23
layout: post
title: Dealing with web services
--- 

A major difference between making your own apps and making apps for others as a contractor or "professional" developer (a common step as people move from hobbyist or indie developers to perhaps more lucrative work) will be the web services you're asked to interact with. 

Nobody's going to pay you to make a Flickr or Twitter client, which is inconvenient because that's what most of the online examples seem to use. If you've never written code that interacts with a web API before, it's hard to approach these things with confidence, especially if you're expected to talk to the developers of that API to work out any issues, and if those APIs are being written and rewritten in parallel with the development of the app. 

In this article I'm going to go over the basics of your networking code. I'm going to assume you're working with an API from this century so we're talking about JSON and REST[^rest]. If you're hearing terms like SOAP from your web people then _run away_.

<!--more-->

[^rest]: I'm not going to define REST  to mean anything other than "things are at a URL", have that argument somewhere else

## Structure

You interact with a web API the same way you interact with the web when browsing: via a URL. Here's one:

https://api.staging.example.com/widgets/details?id=123456

And here's a breakdown of what each part means, and is called:

- **https://** - this is the **scheme**. For most services this will be `https`, but sometimes the staging server will run `http`. This makes it easier to debug and cheaper to run (since you don't need a certificate and the traffic is unencrypted). The production server should really be running https. As of iOS9, any requests to `http` endpoints will fail unless you whitelist them as App Transport Security exceptions.
- **api.staging.example.com** - this is the **domain**. You'd expect different values here to point to different servers (live, testing etc). When coupled with the scheme it is referred to as the **base URL**. Usually  this value is switched out depending on your build settings, so you can make builds targeting different servers without having to change your code. 
- **/widgets/** - this is the **path**. It should be a sensible, human-readable string or strings, separated by `/` characters, giving you some idea of the part of the API you're talking to. 
- **details** - this is the **endpoint**. This describes exactly what you're asking (or telling) the API - it's like the method name. 
- **?id=123456** - this is the **query string**. It's like the parameters of the method that you're calling. 

So the URL above is asking, **securely**, for the **details** of the **widget** with **id 123456** from the **staging** server at **example.com**. There's a lot of information in that URL!

## GET and POST

You'll use these two types of requests. Which one is at the discretion of the people writing the web services, but as a general rule, it's usually GET unless you're uploading some data, like an image file. It's possible (though not recommended) for a GET request to be used to update the server, so the distinction isn't black and white. 

## Building a URL

You've seen all the components of a typical web service URL. How do you go about building them in code? If you're thinking "_stick a bunch of strings together and create an `NSURL`_" then this is exactly the article for you. 

Using strings is a bad idea. If you have a format string like `?this=%@&that=%@`followed by a comma-delimited list of string variables, that is unreadable, unmaintainable code. Not only that, if your user has entered some of that data you're going to _break the internet_[^break] if they've entered an ampersand or an equals or one of the several other characters that URLs use for structure rather than content. 

[^break]: Well, your request won't work, anyway, but you can't be too careful. 

There is a _nice_ way to build up a URL. It's called `NSURLComponents`. This class breaks down all the various parts of a URL into properties, allowing you to read or assign them one by one. You can get the full URL out at any point:

```swift
// Create from base URL
let components = NSURLComponents(string: "http://api.example.com")!
// components.URL is http://api.example.com

//  Add path
components.path = "/widgets/details"
// components.URL is http://api.example.com/widgets/details

// Create query items
let queryItem = NSURLQueryItem(name: "id", value: "123456")
components.queryItems = [queryItem]
// components.URL is http://api.example.com/widgets/details?id=123456

let sillyItem = NSURLQueryItem(name: "escape", value: "?&-=")
components.queryItems = [queryItem, sillyItem]
//components.URL is http://api.example.com/widgets/details?id=123456&escape=?%26-%3D
```

## Making a request

I'm going to be bold here and say that you don't need to use a third party networking component. The Foundation networking classes are good enough for most purposes since the change to `NSURLSession`. Any wrappers around this do little more than obscure your knowledge of how those classes are doing their jobs.

I'm not going to go into the full details of how to use `NSURLSession` here, but this playground code should give you some idea of how simple it is: 

```swift
import Foundation
import XCPlayground

XCPSetExecutionShouldContinueIndefinitely(true)

let letters = NSURL(string: "http://private-8f863-commandshift.apiary-mock.com/letters")!

let request = NSMutableURLRequest(URL: letters)
let session = NSURLSession.sharedSession()
let task = session.dataTaskWithRequest(request) {
    (data, response, error) in
    
    if let error = error {
        error
        return
    }
    
    guard let response = response as? NSHTTPURLResponse  else { return }
    guard response.statusCode == 200 else {
        print("Status \(response.statusCode) returned")
        return
    }

    guard let data = data else { return }
    do {
        let json = try NSJSONSerialization.JSONObjectWithData(data, options: [])
        json
    } catch _ {
        print("Oops")
    }
    XCPSetExecutionShouldContinueIndefinitely(false)
}

task.resume()
```

You can substitute any URL (or build one with components) and immediately see the responses you get. I like to follow a four-stage process in these completion handlers: 

1. **Handle the `error` parameter**: This is returned when something went wrong meaning the connection attempt could not even be made.
2. **Handle the response code**: [Non-20x responses](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) can mean a variety of problems, and how you deal with them depends very much on the needs of your app.
3. **Handle server-level errors**: Often a completely valid JSON response will contain an error flag or error dictionary. You'll usually want to build your own error object out of this.
4. **Handle the valid JSON**: This is the "happy path". Here you'll be parsing out model objects and updating your app.

[None of this code should be in a view controller](http://www.cimgf.com/2015/09/21/massive-view-controllers/), by the way! Your view controllers should, at most, be asking some other object to go fetch stuff, and passing in a completion block with a single error and a single data parameter to be executed when done. But that's a whole other post. 

## Tools

The playground code above is a great way to explore a new API - you can quickly change the URL and see results instantly. But it's a bit of a mess, and you're changing the same thing over and over again rather than building up a good idea of the responses and calls you are working with. Enter [Paw](https://luckymarmot.com/paw), an indispensable and very nicely done app (no affiliation, I am a happily paying customer) which allows you to build up a set of API calls and tweak them with variables, environments, and view the responses. It will even, with plugins, generate the Objective-C or Swift code just like that shown above. 

Maybe your API hasn't been written yet. You could mess about with locally stored JSON files and proxies and so forth to make up for that, or you could use a service like [Apiary](https://apiary.io/) which allows you to stand up quick responses to calls - all you have to do when you're done is switch out the domain. I'm using Apiary for the example above. 

The next important thing about writing your networking code is to use unit tests. Having to run through your UI tapping buttons and so on to see if you've done your networking right is silly, and you need to stop it. Asynchronous unit tests are a doddle now that `XCTestExpectation` exists, and since you're not putting your network code is in a view controller it's all much more testable. 

Once your code is up and running you may have further problems. Maybe the API isn't ready, or you want to get a specific response, or you just want to see the exact responses coming back at a specific time for a specific request, or you want to see what happens if you get a 404 or an error or a timeout. It's time for [Charles](http://www.charlesproxy.com) which, a bit like AppCode, is so useful and powerful you can overlook how hideous it looks. (Again, no affiliation, happily paying customer!) Charles can do all of these things and more, by acting as a proxy and showing you all the traffic passing between the iOS simulator, or your device, and the internet (unless that traffic is correctly using certificate pinning and is secure!). You can replace calls with local text files, breakpoint on requests or responses and edit the contents, rewrite data according to your own rules. I probably know how to use about 10% of its power and I'm getting my money's worth. You know those ridiculous high scores on Game Center? Pretty sure that's Charles. Not me though, I'm all natural. 
