---
layout: post
title:  "Introduction to Railties"
date:   2013-07-10 21:59:37
categories: railties rails gsoc
---

As part of my [Google Summer of Code][gsoc] project I'll be refactoring Railties, the internal engine of Ruby on Rails. When a Rails application launches, there is a complicated process of configuration and initialization that sets the Rails web server.

Before you can actually set up this web server, there are quite a few things that need to be configured. Railties are the engines for reading these configurations, creating an application, and connecting different parts of Rails.

### The Railtie Class

All major components of Rails are created as a subclass of `Rails::Railtie`. For example, Active Record (which handles query syntax to a database) and Action Controller (which handles the base controller logic) are both Railties.

#### So what does `Rails::Railtie` do?

Well, to figure out the function of `Railtie`, we can go look at the source code. Finding the definition isn't too tricky. You can clone the Ruby on Rails source code from [github][railssource]. My primary method for finding class definitions is by using ack-grep, a convenient command line tool for searching through files. I searched for the following:

{% highlight bash %}
ack-grep "class Railtie"
{% endhighlight %}

This gives us the file ["railties/lib/rails/railtie.rb"][railtiefile] which defines `Railtie`. A casual perusal of the file tells us a couple of things:

1. `Railtie` is abstract. This means that it is not intended for direct instantiation. This is pretty clear because there is an error which is thrown inside of the `initialize` method whenever the class attempting to be initialized is found in the `ABSTRACT_RAILTIES` constant.
2. `Railtie` serves mostly as an API that should be subclassed. This observation comes directly from the observation that `Railtie` is abstract. Abstract classes usually define methods which should be overwritten in subclasses. The documentation also gives us a good clue, because it provides examples where one subclasses `Rails::Railtie` and configures that subclass.
3. `Railtie` has four main methods. These are `rake_tasks`, `console`, `runner`, and `generators`. For each of these methods, a block is added to a list of previously defined blocks. Here we can see `Railtie`'s true functionality: providing a way to configure anything which subclasses `Rails::Railtie`.
4. `Railtie` is class based. Notice that the `new` method is private. What this means is that `Railtie` cannot be instantiated unless a subclass makes the `new` method public. This is one of the most interesting design decisions made by the Rails developers. In particular, it means that each subclass of `Railtie` operates without any instances.

#### Class Based Design

The last item I mentioned was the class based design of `Railtie`. To get a better sense of what this means, let's look at an example of how one would create a `Railtie`. Since the `new` method is private, you can't do the following:

{% highlight ruby %}
Rails::Railtie.new
{% endhighlight %}

This would throw a runtime error saying that `new` is private. What is done instead to define a `Railtie` is this:

{% highlight ruby %}
class MyNewRailtie < Rails::Railtie
  initializer "new_initialization_behavior" do
    puts "Hello!"
  end
end
{% endhighlight %}

Something subtle is happening here. The `new_initialization_behavior` initializer is being defined on the class instead of on any particular instance of `MyNewRailtie`. Normally this would mean that all instances would have access to this initializer, but since you can't instantiate `MyNewRailtie`, we don't really worry about that.

Instead, a singleton instance is created behind the scenes whenever you want to change the configuration of `MyNewRailtie`. Let's examine the following code:

{% highlight ruby %}
class AnotherNewRailtie < Rails::Railtie
  config.before_configuration do
    puts "This runs before the initializers run"
  end
end
{% endhighlight %}

We're changing the configuration of `AnotherNewRailtie`, but we're doing this by accessing the singleton of the `AnotherNewRailtie` class. In the code, the singleton is called the `instance`. It's defined as follows:

{% highlight ruby %}
def instance
  @instance ||= new
end
{% endhighlight %}

Thus, this method grabs either the cached object in `@instance` or creates a new `AnotherNewRailtie` object and sets `@instance` to this new object.

All `config` calls on the class are sent to the instance, which will then configure the class. When you call `AnotherNewRailtie.config`, you will obtain the configuration from the singleton `instance`, with the `before_configuration` hook that we defined above.

### Discussion

Although the class based design of `Railtie` gives you a simple syntax for configuring subclasses of `Railtie` and referencing a particular subclass (just get it's constant), it does run into some problems.

The biggest problem is that you can't easily create multiple `Rails::Application` instances. The `Rails::Application` class defines all of the backend logic for a Rails application, but you can't make multiple applications unless you create multiple subclasses (which isn't easy to do programatically) or unless you make the `new` method public (which destroys the API nature of `Railtie`).

My [GSOC project][gsocproject] will be attempting to fix some of these things, so stay tuned!

[gsoc]:             http://www.google-melange.com/gsoc/homepage/google/gsoc2013
[gsocproject]:      http://www.google-melange.com/gsoc/proposal/review/google/gsoc2013/wangjohn/1
[railssource]:      https://github.com/rails/rails
[railtiefile]:      https://github.com/rails/rails/blob/master/railties/lib/rails/railtie.rb
