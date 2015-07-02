---
layout: post
title: "Using dispatch groups to wait for multiple web services"
date: 2014-03-19 20:45
comments: false
categories: 
---

Imagine your app has to run a series of [nice web service calls](http://commandshift.co.uk/blog/2014/01/02/nice-web-services/). These could be for a set-up task, for example - when your app launches it may need to get various bits of configuration information from a server. This could involve hitting several endpoints. 

You want to call a single method to kick this process off, and have a completion block run when it has finished. The web services don't depend on each other. How can this be done?

<!--more-->

There's a horrible way: start each service from the completion block of the previous one, and when the last one is done, call your overall completion block. 

Any time you see this in your code, you're probably doing it wrong:

```objc
                    }];
                }];
            }];
        }];
    }];
```

The _nice_ way to do this is by using dispatch groups. A dispatch group monitors work that has been added to it, and it will know when that work is done. 

## Creating a dispatch group

This is easy:

```objc
dispatch_group_t serviceGroup = dispatch_group_create();
```

## Adding tasks to a dispatch group

There are two ways to do this. The first adds a block of code to the group, to be executed on a specfied queue:

```objc
dispatch_group_async(serviceGroup,queue,^{
    // some work here
});
```

This isn't really suitable for us; our web service calls return straight away and have their own completion blocks, so the group would think the work was done immediately.

The second way is to manually tell the group that you are starting, or have finished, blocks of work. This is done using `dispatch_group_enter()` and `dispatch_group_leave()` respectively:

```objc
dispatch_group_enter(serviceGroup);
[configService startWithCompletion:^(ConfigResponse *results, NSError* error){
    // Do something with the error or results
    dispatch_group_leave(serviceGroup);
}];
```

Each `enter` must be balanced by a corresponding `leave`, or the group will never finish. You `enter` before calling the service, and `leave` at the end of the completion block. 

## Acting when the group is finished

There are two options here as well. You can block the current thread until the group is finished:

```objc
dispatch_group_wait(serviceGroup,DISPATCH_TIME_FOREVER);
// Won't get here until everything has finished
```

Or, you can tell the group to run another block when the work is done, and then return from your method:

```objc
dispatch_group_notify(serviceGroup,dispatch_get_main_queue(),^{
    // Won't get here until everything has finished
});
```

The latter seems more suited to our purposes - we expect methods with completion blocks to return immediately. 

## Putting it all together

We can implement our multiple-web-service-calling method like this:

```objc
-(void)fetchConfigurationWithCompletion:(void (^)(NSError* error))completion
{
    // Define errors to be processed when everything is complete.
    // One error per service; in this example we'll have two 
    __block NSError *configError = nil;
    __block NSError *preferenceError = nil;
    
    // Create the dispatch group
    dispatch_group_t serviceGroup = dispatch_group_create();
    
    // Start the first service
    dispatch_group_enter(serviceGroup);
    [self.configService startWithCompletion:^(ConfigResponse *results, NSError* error){
        // Do something with the results
        configError = error;
        dispatch_group_leave(serviceGroup);
    }];
    
    // Start the second service
    dispatch_group_enter(serviceGroup);
    [self.preferenceService startWithCompletion:^(PreferenceResponse *results, NSError* error){
        // Do something with the results
        preferenceError = error;
        dispatch_group_leave(serviceGroup);
    }];
    
    dispatch_group_notify(serviceGroup,dispatch_get_main_queue(),^{
        // Assess any errors
        NSError *overallError = nil;
        if (configError || preferenceError)
        {
            // Either make a new error or assign one of them to the overall error
            overallError = configError ?: preferenceError;
        }
        // Now call the final completion block
        completion(overallError);
    });
}
```

You keep track of any errors encountered by the individual service and can either make a new NSError object based on what went wrong, or pick a particular one to return. 

This is far superior to chaining calls via their completion blocks - particularly if you decide to remove a call, or add extra ones. 
