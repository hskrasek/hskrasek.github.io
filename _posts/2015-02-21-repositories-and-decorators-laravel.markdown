---
layout: post
title:  "Repositories and Decorators"
date:   2015-02-21 14:34:25
categories: programming meetup talks
tags: talks laravel
---
During the [Laravel Austin][latx] meetup this month, I gave a talk over the repository and decorator pattern, along with a demonstration on how they are useful. First off the repository and decorator pattern is not an official pattern, just the combination of using repositories with the decorator pattern, but "Repository and Decorator Pattern" sounds cooler as a presentation title than "Using repositories with decorators".

### Repositories

So what exactly are repositories? Well, repositories allow us to abstract away the database layer. With repositories, we can remove business logic/database logic from your controllers, and prevent our code from becoming coupled to a single database implementation.

Lets create a few repositories, implementing the following interface:
_Note: The following examples use [Laravel][laravel] specific code, but the concepts can be used anywhere_
{% highlight php %}
<?php

interface UserRepositoryInterface {
    public function getAll($limit = 10, $offset = 0);
}
{% endhighlight %}

With this interface, you could create repositories that use Eloquent, Mongo, Redis, Text files... I think you get the point.
{% highlight php %}
<?php

class EloquentUserRepository implements UserRepositoryInterface {
    public function getAll($limit = 10, $offset = 0) {}
}

class MongoUserRepository implements UserRepositoryInterface {
    public function getAll($limit = 10, $offset = 0) {}   
}

class RedisUserRepository implements UserRepositoryInterface {
    public function getAll($limit = 10, $offset = 0) {}
}
{% endhighlight %}

Now instead of referencing Eloquent, or the repository directly, lets inject the interface instead. This way, it doesn't matter what implementation we use, as long as it is an instance of UserRepositoryInterface.
{% highlight php %}
<?php

class UserController extends Controller {
    public function __construct(UserRepositoryInterface $user)
    {
        $this->users = $users;
    }

    public function index()
    {
        $users = $this->users->getAll(Input::get('limit', 10), Input::get('offset', 0));

        return Response::json($users);
    }
}
{% endhighlight %}
### Decorators

So now that we have a basic idea of what repositories are, what about decorators, what are they needed for? Let me 
pose a hypothetical.

Your application stores users on a MySQL database, and things are running smoothly until you start to notice your 
application running slow. You notice the slow down is due to constantly looking up users in the database, so what do 
you do? Add more database servers? Shard your MySQL instance? All of these would work, but they only mask the 
problem, and expanding horizontally like that begins to add up in server bills. So lets add some caching instead, so 
that we don't have to hit our database servers as often.

Your first reaction might be to add caching to the _UserController_ around the repository calls, but handling caching
 really isn't the responsibility of a controller. So, lets edit the _EloquentUserRepository_ directly, and add 
 caching to that. Well, even that has its issues. Every time you touch a class to add functionality, you run the 
 risk of introducing new issues, so how should we do this? That's where decorators come in!

Decorators allow you to wrap classes to extend functionality, without editing the original class. If structured 
correctly, you can decorate a class over and over again to add new functionality. How do you accomplish this? Well, 
thats where the _UserRepositoryInterface_ comes in handy, it can be used as a contract to ensure that the decorator 
implements all the required functions. Lets create a decorator to add caching to our application.
{% highlight php %}
<?php

class CachingUserRepository implements UserRepositoryInterface {
    public function __construct(UserRepositoryInterface $repository, Cache $cache) 
    {
        $this->repository = $repository;
        $this->cache = $cache;
    }

    public function getAll($limit = 10, $offset = 0) 
    {
        // Pull the users out of cache, if it exists...
        return $this->cache->remember('users.all', 60, function() use ($limit, $offset) {
            // If cache has expired, grab the users out of the database
            return $this->repository->getAll($limit, $offset);
        });
    }
}
{% endhighlight %}

As you can see, we inject an instance of _UserRepositoryInterface_ into the _CachingUserRepository_ class that handles the caching functionality, while using the injected _UserRepositoryInterface_ to load users if the cache has expired. This way we can maintain our seperation of concerns, and keep our code nice and clean. How would this be constructed? Well, when we bind our concrete implementation to our IoC container, it might look something like this:
{% highlight php %}
<?php

public function register()
{
    $this->app->singleton('UserRepositoryInterface', function() {
        $eloquentRepo = new EloquentUserRepository(new User);
        $cachingRepo = new CachingUserRepository($eloquentRepo, $this->app['cache.store']);

        return $cachingRepo;
    });
}

{% endhighlight %}

### Wrapping up

In closing, this pattern allows you to write clean and highly extendable code, that follows [SOLID][solid] principles. But, before you jump into using this for every single project you start, remember this is better suited for large applications. Obviously there is nothing wrong with "over engineering" a project for the sake of learning a new concept, but I would slow down a personal weekend project worrying about clean code. Refactoring exists for a reason, and you can always go back later and clean up that weekend project if it grows into something more.

Would you like to learn more about using decorators and repositories? You should check out this [video][laracast-video] from [LaraCasts][laracasts]. A few other videos from [LaraCasts][laracasts] about these topics that I highly recommend are:

* [The Decorator Pattern][lara-decorator]
* [Repositories Simplified][lara-repositories]
* [Repositories and Inheritance][lara-repo-inheritance]

[latx]:                  http://laravelaustin.com/
[solid]:                 http://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29
[laravel]:               http://laravel.com/docs/5.0/installation
[laracasts]:             https://laracasts.com/
[laracast-video]:        https://laracasts.com/lessons/decorating-repositories
[lara-decorator]:        https://laracasts.com/lessons/the-decorator-pattern
[lara-repositories]:     https://laracasts.com/lessons/repositories-simplified
[lara-repo-inheritance]: https://laracasts.com/lessons/repositories-and-inheritance