---
layout: post
title:  "Rails Coding Conventions"
date:   2013-08-26 22:28:49
categories: activerecord rails associations
---

I haven't found an official guide for what Ruby code should look like in the Rails master branch. When committing patches to [Ruby on Rails][railsgithub], you're basically expected to follow an implicit coding style: "When in Rome, do as the Romans do."

There is a tiny [official style guide][styleguide] for pull requests to Rails, but it doesn't cover all of the things you might find while writing Ruby code. In this post, I'll go through a couple of conventions that I've found are used throughout Rails code.

## Ruby Conventions

1. Prefer `define_method` to `class_eval` when making dynamic method definitions. Aaron Patterson (Tenderlove) posted a [blog post][tenderlovemethoddef] about making dynamic method definitions, making some benchmarks and showing that in general `define_method` is faster and cleaner.
2. Provide comments and examples at the beginning of the class and minimize comments in the method definitions themselves. Comments inside of method definitions tend to get stale very quickly because of changing implementations. However, the Rdoc on the specification of the class doesn't change as often (only when the API changes), so comments there are more likely to be read and stay consistent.
3. A variable that can holds a reference to a class should usually be named `klass`. This is because `class` is a reserved keyword. See [this][klassdefinition] Stack Overflow article about it.
4. Don't use the `return` keyword unless you are exiting from a method prematurely. Since the last result in a method is always returned in Ruby, it is redundant to add a return statement at the end of a method.


## Rails Specific Conventions

1. Create new modules by autoloading. For example, [Active Record][activerecorddef] creates all of its modules by defining a module and naming the file in which it lives to be the underscored name of the module, then using `autoload` inside of the base class.


[railsgithub]: https://github.com/rails/rails
[styleguide]: http://edgeguides.rubyonrails.org/contributing_to_ruby_on_rails.html#follow-the-coding-conventions
[tenderlovemethoddef]: http://tenderlovemaking.com/2013/03/03/dynamic_method_definitions.html
[activerecorddef]: https://github.com/rails/rails/blob/master/activerecord/lib/active_record.rb
[klassdefinition]: http://stackoverflow.com/questions/4299289/what-is-the-difference-between-class-and-klass-in-ruby