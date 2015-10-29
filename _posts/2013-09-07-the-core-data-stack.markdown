---
layout: post
title: "The Core Data Stack"
date: 2013-09-07 22:36
comments: false
categories: 
---
Recently I attended a Core Data workshop given by [Marcus Zarra](https://twitter.com/mzarra) at [iOSDevUK](http://www.iosdevuk.com). It was brilliant. He threatened to talk and talk until we all passed out, which would have been great, except I had to catch the train home. If you are at all interested in Core Data (and if you're reading this, I have to assume you are) then listening to the man who [literally wrote the book on it](http://pragprog.com/book/mzcd2/core-data) is an opportunity you shouldn't miss. 

One of the things he mentioned was the "Core Data Stack", the term typically used to refer to the `NSManagedObjectModel`, `NSPersistentStoreCoordinator` and `NSManagedObjectContext` that make up the heart of Core Data.

> "You can set this up in five lines of code"

Talking to others in the room during the break, it seemed that  a lot of people were sticking (or stuck with) with Apple's template code, which is considerably more than 5 lines and, to add insult to injury, also lives in the app delegate. 

While I'm waiting for Marcus' book to arrive, let's try and set up a core data stack in five lines of code and see what happens. Note that none of this is code we were shown in the workshop, and is not endorsed by Marcus Zarra in any way. By the end of the post, I'll have explained how to add a fully functional Core Data stack to your project.

<!--more-->

It's worth noting that this isn't intended to be _the_ core data stack you should be using - it just gives you the same functionality as the Apple template code, in a hopefully more readable fashion. 

### Line 1: The managed object model

The `.xcdatamodel` is converted at build time to a .momd file which is used to drive the whole model. You can name a specific file or use `mergedModelFromBundles:`, which does exactly what it sounds like and is better for code reuse, as it will be the same in every project. Once you've created this object and passed it to the persistent store coordinator, you'll never refer to it again. Here's how we do that:

```objc
NSManagedObjectModel *model = [NSManagedObjectModel mergedModelFromBundles:[NSBundle allBundles]];
```

### Line 2: The persistent store coordinator

This object is the real brains behind core data. It moves your data between objects in your application and objects in the persistent store (typically, an SQLite file). We just need to create it. Again, once we've made it and handed it to the managed object context, we don't need to refer to it again. 

```objc
NSPersistentStoreCoordinator *psc = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:model];
```

### Line 3: The storage location

This part will vary between your applications. Most commonly it's a path to a specific file in the documents directory. There's no reason not to call it the same thing in every app you ever make, though. 

```objc
NSURL *url = [[[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask] lastObject] URLByAppendingPathComponent:@"Database.sqlite"];
```

This one is a bit of a cheat, and should probably be on more lines - four opening square brackets is never a good sign. 

### Line 4: Creating a persistent store

Since we're trying to fit into five lines, I'll gloss over options and error checking here, but I will cover it later on. 

```objc
[psc addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:url options:nil error:nil];
```

### Line 5: The managed object context

This is the public face of your core data stack, so would be a property declared in the interface of whatever object is handling your core data stack. This stack handling object can be a singleton, or an object created at launch time and passed into your controllers by dependency injection. Mr. Zarra expressed a preference for the latter option. I'm a singleton fan myself, but [astute](https://twitter.com/abizern) [readers](https://twitter.com/frosty) have pointed out several advantages, including testability and the fact that you would probably pass in the store URL as an initialisation argument, saving us a line above. I will revisit this in future projects. 

Here's how you create a managed object context and assign it to this public property:

```objc
self.managedObjectContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSMainQueueConcurrencyType];
```

### Line 6: Adding the persistent store to the coordinator
Wait, line 6? Well, there are many reasons I'm not the world's foremost Core Data expert, and this is probably one of them.

```objc
self.managedObjectContext.persistentStoreCoordinator = psc;
```

There we go! Core data in <strike>five</strike> six lines. We could have got it into fewer lines, but it would have involved a bit more nested code and made explaining each line twice as hard. 

In practice, and with a code reuse and practicality in mind, we can make some improvements. These are both to the `addPersistentStore...` method. 

### Error handling

You have to have this, really. You'll get errors when adding the persistent store to the coordinator, usually in the following circumstances:

* You're in initial development, and you are making changes to your managed object model. You wouldn't bother making a new version of the model for changes like this, so your app crashes every time on launch (if using Apple's template) telling you it can't create the persistent store.
* You've released a new version of your app with a new data model, and haven't versioned or enabled migration
* Something terrible is happening on the device and you can't create the file in the place you expected to.

In the first case, you should just delete the existing storage file and start again. In the second case, you should have enabled migration and versioning, but if that has failed, deleting the data is probably best. 
In the latter case, there isn't much you can do except show the error.  

### Options

Here you can tell the persistent store coordinator to encrypt the data store (while the device is locked or booting). Note that this only helps your user's security if they have also enabled a PIN unlock on the device. 

You can also set automatic migration up. This allows Core Data to cope with changes to your model, as long as you've made those changes in a new version of the model. 

The dictionary for these options then becomes:

```objc
NSDictionary *options = @{NSPersistentStoreFileProtectionKey: NSFileProtectionComplete,
                              NSMigratePersistentStoresAutomaticallyOption:@YES};
```

### A core data stack in slightly more than five lines

Let's put it all together.

```objc
-(void)setUpCoreDataStack
{
    NSManagedObjectModel *model = [NSManagedObjectModel mergedModelFromBundles:[NSBundle allBundles]];
    NSPersistentStoreCoordinator *psc = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:model];
    
    NSURL *url = [[[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask] lastObject] URLByAppendingPathComponent:@"Database.sqlite"];
    
    NSDictionary *options = @{NSPersistentStoreFileProtectionKey: NSFileProtectionComplete,
                              NSMigratePersistentStoresAutomaticallyOption:@YES};
    NSError *error = nil;
    NSPersistentStore *store = [psc addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:url options:options error:&error];
    if (!store)
    {
        NSLog(@"Error adding persistent store. Error %@",error);

        NSError *deleteError = nil;
        if ([[NSFileManager defaultManager] removeItemAtURL:url error:&deleteError])
        {
            error = nil;
            store = [psc addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:url options:options error:&error];
        }
        
        if (!store)
        {
            // Also inform the user...
            NSLog(@"Failed to create persistent store. Error %@. Delete error %@",error,deleteError);
            abort();
        }
    }
    
    self.managedObjectContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSMainQueueConcurrencyType];
    self.managedObjectContext.persistentStoreCoordinator = psc;
}
```

_(Thanks to [@paulio87](https://twitter.com/paulio87) for correcting the error handling from the original)_

This can be used to replace the confusing mess from the Apple template in any core data project. To add core data to an existing project and avoid the Apple template in the first place, all you need to do is:

* Include the Core Data framework
* Import the header in `prefix.pch` : `#import <CoreData/CoreData.h>` alongside the existing UIKit and Foundation header imports
* Create a managed object model, which can be added using the standard add file dialogue. 

There you go, freed from the clutches of the rubbish template code! No longer will you have to create a master-detail app template and delete most of it just to get core data added to your project!

