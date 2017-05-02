--- 
date: 2015-07-14
layout: post
title: Things I learnt about the Photos framework
--- 

I recently worked on an app featuring a photo browser. The work involved updating the code (previously using Assets Library) to use the Photos framework. 

I found out a few things that don't seem to be represented very well in the documentation or the examples floating round online, so here they are. 

<!--more-->

## Image request completion blocks

For `.Opportunistic` delivery mode requests, which is the option you want when scrolling through an album, the completion block may be fired multiple (usually two) times - the image manager will get a fast, low-quality version, call the completion block, then call it again when it gets a higher quality version.  

What's important to note is that the first completion block can be executed _before the request function has returned_. This has a few implications. 

Async processes kicked off during cell configuration typically run like this:

1. Kick off the request
2. In the completion block, get the cell of interest, and set the relevant property

This setup means that if the async process takes some time to return, and the cell has been reused, the data isn't assigned to the wrong cell. You normally don't just capture the cell inside the completion block, then configure it, you'd normally capture the index path, then use that to get the appropriate cell from the table or collection view. If that returns nil, your cell is offscreen. 

However, for a completion block that can be executed immediately _or_ after some time, that pattern can't apply. You need to assign some identifier to the cell before you request the image, then check that identifier is the same when the completion block runs. 

[Apple's sample code](https://developer.apple.com/library/ios/samplecode/UsingPhotosFramework/Listings/SamplePhotosApp_AAPLAssetGridViewController_m.html) uses an incrementing integer value which is assigned to the tag of the cell. Normally I wouldn't touch the `.tag` property with a bargepole but this use almost makes sense. You could also use a property to hold the index path. You _can't_ use the request identifier, since the block can be executed before the request identifier has even been returned. 

The completion block is also called if the request is cancelled, so if you're canceling requests while scrolling (which is a good idea), you need to bear that in mind. You can see if the completion block is called for a cancelled request by checking in the `info` dictionary. 

Finally, for reasons unknown, sometimes image requests would simply never return a full sized image. The completion block would be called, there would be nothing amiss in the `info` dictionary, but the `result` comes back as nil. I had a single image where this would happen all the time. 

## Resizing mode

For scrolling through a large library, always use the `.Fast` resizing mode. You'd think that `.Exact` combined with opportunistic delivery would be optimal, but I found it made a huge performance difference, for no discernible quality difference, to stick with `.Fast`. 

## Caching

`PFCachingImageManager` is well worth using. For a huge library, you can't cache everything, and if the user is scrolling super fast then caching doesn't seem to offer much advantage. It does work very well for caching a few screens-worth of images either side of the current scroll position - with this in place  you get high-quality thumbnails very quickly when scrolling to the next page of images. 

Ignore caching when scrolling, but update the caches when scrolling stops, was the combination I settled on in the end. 

It's very tempting to optimise for the "scroll like a madman through the entire library" use case, it's very satisfying seeing those 60fps numbers and thinking of all the super-fast work that's happening, but it's only developers and QA engineers that do that in real life! 

## Change notifications

If you register for library change notifications, be very careful about how you handle them. It appeared to me that change notifications would come through very frequently when using an album built from a fetch request (which was every album in the project). The `PHFetchResultChangeDetails` object would contain incremental changes, which lead you to believe that there's only a few updates to deal with, but then I'd see _hundreds_ of indexes in the `changedIndexes`. Telling the collection view to reload all those cells, as suggested in the documentation, was a performance killer. 

I believe that these notifications are sent out when the framework is updating its "optimised storage" of the library, replacing its local caches with larger or smaller versions of the images. If you're showing an album view with identically sized thumbnail cells then you can safely ignore changes if they apply to cells outside the visible area. 

Moves, insertions and deletions seemed more reasonable and handling them as described in the documentation was fine. 

## Hardware environment

When you're testing, the phone and the environment make a huge difference. Network access speed and  device free space all affect performance. I had the same 10,000-image photo library take between 1.5GB and 5GB, depending on the available space on the device. This means your library is going to be faster on the device with more space, since it doesn't have to fetch from iCloud as often. 

## Closing thoughts 

The Photos framework is a big improvement over the Assets Library, and it does a great job of dealing with network requests, image resizing and caching for you. It would be amazing if it could also be used to deliver images from partner services like Facebook - there is already OS-level integration with some services, and extending it to Photos would make my job a lot easier!
