---
layout: post
title:  "UUID Paw Extension"
date:   2015-02-22 15:54:00
categories: programming
tags: paw extensions paw
---
While working on API's I like to user various tools to test endpoints, and in the past I would use the Google Chrome 
extension [POSTman][postman], which I would highly recommend. Recently though I have started using the Map 
application [Paw][paw], which despite costing money, is well worth the cost. 

One benefit of Paw is that it comes with many built in exentsions, as well as the ability to create your 
[own][paw-extensions]. While working on Perk.com's API, I needed a way to generate different device identifiers such 
as IDFA, IDFV's and more, which are typically in the format of UUID's. Unfortunately Paw doesn't have a built in UUID 
generator, so I decided to create my own. Paw's extensions are built on JavaScript code, and even support being 
written in CoffeeScript if your into that sort of thing, so creating the extension was rather easy. I decided to try 
my hand at CoffeeScript and came up with the following script:

{% highlight coffeescript %}
RandomUuidValue = ->
    @evaluate = (context) ->
      "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace /[xy]/g, (c) ->
        r = Math.random() * 16 | 0
        v = (if c is "x" then r else (r & 0x3 | 0x8))
        v.toString 16

    @title = ->
      "UUID"

    return

RandomUuidValue.identifier = "com.hskrasek.PawExtensions.RandomUuidValue"
RandomUuidValue.title = "Generate Random UUID"
RandomUuidValue.inputs = []
registerDynamicValueClass RandomUuidValue
{% endhighlight %}

Once this has been compiled into JavaScript, Paw can use this to generate UUID4's wherever you may need them. I have 
been trying to get the extension listed on the site, which makes installation easier, but unfortunately haven't had 
much luck. If you'd like to use this exension though, you can install it with the following commands:

{% highlight bash %}
cd ~/Library/Containers/com.luckymarmot.Paw/Data/Library/Application\ Support/com.luckymarmot.Paw/Extensions
git clone https://github.com/hskrasek/Paw-UUIDDynamicValue com.hskrasek.PawExtensions.RandomUuidValue
{% endhighlight %}

And then if you open Paw, it will be available under the extensions menu. If you already have Paw open you can open 
Preferences, go to the Extensions tab, and click Reload Installed Extensions, and you should be good to go.

If you'd like to contribute to this extension, please feel free to open a pull request on [Github][gh-link]. 

[paw]:              http://luckymarmot.com/paw
[postman]:          https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm?hl=en
[paw-extensions]:   http://luckymarmot.com/paw/extensions/
[rfc]:              http://www.ietf.org/rfc/rfc4122.txt
[gh-link]:          https://github.com/hskrasek/Paw-UUIDDynamicValue