---
layout: post
title:  "Introduction to Railties"
date:   2013-07-10 21:59:37
categories: railties rails gsoc
---

As part of my [Google Summer of Code][gsoc] project I'll be refactoring Railtie, the internal engine of Ruby on Rails. When a Rails application launches, there is a complicated process of configuration and initialization that sets the Rails web server.

Before you can actually set up this web server, there are quite a few things that need to be configured. Ruby on Rails was developed under the ["convention before configuration"][conventionconfig] mantra. What this means for you is that most of the configuration that needs to be done for a Rails web server has already been set - the defaults should take care of everything.

Railties are the engines for reading these configurations, creating an application, and connecting different parts of Rails.

### The Railtie Class

All major components of Rails are created as a subclass of Rails::Railtie.

[gsoc]:             http://www.google-melange.com/gsoc/homepage/google/gsoc2013
[conventionconfig]: http://www.google.com
