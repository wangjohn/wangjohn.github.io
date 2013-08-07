---
layout: post
title:  "Isolated Namespaces"
date:   2013-08-07 22:49:10
categories: railties rails gsoc engines isolation namespaces
---

One interesting feature that Ruby on Rails provides is the ability to isolate an engine from the main application. When creating a new engine, one can use the `isolate_namespace` method to keep the methods from outside a module from being available to that module. The syntax for using this is as follows:

{% highlight ruby %}
module MyEngine
  class Engine < Rails::Engine
    isolate_namespace MyEngine
  end
end
{% endhighlight %}

# Overview of Namespace Isolation

The Rails [documentation][docs] provides a good description of what namespace isolation actually accomplishes. The main things are:

1. Isolated engines only have access to its own `helpers` and `url_helpers`.
2. Routes will automatically be namespaced correctly. Inside of an isolated engine, you don't have to worry about model prefixes. For example, you don't have to append `my_engine` before calls to ``MyEngine::SomeModelName``.
3. Table name prefixes are set correctly. If you have a model like ``MyEngine::Article`` and ``MyEngine`` has been isolated, then the table name that ``MyEngine::Article`` accesses will be ``my_engine_articles``.

# Implementation of `isolate_namespace`

[docs]: http://edgeapi.rubyonrails.org/classes/Rails/Engine.html#label-Isolated+Engine
