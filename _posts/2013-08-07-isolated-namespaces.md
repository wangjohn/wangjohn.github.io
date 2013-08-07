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

# Implementation of isolate_namespace

Implementing the `isolate_namespace` method actually requires only a small amount of code. The entirety of the method definition can be found [here][isolatedef]. We're going to step through the method and understand how each part of the code works.

## Generating Engine Name

First, the method sets the `engine_name` by invoking `generate_railtie_name` on in module that was passed as an argument. This adds underscores to the module name. This means that whenever you call `engine_name` on the engine, you will receive an underscored and nicely readable representation of the module. For example:

{% highlight ruby %}
MyEngine::Engine.engine_name # => my_engine
{% endhighlight %}

## Railtie Namespace

Second, the `isolate_namespace` method defines a number of new singleton methods on the module. The first such method is created by code which looks like this:

{% highlight ruby %}
unless mod.respond_to?(:railtie_namespace)
  name, railtie = engine_name, self

  mod.singleton_class.instance_eval do
    define_method(:railtie_namespace) { railtie }
  end
end
{% endhighlight %}

Here, the `railtie` variable refers to the `Engine` class inside of which the namespace is being isolated. These lines create a new singleton method `railtie_namespace` on the module. You can now see something like this:

{% highlight ruby %}
MyEngine.railtie_namespace  # => MyEngine::Engine
{% endhighlight %}

Methods for `table_name_prefix` and `use_relative_model_naming?` are similarly created on the singleton module.

## Naming of Active Models

Once these methods have been defined, one can check whether these methods exist. For example, inside of the ``ActiveModel::Naming`` module, we have the following logic:

{% highlight ruby %}
def model_name
  @_model_name ||= begin
    namespace = self.parents.detect do |n|
      n.respond_to?(:use_relative_model_naming?) && n.use_relative_model_naming?
    end
    ActiveModel::Name.new(self, namespace)
  end
end
{% endhighlight %}

Thus, when Active Model looks for the name of a model, it will first detect whether or not is uses relative namespacing. The relative namespacing could be defined by using `isolate_namespace` inside of an engine. The Active Model logic is in charge of checking to see whether a relative namespace should be used.

This paradigm is used throughout `isolated_namespace`. It involves defining a method which can be thought of as a configuration, then checking whether the method exists in the appropriate methods that need this configuration. For example, the paradigm is used for making sure that the helper methods in an isolated engine are isolated correctly.

## Helper Methods

The following two methods are defined on the singleton module:

{% highlight ruby %}
unless mod.respond_to?(:railtie_helpers_paths)
  define_method(:railtie_helpers_paths) { railtie.helpers_paths }
end

unless mod.respond_to?(:railtie_routes_url_helpers)
  define_method(:railtie_routes_url_helpers) { railtie.routes.url_helpers }
end
{% endhighlight %}

Although these may not seem to do much, they actually help Action Controller pull in the correct `helpers_paths` and `url_helpers` from the isolated engine. To see how these two methods work with Action Controller, we need to look at the `RoutesHelpers` module, as well as the Action Controller railtie which sets the configuration of `ActionController`.

The `RoutesHelpers` module defines a single method which looks like this:

{% highlight ruby %}
module RoutesHelpers
  def self.with(routes)
    Module.new do
      define_method(:inherited) do |klass|
        super(klass)
        if namespace = klass.parents.detect { |m| m.respond_to?(:railtie_routes_url_helpers) }
          klass.send(:include, namespace.railtie_routes_url_helpers)
        else
          klass.send(:include, routes.url_helpers)
        end
      end
    end
  end
end
{% endhighlight %}

This defines a module which has an inherited hook (i.e. when the module is inherited, the `inherited` method is run). Inside of this hook, we can see that all of the modules inside of `routes.url` are included in the base class if there is no parent class which has defined the `railtie_routes_url_helpers` method. Otherwise, all of the modules inside of `namespace.railtie_routes_url_helpers` are included.

Thus, under normal circumstances when `isolated_namespace` is not used, using `RoutesHelpers.with` will return a module which, once inherited, will include the `url_helpers` from the argument to `RoutesHelpers.with`. If the `railtie_routes_url_helpers` method is defined for some parent of the base class which is inherited, then the modules defined in this class will be inherited.

Put more simply, if `isolate_namespace` has been used, then `RoutesHelpers.with` will detect the helpers that it needs to pull in based on whether or not a singleton method exists. This singleton method holds the correct helpers if `isolate_namespace` has been used. If the singleton method has not been defined, then `RoutesHelpers.with` falls back getting the normal helpers.

Inside of the Action Controller initializer, this machinery is put to good use in something that looks like this:

{% highlight ruby %}
initializer "action_controller.set_configs" do |app|
  ActiveSupport.on_load(:action_controller) do
    extend ::AbstractController::Railties::RoutesHelpers.with(app.routes)
  end
end
{% endhighlight %}

This finishes off the process of isolating the helpers by inheriting the module returned by `RoutesHelpers.with`.

[docs]: http://edgeapi.rubyonrails.org/classes/Rails/Engine.html#label-Isolated+Engine
[isolatedef]: https://github.com/rails/rails/blob/master/railties/lib/rails/engine.rb#L374-L403
