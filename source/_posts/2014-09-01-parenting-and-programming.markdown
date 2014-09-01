---
layout: post
title: "Parenting and Programming"
date: 2014-09-01 21:14:56 +0100
comments: false
categories: 
---

My second daughter was born this week. In a change to your usual programming, here's a little whimsical reflection on the parallels between being a parent (though my experience of the former only goes up to 4 years) and being a programmer. Mostly because it's hard to write code-based blogs with a sleeping baby on your lap. 

<!--more-->

## You didn't write it
You didn't even write 50% of it, despite what you may have learned about genetics at school. Evolution has been working on this for _ages_; there's a lot of legacy code and uncommented workarounds in there.

Like coming in to a half-finished project by someone else, there's a lot of head scratching and experimentation before you start to figure out how the thing works, and there's a lot of stuff that you would have implemented differently, given the requirements. Unfortunately, a rewrite isn't on the cards. 

## Crashes during development 

Adding new features is frequently accompanied by breakages and the temporary introduction of apparently unrelated bugs in different areas, while underlying frameworks are rewritten, upgraded and integrated. I've found that my eldest daughter follows a fairly predictable pattern of a couple of weeks of bad (or less good, she's a delight) behaviour, followed by the appearance of a new power, like forming sentences or contracting verbs properly. 

## Nervousness at release time

You can have the best QA in the world but sending your little app out on its own to fend for itself is always a little nerve-wracking.

## Don't expect to do a good job when you're tired

Unfortunately, you often don't have a choice. Parenthood is a state of perpetual tiredness. You must make the most of any down time you do get. Don't, for example, sit up until 1am writing blog posts about how parenting is just like programming. Recognise your own tiredness and know that you won't be doing your best work.  

## Don't expect to do a good job whilst constantly checking social media 

<blockquote class="twitter-tweet" lang="en"><p>PRETEND you&#39;ve got children by looking up from Twitter every five minutes to absent-mindedly shout something stern. (via <a href="https://twitter.com/superluckytiger">@superluckytiger</a>)</p>&mdash; Twop Twips (@TwopTwips) <a href="https://twitter.com/TwopTwips/statuses/481356028916031489">June 24, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Nobody really knows what they're doing yet the internet is full of "experts" 

...except of course people with a high reputation on say, [Stack Overflow](http://stackoverflow.com) or [Potty Overflow (well, Parenting Stack Exchange)](http://parenting.stackexchange.com).

## Notifications versus polling
You're on a car journey. Being asked "are we there yet?" every 30 seconds is inefficient and a drain on limited resources (of patience, primarily). A more sensible system is for the child to fall asleep and be woken when the journey is over. Similarly, use notifications or completion blocks or even (if you're feeling retro), delegate methods to detect when a long-running process has ended. Don't create a timer and poll all the time. 

## Single responsibility principle
There's a toy which looks fantastic. It's a robot, that flies, and talks, and teaches you to spell. No it doesn't. None of it works properly and it breaks really easily. The best toys, like the best classes, do one thing and do it well. 

## Reference cycles
If a `Parent` object holds a strong reference to a `Child` object, and the `Child` object holds a strong reference back to the `Parent`, then both objects will live on forever. This is more desirable in parenting. It should generally be avoided when programming. 

