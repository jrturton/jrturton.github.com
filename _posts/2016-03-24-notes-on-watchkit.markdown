--- 
date: 2016-03-24
layout: post
title: "Notes on WatchKit"
--- 

I recently completed a project involving a WatchKit app. It was not a pleasant experience, so here's a screed of vague complaints with some half-baked possible solutions, and a possible ray of sunshine at the end. 

## Tools and hardware 

The simulator seems unreliable at connecting to the phone simulator and the debugger. In this respect it is an accurate simulator. 

Installing and debugging on the watch is a painful experience. Communication is over Bluetooth, via a connected phone. Swift watch apps have a minimum footprint of around 7MB due to inclusion of the standard libraries, so installation takes a while, if it works at all. If you're not interested in debugging, it's often more reliable to build and install the phone app, then use the Watch app to install the watch app, which is not an easy sentence to read. 

You can't wait for the app to launch, like you can with phones, so debugging the arrival of notifications or startup issues is not possible. That's a shame, because the arrival of notifications and app startup is a fraught and bug-prone process. 

Very often the debugger will fail to attach to the process or claim it has finished running the app. The usual dance of restarts seems to fix this, but the key to success seems to be that you need the "Trust this computer?" dialogue to show up _on the watch_. After that, you're usually ok. 

Because of the slow communication round trip, breakpoints and debugger commands can take some time to execute. The watch also has chronic logorrhoea, if you look at the devices tab in Xcode, which can make tracking issues by console logging (because you can't wait for the app to launch...) something of a filtering exercise. 

All of this is very hard work on the watch, which is a tiny device not designed for long periods of heavy Bluetooth and screen use. This means the battery will die rather quickly. You could put it on the charger, but then you have to enter the PIN every time to unlock the device. You could remove the PIN, but then goodbye Apple Pay. You could buy a development watch, but there's no economic incentive to develop a watch app since they come "free" with your iOS app. 

Solution? Some sort of developer dock which charges _and_ keeps the device unlocked? Bonus points if it enables direct communication as well. 

## SDK

WatchKit seems very stuck with the limitations from watchOS 1 - write-only access to most UI elements being the most glaring one. That didn't seem to present too much of an issue. The following things did trip me up; they're mentioned in the documentation but perhaps not in big enough writing:

- `WCSession` messages are _always_ handled on a background thread. If you're making UI changes in response to them, you have to dispatch to the main thread. 
- UI updates are ignored if the interface controller is not active. If you are updating UI in response to messages or network calls or something else you don't control the timing of, you may need to maintain some separate state and update your UI when that changes, and again when the interface controller activates. 
- For navigation-based interfaces there is no non-animated update to the navigation stack, which can make some UI updates jarring. 
- Shared user defaults and core data storage disappeared with watchOS 2. You are responsible for shuttling data back and forth using `WCSession`, which can be quite tiresome. 

None of the above are impossible to surmount, except the non-animated navigation stack. Overall the SDK reflects well the limitations of the device, from a capability and UX perspective. I expect improvements for watchOS 3, and will be interested to see what the next generation of devices are like. 

## Bugs

[The major bug encountered](http://openradar.me/25214443) was due to app startup in response to a notification being tapped. A lot of time was burnt on this, because the watch app does some interesting things on startup which we assumed were being done wrong, until we realised it was a simple matter of tapping the notification too quickly. 

Even if that bug didn't exist, app launching is painfully slow, and must be the major factor preventing the use of third party apps. 

##  Good things

The best things, by a country mile, are the dynamic notifications. This allows you to build a custom UI to display in response to a notification. Since notifications are undoubtedly the watch's "killer app", this had to be good, and it is. The following things would be even nicer:

- Dynamic notifications without the requirement for an actual watch app. 100% of the time I just read a notification and either dismiss it or hit one of the actions (usually to delete an email or like a tweet). I don't need to open an app on the watch, with all the waiting and the limited UI, and developers of apps that have interesting notifications but no need for a watch app should be able to take advantage. 
- Dynamic notifications on iOS. Having seen what's possible on the watch, showing a bit of text and some buttons seems terribly limited now. 

## Lessons

Apple say that you should expect interactions with watch apps to be measured in seconds. Assuming they're not counting the seconds spent waiting for the app to start, this is true. Nobody wants to spend time prodding at a tiny screen whilst holding their arm up awkwardly. 

The third party apps I use on the watch are: 

- Runkeeper
- Omnifocus (rarely)

I have several more installed, but day-to-day I simply don't use them. I use _lots_ of notification actions, though. 

Your watch app should deliver the absolute minimum useful level of functionality (or, it should not exist - that is a valid choice and correct in most cases!). Think very carefully before adding navigation stacks, master-detail interfaces and the like. They're not nice to write, and not nice to use. 

Start with a single screen. And in most cases, stop right there. 
