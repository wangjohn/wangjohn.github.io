---
layout: post
title:  "Introduction to Railties"
date:   2013-07-10 21:59:37
categories: railties rails gsoc
---

As part of my [Google Summer of Code][gsoc] project I'll be refactoring Railtie, the internal engine of Ruby on Rails. When a Rails application launches, there is a complicated process of configuration and initialization that sets the Rails web server.

Before you can actually set up this web server, there are quite a few things that need to be configured. Railties are the engines for reading these configurations, creating an application, and connecting different parts of Rails.

### The Railtie Class

All major components of Rails are created as a subclass of Rails::Railtie. For example, Active Record (which handles query syntax to a database) and Action Controller (which handles the base controller logic) are both Railties.

#### So what does Rails::Railtie do?

Well, to figure out the function of Railtie, we can go look at the source code. Finding the definition isn't too tricky. You can clone the Ruby on Rails source code from [github][railssource]. My primary method for finding class definitions is by using ack-grep, a convenient command line tool for searching through files. I searched for the following:

{% highlight bash %}
ack-grep "class Railtie"
{% endhighlight %}

This gives us the file which ["railties/lib/rails/railtie.rb"][railtiefile] which defines Railtie. A casual perusal of the file tells us a couple of things:

1. Railtie is abstract. This means that it is not intended for direct instantiation. This is pretty clear because there is an error which is thrown inside of the initialize method whenever the class attempting to be initialized is abstract. Also, the new method is private which is a pretty big hint.
2. Railtie has three main methods.
3. Railtie is based on a class.

[gsoc]:             http://www.google-melange.com/gsoc/homepage/google/gsoc2013
[railssource]:      https://github.com/rails/rails
[railtiefile]:      https://github.com/rails/rails/railties/lib/rails/railtie.rb
