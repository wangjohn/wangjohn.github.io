---
layout: post
title:  "Active Record Configurations"
date:   2013-09-10 22:34:12
categories: activerecord rails railties
---

For my [Google Summer of Code][gsoc] project for Ruby on Rails, I've been trying to reorganize how Rails get initialized. My ultimate goal is to provide support for starting up multiple applications in a single Ruby process. Getting to this point, however, requires quite a few changes to the Rails codebase beforehand.

Currently, there are a bunch of configurations that are set up through `ActiveRecord::Base` and are shared across Active Record. These configurations are global in a Ruby process and Rails application (since currently there is only a notion of a single application per process). Since the design of Rails originally only called for a single application, having these global configurations isn't a really big deal. It's only when you start trying to configure multiple applications at the same time when things become a headache.

# Setting Configurations

The current state of configurations in Active Record isn't very pretty. Let's walk through the process of configuring the `pluralize_table_names` property. First, what you'll do is change `application.rb` file:

{% highlight ruby %}
module MyApplication
  class Application < Rails::Application
    config.active_record.pluralize_table_names = false
  end
end
{% endhighlight %}

We've now set the Active Record configuration so that `pluralize_table_names` is `false`. Let's see how this gets sent to the correct place in the Rails codebase.

## The Active Record Railtie

Once we've set `config.active_record.pluralize_table_names`, the Active Record Railtie will try to set the configurations on `ActiveRecord::Base` correctly. A railtie is basically a class used to pull in separate components inside of Rails. The Active Record railtie makes sure that all of the necessary classes and modules are required when `ActiveRecord` is loaded, and also that the correct initializers are set. The initializers are basically blocks of code that are run as soon as `ActiveRecord` is pulled into the Rails application.

If we look at a high-level overview of the Active Record Railtie, we'll find something like this:

{% highlight ruby %}
module ActiveRecord
  class Railtie < Rails::Railtie
    initializer "active_record.set_configs" do |app|
      ActiveSupport.on_load(:active_record) do
        app.config.active_record.each do |k,v|
          send "#{k}=", v
        end
      end
    end
  end
end
{% endhighlight %}

Basically, what this is saying is that Rails should take each of the configurations that is held on `app`, the application being initialized, and use attribute writers to set their values correctly. In our example, one of the key values on `app.config.active_record` would have been `pluralize_table_names`. This means, at some point in the loop, we would have set the following:

{% highlight ruby %}
  send "pluralize_table_names=", false
{% endhighlight %}

This would change the attribute on `ActiveRecord::Base`, so that `ActiveRecord::Base.pluralize_table_names == false`.

## Running the Initializers (Addendum)

Note that the application in the initializer which is referred to as `app` is the application which is being initialized. If you check out the `initialize!` method on the `Rails::Application` class, then you have the following definition:
{% highlight ruby %}
def initialize!(group=:default) #:nodoc:
  raise "Application has been already initialized." if @initialized
  run_initializers(group, self)
  @initialized = true
  self
end
{% endhighlight %}

The initializers are run with `self`, which in this context is the application being initialized. For example, I might call `initialize!` when booting up my application by using `MyApplicationName.initialize!`.

# Application Record

So now that we better understand configurations in Active Record, what is wrong, and how can we change it?

[gsoc]: http://www.google-melange.com/gsoc/homepage/google/gsoc2013
