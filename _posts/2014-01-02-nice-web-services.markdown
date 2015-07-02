---
layout: post
title: "Nice web services"
date: 2014-01-02 21:29
comments: false
categories: 
---
I don't like "clever" code. Debugging code is much harder than writing it, so if you write the cleverest code you can, you'll never be smart enough to debug it. 

I do, however, like nice code. Nice is an offensively inoffensive word, and pretty hard to define, but you know it when you see it. I therefore humbly present a nice way of writing web service consumer classes for your app. The niceness is derived from:

* Limited responsibility
* Simple interfaces
* Neat encapsulation
* Minimising dependencies
* Easy implementation

I came up with this whilst working on a recent project, and colleagues have since tried the same design and liked it, so it's probably good enough to post here. I make no claims to its originality, but it doesn't resemble any other networking code I've encountered. 

<!--more-->

## Interface
A web service consumer class should do two things: send a request over the web, and give back the response. Users of the class shouldn't know or care how it is doing this, and they should be able to pass parameters in and read responses out **without having to know anything about how the web request or response is structured**. So, to take the tired old example of a log in service, it is *nice* to be able to do this: 

```objc
JRTLoginService *loginService = [JRTLoginService new];
loginService.username = @"CommandShift";
loginService.password = @"1234";
[loginService startWithCompletion:^(JRTLoginResponse *response, NSError *error){
	if (error)
	{
		// Do something with the error
		return;
	}
	if (response)
	{
		// Do something with the response object
	}
}];
```

All web service parameters are exposed as properties on the web service class. All response values are properties on a *response object*. 

The response object is a dedicated object used for storage of the response from this service. This has the following advantages over returning a dictionary from the JSON parser:

* All `NSNull` responses can be removed - hands up if your app has crashed because a null object has been returned in a JSON dictionary for a value you expected to be a string? Ever wondered why is this not an option on `NSJSONSerialization`? [It is now!](https://github.com/jrturton/NSJSONSerialization-NSNullRemoval)
* All `NSNumbers` are converted back to the appropriate integer, float or boolean representations - `NSNumber` is only [really useful for temporarily storing scalar values in objects](http://confusatory.org/post/56153849516/api-smell-nsnumber).
* Any date formatting is taken care of behind the wall of your service
* URLs returned as strings are converted to ready to use `NSURL` objects
* There is no exposure to the keys required from the web service - even if you declare these as constants to use with dictionaries, with a sufficiently complex API and proper namespacing you end up with very unwieldy code
* All response properties are now autocompleted and typed, and anything returned that you don't actually need isn't included, reducing complexity for the consumer
* Mocking web service responses (for unit testing the web service consumers) using a dedicated response object is far easier than creating a dictionary matching the web service response structure

Note however, that this object is *not* necessarily related to the app's data model. It is simply there to represent the results from the web service. 

With auto-synthesis of properties it is trivial to create and maintain custom dumb container classes like this. I tend to add the interface for the container object to the interface file for the associated web service, and an empty implementation at the top: 

```objc
@interface JRTLoginResponse : NSObject
@property (nonatomic,copy) NSString *accessToken;
@property (nonatomic,strong) NSDate *signupDate;
@end

@interface JRTLoginService : JRTWebService
@property (nonatomic,copy) NSString *username;
@property (nonatomic,copy) NSString *password;
@end
```

``` objc
@implementation JRTLoginResponse 
@end

@implementation JRTLoginService
...
```

There may be exceptions to this; it instinctively seems wasteful to convert a 2000-member array of simple dictionaries containing strings to specialist response objects, but it may not be - you'd have to use the profiler. *Always* go for code clarity and readability first, then work on performance **if you have a measurable problem.** 

The completion block of an individual web service class will either return a single response object, or an array of them, depending on the nature of the service. The response object is typed as `id` in the block definition, which means when using it you can [cast to the appropriate type](http://commandshift.co.uk/blog/2012/12/06/enumerate-using-block/). You could indicate the correct type for a specific service in comments in the header file, or redeclare the `startWithCompletion:` method to have the appropriate type in there, if you were being nice. 

## Implementation

Web services are all the same, except for the differences. So, our nice web service code should do all the generalisable work in the superclass, and offer the appropriate hooks for customisation by subclassing. I'd advise that for anything other than the most basic API, the network heavy lifting is done by [AFNetworking](http://afnetworking.com), but that is an implementation detail that consumers of the class don't need to know about. AFNetworking is great, but it's a generalised tool, and shouldn't be directly exposed in something like a view controller. If you decided to change your networking layer (say, if Apple released great [new networking classes](https://developer.apple.com/Library/ios/documentation/Foundation/Reference/NSURLSession_class/Introduction/Introduction.html) as part of a major iOS release, or [the networking framework you've been using falls behind](http://allseeing-i.com/ASIHTTPRequest/)) then you don't want to have to change code everywhere in your app.  

Those customisation hooks are:

* Configuring the request
* Creating the response object

Configuring the request involves three things: setting an endpoint, adding parameters and tweaking the URL request. 

For setting the endpoint, I have a method that returns an `NSString`, which subclasses must override. To make this clear the base class implementation contains an assert:

```objc
- (NSString *)endpoint
{
    NSAssert(NO, @"Subclasses must implement this method");
    return nil;
}
```

Asserts are great. It's like a comment that smacks you in the face if you ignore it.

For additional parameters, such as the username and password, all our subclasses need to do is convert those parameters into an `NSDictionary` that the  superclass can use to form a suitable URL request. This is the only method where you need to know API request-specific details such as URL parameter names, so in most cases you can happily add them right in here without needing to declare a bunch of constants:

```objc
- (NSMutableDictionary *)parameters
{
    NSMutableDictionary *parameters = [super parameters];
    [parameters setValue:self.email forKey:@"USERNAME"];
    [parameters setValue:self.password forKey:@"PASSWORD"];
    return parameters;
}
```

Note: my use of `setValue:forKey:` rather than `setObject:forKey` is deliberate; if any of these parameters have not been supplied, then this [method will still return a valid dictionary](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSMutableDictionary_Class/Reference/Reference.html#//apple_ref/doc/uid/20000141-BAJJCBIE) rather than crash. Optional API parameters are therefore taken care of without the need for conditional statements.

Any parameter types not compatible with URLs must be converted at this point, in a reverse of the response object principle, for example:

* Scalar values can be boxed : `setValue:@(self.isVIP) forKey:...`
* Dates would be formatted into strings as defined by the API

Finally, you [may need to be able to change properties on the NSMutableURLRequest itself](http://commandshift.co.uk/blog/2013/11/17/bug-hunting/). So create a method which takes the request as a parameter, which subclasses can optionally implement:

```objc
-(void)configureRequest:(NSMutableURLRequest *)request
{
    request.cachePolicy = NSURLRequestReloadIgnoringLocalCacheData;
}
```

Creating the response object is the only place where you need specific knowledge of the API's response structure. In most cases, it's a pretty simple mapping, but the principle here is that this method is the *only* place you need dirty your hands with the hard coded web service response keys:

```objc
- (id)responseObjectFromServiceResponse:(id)serviceResponse
{
    NSDictionary *dataDictionary = [super responseObjectFromServiceResponse:serviceResponse];
    JRTLoginResponse *response = [JRTLoginResponse new];
    response.accessToken = [dataDictionary objectForKey:@"access_token"];
    response.signupDate = [self.dateFormatter dateFromString:dataDictionary[@"signupdate"]];
    return response;
}
```

From this point on, you're only dealing with type-checked, autocompleted properties of a specialist object. It makes dealing with the response *so* much nicer. 

With this structure in place, an implementation of a subclass to deal with a specific API call can be done in very few lines of code. 

## Networking

All of the networking code is usually in the web service base class. I'm reluctant to post too much code on this because you'd basically insert your preferred networking methods here, but here's a basic outline using AFNetworking 1.3 (iOS6 compatible):

```objc
- (void)startWithCompletion:(JRTHTTPClientCompletionBlock)completionBlock
{
    // Set up a basic completion block to prevent having to check it all the time later
    if (!completionBlock)
    {
        completionBlock = ^(id response, NSError *error){};
    }
    // Here we call the endpoint and parameter methods from the subclass to prepare the request
    NSMutableURLRequest *request = [[JRTHTTPClient sharedClient] requestWithMethod:@"GET" path:[self endpoint] parameters:[self parameters]];
    // This allows any additional changes to the request
    [self configureRequest:request];
    AFHTTPRequestOperation *requestOperation = [[JRTHTTPClient sharedClient] HTTPRequestOperationWithRequest:request
                                                                                                     success:^(AFHTTPRequestOperation *operation, id responseObject)
        {
            NSError *serviceLevelError = [self serviceLevelErrorFromServiceResponse:responseObject];
            if (serviceLevelError)
            {
                completionBlock(nil,serviceLevelError);
            }
            else
            {
                // Here we call the response object method implemented in the subclass
                id processedResponse = [self responseObjectFromServiceResponse:responseObject];
                completionBlock(processedResponse, nil);
            }
        }
            failure:^(AFHTTPRequestOperation *operation, NSError *error)
        {
            NSError *friendlyError = [self serviceLevelErrorFromNetworkLevelError:error];
            completionBlock(nil, friendlyError);
        }
        ];
    [[SMOHTTPClient sharedClient] enqueueHTTPRequestOperation:requestOperation];
}
```

Obviously this only works for `GET` requests against JSON-based services. For the API this was developed against, all except one of the calls was of this type. For the single `POST` service, which was a multi-part form request requiring a progress block, the method above was completely overridden. There wasn't much repeated code from the main class because the setup was so different. For building against a different API, the code could perhaps be refactored to create the requests in a more customisable way. The principle remains the same: all of this stuff is hidden away from the consumer. 

## Error handling

You'll have spotted in the code above two methods referring to error handling. All web services are susceptible to network level errors (e.g. no connection, time out...), and some may return perfectly valid responses, which nevertheless indicate that an error has occurred. I refer to these as service level errors. As far as the consumers of your web service class are concerned, there are only errors, and you should return an `NSError` object containing whatever information you deem relevant. Depending on requirements you may wish to present specific error details, or just return blanket cover-all messages. You return any user-facing text in the localised description of the error's user info dictionary, and I like to use separate error domains for network and service level errors. 

## Recap

This is a nice way of handling your web services. Particularly when you compare it to the absolute car crash that you might think was acceptable if you read [certain popular tutorial web sites](http://www.raywenderlich.com/51127/nsurlsession-tutorial), where every detail of the networking code is exposed in methods in a view controller. That kind of thing may be acceptable in a tutorial because the point of the tutorial is to see the networking code all working in one place, but if you do your production code like that, you're doing it wrong.

The design outlined above keeps your web services separate to the rest of your app's logic, makes calling, modifying and testing them easier, and greatly simplifies using the responses in your app. To demonstrate the advantages,  I built a simple [Stack Overflow](http://stackoverflow.com) client[^1], [available on GitHub](https://github.com/jrturton/NiceOverflow). Hopefully the benefits are clear:

[^1]: Since I haven't used it before, I built this app using `NSURLSession` instead of NSURLConnection or a third-party library. After this brief adventure I'd still recommend AFNetworking - I found myself rewriting a couple of things (such as adding query strings to requests) which have already been implemented, more comprehensively, by that library. Cancellation of individual requests (which I didn't implement for this  example as it wasn't required) is also trivial if you base your networking on AFNetworking. 

* Individual web service implementations are very short and simple
* View controller implementations are not cluttered with networking code
* Integration tests with the API are straightforward and easy to add

Do your future self a favour, and rethink how you are implementing web services in your app! 

