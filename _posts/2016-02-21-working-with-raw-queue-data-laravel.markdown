---
layout: post
title:  "Working with raw queues in Laravel"
date:   2016-02-21 23:08:00
categories: programming laravel
tags: featured 
---

> ["47% of users on the web expect websites to load in two seconds or less."][1] 

Two seconds may not seem like a hard feat to accomplish, but some tasks can really slow an application down, such as sending email(s). One way to get around this is to defer the task to a queue, to be processed later. 

Laravel has a built-in [queue service][2] that offers a unified API for working with multiple queueing back-ends. This queue service makes working with and hanling queues within the framework a breeze by pushing a JSON payload onto the queue containing a PHP serialization of the "job" class.

```javascript
{
    "job":"Illuminate\\\\Queue\\\\CallQueuedHandler@call",
    "data":{
        "command":"O:29:\\"Acme\\\Jobs\\\FooJob\\":4:{s:11:\\"fooBar\\";s:7:\\"abc-123\\";s:5:\\"queue\\";N;s:5:\\"delay\\";N;s:6:\\"\\u0000*\\u0000job\\";N;}"
    }
}
```

Potential security implications aside, this has a few drawbacks. For one, you can only ever process this queue job with PHP (unless you reverse engineer the unserialze function). Secondly if you want to, or need to, trigger a queue job from multiple codebases you will have to either copy and paste the class(es) to each codebase or go through the trouble of packaging and distributing the classes.

Thankfully, Laravel's queue API has the ability to [push raw data][3] onto whichever queue service you decide to go with. What if you want to process the raw queue data with Laravel though? While not specifically documented, it is possible if you dig through some of the frameworks inner workings.

If you look around line 125 of `Illuminate\Queue\Jobs\Jobs` you'll see the main logic that handles resolving and firing queue data to be processed. Within this logic you can see the recieved payload from the queue driver needs to have two keys within the JSON payload: `job`, and `data`. Job is a string with the fully qualified class name (FQCN) of the class working the queue data, and optional method name (default is `fire`) to be executed. Data is just that, it can be any sort of data in any format thats JSON complaint, and will be passed into the method/class specified in the `job` key as an associative array.

```javascript
{
    "job": "Acme\\\\Jobs\\\\FooJob",
    "data": {
        "fooBar": "abc-123",
        "baz": false,
        "bazBar": 1
    }
}
```

While you may have to maintain the FQCN string somewhere in your other codebases, I think this is a lot easier than reverse engineering `unserialize`, and allows you to still leverage the power included with the framework.

Hope this was helpful to some of you, if you have any questions, or notice something I got wrong, feel free to open an issue [here](https://github.com/hskrasek/hskrasek.github.io/issues). Would love to help out the best I can, or fix any of my mistakes (you can even open a PR and fix them for me if you so choose too).


[1]: https://blog.kissmetrics.com/loading-time/?wide=1
[2]: https://laravel.com/docs/5.2/queues
[3]: https://laravel.com/api/5.2/Illuminate/Contracts/Queue/Queue.html#method_pushRaw