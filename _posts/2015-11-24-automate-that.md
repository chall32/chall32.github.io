---
comments: true
layout: post
title: Automate That!
tags: []
date: 2015-11-24T18:30:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Zapier and IFTTT" src="/images/iftttzap.png">
Why not put the internet to work for you?

Internet automation apps are nothing new.  

What are internet automation apps? Put simply, they  connect together web apps.  An integration between two apps is called a Zap (Zapier) or a recipe (If This Then That – aka IFTTT).  A Zap or recipe is made up of a Trigger and an Action.  Whenever the trigger happens in one app, Zapier or IFTTT will automatically perform the action in another app.

It is within the Zap or recipe creation that the real fun begins.  If it happens, you can trigger something else happen. Attached are my uses of this technology. I'm just scratching the surface of what can be done here! 

Regular readers of this blog will know that I like to use twitter as a method of notification.   Therefore most of these are twitter related.

For all but the first recipe, I have another private twitter account that is tied to Zapier/IFTTT and I prefix all my notification tweets "@chall32".

Having said that there is no reason why you could not use some other form of notification; email, facebook, SMS are all possible.
  
{% include _toc.html %}

### Tweet New Polar Clouds Posts (IFTTT Recipe)
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Polarclouds Tweet" src="/images/polarclouds-tweet.png">
Post a tweet to [@PolarCloudsUK](https://twitter.com/polarcloudsuk) when a new blog post appears at [{{ site.url }}]({{ site.url }}). Include post URL.

### Tell me the Final Aresnal Score (IFTTT Recipe)
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Arsenal Score Tweet" src="/images/tweet-gunners.png">
Tweet [me](https://twitter.com/chall32) the final score of the Arsenal match.

### Tell me Tomorrow's Weather (IFTTT Recipe)
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Tweet Tonmorrow's Weather" src="/images/tweet-weather.png">
At 10pm in the evening, tweet [me](https://twitter.com/chall32) tomorrow's weather forecast. Include a 10 day forecast URL.

### Tell me if "Nexus" is Posted at avforums (Zapier Zap)
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Tweet if Nexus in avforums" src="/images/zap-avforums.png">
Follow [avforums](https://www.avforums.com/forums/) mobile phone classifieds [rss feed](https://www.avforums.com/forums/mobile-phone-classifieds.330/index.rss), filter feed item title to contain the text "nexus". If filter matches, tweet [me](https://twitter.com/chall32)

### Tell me if there is a New CyanogenMod Blog Post (Zapier Zap)
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Tweet if new CM Blog Post" src="/images/zap-cmblog.png">
Follow CyanogenMod's blog [rss feed](http://www.cyanogenmod.org/feed) and tell [me](https://twitter.com/chall32) when a new blog post appears at [www.cyanogenmod.org](http://www.cyanogenmod.org/blog)

### Tell me if any VMware related Jobs are posted in Kent (Zapier Zap)
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Tweet VMware Jobs" src="/images/zap-jobserve.png">
Nice one this.  Create an account on Jobserve, create a search (I use "VMware" and "Kent" as my search terms) and use the alert function (bell icon on the Jobserve search results) to create an RSS feed and create a Zap to monitor the Jobserve custom rss feed.  Tweet [me](https://twitter.com/chall32) job title and a link to newly posted job vacancies.

### Tell me About Unplanned Events on the M20 Motorway (Zapier Zap)
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Tweet M20 Incidents" src="/images/zap-m20.png">
Follow [Highways England](http://www.highways.gov.uk/traffic-information/) traffic information [rss feed](http://hatrafficinfo.dft.gov.uk/feeds/rss/UnplannedEvents.xml), filter feed item title to contain the text "M20". If filter matches, tweet [me](https://twitter.com/chall32).  Can be extended to cover other motorways too (M25 I'm looking at you!)

### Tell me if the OP Posts to an XDA Thread (Zapier Zap)
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Tweet XDA OP" src="/images/zap-xda.png">
Following custom ROM threads on [xda-developers](http://forum.xda-developers.com/) can be painful, with any Tom, Dick, Harry posting XYZ in the forum threads.  
Over the years, it occured to me that I'm really only intrested what the Original Poster (OP) - the custom ROM developer - has to say. For example "I've posted a new ROM build" or "Yes, fix for that is...". So here is how to get notified of OP posts on XDA forums.  

1. Create an email trigger in Zapier
2. Create XDA account as normal
3. Set your XDA contact email address to that provided by Zapier for incoming notifications (as set in step 1)  
4. Subscribe to the a thread you are intrested in. For example: [Nexus 7 Marshmallow - Android 6.0](http://forum.xda-developers.com/nexus-7/development/wip-nexus-7-marshmallow-android-6-0-t3222239)
5. Set XDA forums to instantly email you everytime someone posts to the thread
6. Configure Zapier to filter on "Body Plain" of the notification email for text that contains "XYZ has just replied" - so in the case of the example thread above, my text filter would be "Motorhead1991 has just replied"
7. If filter matches, tweet [me](https://twitter.com/chall32)
8. Repeat steps 4 to 6 for OP of any other threads you have subscribed to

Theoretically this could work on any forum thread of any forum that sends immediate post notification emails. 


Well there we have it friends, internet automation.  Enjoy!

-Chris
