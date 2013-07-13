---
layout: post
title:  "Global Configurations"
date:   2013-07-12 18:35:37
categories: rails gsoc
---

One of the first things you learn as a programmer is that global configurations and variables are evil. Personally, it was hard to get someone to give me a convincing argument as to why this is true. In this post, I'll provide my explanation for the evils of global configs and also some instances when you might want to use them.

### Intro to Global Configs

First of all, what are global configurations? They're data that can be accessed anywhere from within a program or application. For example, if I were writing Ruby code, I could make myself a configuration class like so:

{% highlight ruby %}
class Configuration
  attr_accessor :name, :birthday, :hometown

  def initialize
    @name = nil
    @birthday = nil
    @hometown = nil
  end
end
{% endhighlight %} 

This class could store the configuration for some user. Now, if I wanted to make a global configuration and use it throughout my Ruby code, I could do something along these lines:

{% highlight ruby %}
class GlobalConfig
  class << self
    def config
      @config ||= Configuration.new
    end
  end
end
{% endhighlight %}

This `GlobalConfig` class stores a single `Configuration` object and can be accessed from anywhere within my Ruby code (since I haven't scoped it at all). This will allow me to be inside of some class or module, yet still be able to access the `GlobalConfig`. We could have something like this:

{% highlight ruby %}
module Rails
  module SomeSubModule
    class Party
      def self.date
        GlobalConfig.config.birthday
      end
    end
  end

  class Map
    def self.location
      GlobalConfig.config.hometown
    end
  end

  class dashboard
    def self.hometown
      GlobalConfig.config.hometown
    end
  end
end
{% endhighlight %}

Notice that all of these classes can access the singleton `Configuration` object that is stored in `GlobalConfig`. This is essence of a global configuration: being able to access a group of data from anywhere.

### Upsides and Downsides to Global Configurations

#### Upsides

Although people say that global configurations are evil, actually they are sometimes quite useful.

1. Global configurations are really easy to use. You can create a single class with everything you'd ever want to configure, and this allows you to code without worrying about configurations elsewhere.
2. Global configurations keep everything in one place. This means if you want to configure something, you know exactly where to go to do it.
3. Global configurations lend themselves to a single logical unit of something. If you know that there will only ever be one of something you are making, then global configurations are exactly what you want since they fit logically with a single object. For example, if you are making software that flies an airplane, you can be pretty sure that you won't have to handle configurations for multiple airplanes.

#### Downsides

However, you might want to stray away from global configurations most of the time. It is rare that you will ever have one of something. Even if you think this may be the case at the beginning of your project, going down the road of global configurations makes it really hard for your software to be extensible in the future if you ever decide to relax those constraints.

1. Changing global configurations can be deadly since you don't necessarily know who's using what.
2. Debugging configurations is hard. If one of your configurations causes a bug, many places could be accessing that configuration. Understanding how a configuration is used becomes more difficult with more code.
3. Information overload. Having everything in one place can prevent you from seeing what each configuration does. This is especially bad since most configurations require context.
4. By definition, global configurations disallow multiple configurations. This prevents you from creating multiple instances of something.

Global configurations can be a crutch, since they allow you to solve the configuration problem quickly and easily. However, you might be banging your head against your desk in the future. In a small codebase, where you know about mostly everything, global configurations are fine. However, as the complexity of your code increases, keeping a global configuration can be a headache.
