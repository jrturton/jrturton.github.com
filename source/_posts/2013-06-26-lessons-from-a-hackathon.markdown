---
layout: post
title: "Lessons from a Hackathon"
date: 2013-06-26 19:58
comments: false
categories: 
---

At the weekend I attended the [EE Hackathon](http://community.ee.co.uk/t5/featured-trending/4GEE-HACKATHON-2013/ba-p/25340) which was arranged in partnership with [Mubaloo](www.mubaloo.com), my employer. I've never been to an event like this before, so I had no real idea what to expect or how it would go. Here are a few of the things that I learnt.
<!--more-->
##I work with some very talented people
[@frosty](https://twitter.com/frosty), [@tealspoon](https://twitter.com/tealspoon), [@dnlkbox](https://twitter.com/dnlkbox) , who were the Mubaloo "B" team, and [@liamnichols_ ](https://twitter.com/liamnichols_) and [@valentinkalchev](https://twitter.com/valentinkalchev), who were with me on the, well, OK, the "C" team, were all fantastic, if only at putting up with me for 36 hours. That's a feat of endurance few can manage. Between us we also managed to produce two pretty good apps, without much in the way of hiccups. 

##Unit tests are great
{% blockquote one of the guys from cloubase.io %}
Unit tests? In a Hackathon? 
{% endblockquote %}

Unit tests indeed. When you've got three people working on separate components of the same app under a tight deadline, you all have to be working in parallel. This means  you need to be writing functioning code without depending on others to get their stuff finished first. Using a unit test framework to exercise the code independently worked brilliantly. Also, unit tests are always good.

##[AppCode](http://www.jetbrains.com/objc/) is great
You may have known that already. My two AppCodeless colleagues looked on with envy as imports were automatically done, properties were added as I was referring to them and method signatures were updated without so much as breaking a sweat.

##Source control is a challenge in a fast-moving project
We used [Bitbucket](https://bitbucket.org) to create a shared private repository, and supercharged our git command lines using [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh). With over 150 commits during the project, we had one or two problems with merging, almost always in the `project.pbxproj` file. When you have three people adding files to a project, stuff like this happens. Luckily they were usually pretty straightforward to fix, though it sometimes seemed a little dispiriting:

{% tweet https://twitter.com/richturton/statuses/348491191240781824 align='center' %}

And we also made some common typos:

{% tweet https://twitter.com/richturton/statuses/348473671154806784 align='center' %}

##4G is fast. And reliable.
According to the guys from EE, we were all on a single 4G cell centered in Leicester Square. There were several 4G WiFi routers dotted around a workspace filled with about 100 nerds, so as you can imagine there was a lot of running of [SpeedTest](http://www.speedtest.net) and torrenting going on. We never noticed a drop in network activity or anything other than high speed connections. Even after this:

{% tweet https://twitter.com/richturton/statuses/348457789502406656 align='center' %}

##Do it for the experience, not the win
Given our connection to the organisation, we weren't allowed to actually _win_ anything. This allowed us to relax and enjoy ourselves, even though [the boss](http://uk.linkedin.com/in/markemason) was turning up to judge the entries. We dealt with this pressure using beer. Lots of lovely, **free** beer. 

{% tweet https://twitter.com/frosty/statuses/348499379432349696 align='center'%}

##Sleep when it's done
You've only got 30 odd hours. Spending any of them sleeping is not a great idea, particularly since that would risk waking up hung over. A policy of continual, moderate alcohol intake seemed to keep us on a fairly productive plateau, give or take a few moments of rambling weirdness. 

##Eat good food
I was expecting a beige buffet, and was very pleasantly surprised with the quality and standard of the food we were given. Great salads, pies, pizzas, pastries at breakfast, and fruit. Of course, you can't help some people:

{% tweet https://twitter.com/richturton/statuses/348412823132581888 align='center' %}

##Keep up morale
Our teams were definitely _noisier_ than the others we were sat near (sorry!). There was plenty of laughter, and during the second morning we were all looking forward to the official Mubaloo Ministry of Fun turning up:

{%tweet https://twitter.com/claironeill/status/348692429697871872 %}
Who's driving? Sarah, Claire and JP on the way to hoover up any remaining free beer.

##Demos and presentations are hard
Our presentation to the judges was fine, but it all went wrong when I was presenting to the rest of the group. I wanted to run the app on the simulator on my laptop which was plugged into the projector, while the other guys used it on their iPads so we could demonstrate the real time updating over the network. The following occurred:

- I plugged in the laptop and somehow it decided that the resolution to display was the size of a postage stamp. We had three minutes to do the whole presentation, including setup time, and I should have had the display preferences pane open before I started. I should also not have had the retina simulator running so I could have shrunk the window down slightly.
- The network, or one of our back end services, chose demo time to stop working. We'd moved to a different area of the building for this and though we'd brought our router, we didn't test it in the audience before we got up. Therefore our demo failed miserably. Luckily(?) a lot of previous teams had also had problems. We should have recorded a video to use in the event of a failure like this. We'd had no problems of this type leading up to the presentation, and had probably got complacent.

##Be careful what you tweet about
It was a networking-themed event with a focus on realtime information. One of the demos involved tweets with the #eehackathon hashtag floating about on the screen, this one seemed to show up quite a lot:

{%tweet https://twitter.com/richturton/statuses/348620691639255041 align='center'%}

##Book the following day off work
If you don't, stuff like this happens:

{%tweet https://twitter.com/frosty/statuses/349092375001444353 align='center' %}