---
layout: post
title:  "Application Record"
date:   2013-09-10 22:34:12
categories: activerecord rails railties
---

For my [Google Summer of Code][gsoc] project for Ruby on Rails, I've been trying to reorganize how Rails get initialized. My ultimate goal is to provide support for starting up multiple applications in a single Ruby process. Getting to this point, however, requires quite a few changes to the Rails codebase beforehand.

Currently, there are a bunch of configurations that are set up through `ActiveRecord::Base` and are shared across Active Record. These configurations are global in a Ruby process and Rails application (since currently there is only a notion of a single application per process). Since the design of Rails originally only called for a single application, having these global configurations isn't a really big deal. It's only when you start trying to configure multiple applications at the same time when things become a headache.

# Active Record Configurations

The current state of configurations in Active Record isn't very pretty. Let's walk through the process of configuring the `pluralize_table_names` property. First, what you'll do is change `application.rb` file:

{% hightlight ruby %}
module MyApplication
  class Application < Rails::Application
    config.active_record.pluralize_table_names = false
  end
end
{% endhighlight %}

We've now set the Active Record configuration so that `pluralize_table_names` is `false`. Let's see how this gets sent to the correct place in the Rails codebase.

## The Active Record Railtie



[gsoc]: http://www.google-melange.com/gsoc/homepage/google/gsoc2013
