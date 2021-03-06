---
layout: post
title:  "Rails Coding Conventions"
date:   2013-08-26 22:28:49
categories: activerecord rails associations
---

While doing my [Google Summer of Code][gsoc] project for Ruby on Rails, I had some trouble figuring out best practices for committing code. I haven't found an official and comprehensive guide for what Ruby code should look like in the Rails master branch. When committing patches to [Ruby on Rails][railsgithub], you're basically expected to follow an implicit coding style: "When in Rome, do as the Romans do."

There is a tiny [official style guide][styleguide] for pull requests to Rails, but it doesn't cover all of the things you might find while writing Ruby code. In this post, I'll go through a couple of conventions that I've found are used throughout Rails code.

However, I should preface everything I'm about to say with this: you shouldn't change code that is already in the codebase to conform with the style guidelines if it obscures the git history. Often it is tempting to change spacing or blatantly broken style, but you should refrain from doing it unless you are sure about it and have talked to a Rails Core Team member. While having consistent styling across the code is important, it is arguably more important for people to be able to find things easily in the git history.

Note, these conventions are only my observations of what Rails code looks like. You shouldn't think of them as being comprehensive or even totally correct. The Rails codebase was built by thousands of contributors over many years, so if you really want to figure out how some piece of code should be styled, you should talk to a Core Team member.

## Ruby Conventions

1. Prefer `define_method` to `class_eval` when making dynamic method definitions. Aaron Patterson (Tenderlove) posted a [blog post][tenderlovemethoddef] about making dynamic method definitions, making some benchmarks and showing that in general `define_method` is faster and cleaner.
2. Provide comments and examples at the beginning of the class and minimize comments in the method definitions themselves. Comments inside of method definitions tend to get stale very quickly because of changing implementations. However, the Rdoc on the specification of the class doesn't change as often (only when the API changes), so comments there are more likely to be read and stay consistent.
3. A variable that can holds a reference to a class should usually be named `klass`. This is because `class` is a reserved keyword. See [this][klassdefinition] Stack Overflow article about it.
4. Don't use the `return` keyword unless you are exiting from a method prematurely. Since the last result in a method is always returned in Ruby, it is redundant to add a return statement at the end of a method.
5. Don't use ternaries if they will take up more than one line. It's very rare to see a long ternary in the Rails codebase, and it's generally preferred to write out the if statement instead.
6. Try to namespace as much as possible. If you can, put something inside a well named module which can be easily ported around and included in other parts of the codebase. Modules, like their name implies, helps kee code modular and easy to read. Almost all of the Rails codebase is written in this modular format (except for a couple places in Railties).
7. Try to keep the chaining of enumerable methods at a reasonable level. No one likes to go through the code and have to disentagle a jungle of `.map`, `.each`, `filter`, etc. If you can, try to only chain one or two enumerable methods together, save the result in a well named variable, and continue performing operations on the new variable. This makes the code much easier to read.

## Rails Specific Conventions

1. Create new modules by autoloading. For example, [Active Record][activerecorddef] creates all of its modules by defining a module and naming the file in which it lives to be the underscored name of the module, then using `autoload` inside of the base class. Using `autoload` helps improve performance and also helps modularize the code.
2. Know and use Active Support helpers. There are a bunch of methods like `mattr_accessor` or `delegate` which are incredibly useful and defined in Active Support. Use these methods instead of trying to implement analogs on your own, since these methods are more well-known and probably have better performance.


[railsgithub]: https://github.com/rails/rails
[styleguide]: http://edgeguides.rubyonrails.org/contributing_to_ruby_on_rails.html#follow-the-coding-conventions
[tenderlovemethoddef]: http://tenderlovemaking.com/2013/03/03/dynamic_method_definitions.html
[activerecorddef]: https://github.com/rails/rails/blob/master/activerecord/lib/active_record.rb
[klassdefinition]: http://stackoverflow.com/questions/4299289/what-is-the-difference-between-class-and-klass-in-ruby
[gsoc]: http://www.google-melange.com/gsoc/homepage/google/gsoc2013
