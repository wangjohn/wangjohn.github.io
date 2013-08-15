---
layout: post
title:  "Automatic Inverses"
date:   2013-08-14 16:35:43
categories: activerecord rails associations
---

Have you ever wondered whether Rails does anything smart when you have two associations which are inverses of each other? Does an `ActiveRecord` object get loaded twice if it comes from two different associations? Consider the following:

{% highlight ruby %}
class Post < ActiveRecord::Base
  has_many :comments
end

class Comment < ActiveRecord::Base
  belongs_to :post
end
{% endhighlight %}

With the models that we've made above, we can go ahead and find an object from both associations like so:

{% highlight ruby %}
comment = post.comments.first
comment.post = nil
post.comments.include?(comments)
{% endhighlight %}

In this case, we notice that `comment.post` and `post` should belong to the same database object. But, is Rails smart enough to know that the comment should be removed from both of the associations? Or are `comment.post` and `post` different representations of the same database row?

# Identity Map

It used to be that the Identity Map was a step in the right direction. The Identity Map used to poke its head around back in Rails 3.2. Basically, it kept an in-memory store of all Active Record objects. Rails would go to the Identity Map and check for the presence of an object with a particular id before going to the database. This ensured that all database objects could be accessed and was supposed to increase performance by not having to go to the database twice.

However, it caused a bunch of problems. For example, consider the following:

{% highlight ruby %}
class Post < ActiveRecord::Base
  has_many :comments, dependent: :destroy
end

comment = post.comments.first
comment.post = nil
comment.save
Post.destroy(post.id)
{% endhighlight %}

Without the Identity Map, the post would be destroyed while the comment object would still be intact. Unfortunately, once the Identity Map is turned on, the post object being loaded by `Post.destroy(post.id)` is the exact same object as `post`, which means the comment object will also be removed.

This is inconsistent behavior, and so the Identity Map was removed in Rails 4.

# Inverse Association Definitions

In order to get correct behavior, yet still performance, the `inverse_of` option was created in Rails 4. This allowed you to define associations like so:

{% highlight ruby %}
class Post < ActiveRecord::Base
  has_many :comments, inverse_of: :post
end

class Comment < ActiveRecord::Base
  belongs_to :post, inverse_of: :comments
end
{% endhighlight %}

Once the following relationships are set, then the `c = posts.comments.first` and `p = c.post` actually become the same in-memory objects. The `inverse_of` option let's both associations know that they are inverses of each other. Thus, each association can check whether the Active Record object that is being queried is in its inverse.

This enhances performance by eliminating an extra call to the database for an object that has already been loaded in memory, but also gracefully handles the situation that the Identity Map wasn't able to handle correctly. 

Unfortunately, the developer had to set the `inverse_of` option for every such relationship, which requires much more syntax. This was fixed in a relatively recent patch, which I will discuss in detail.

# Automatic Inverse Associations

The need to declare inverses is especially annoying since most times, the relationships are really clear. One would think that a minimal amount of code would be able to handle 80% of use cases. In fact, this is the case.

This [patch][commit] makes `inverse_of` detection automatic and should work for most use cases. This means you no longer have to declare `inverse_of` on two associations which have good names. For example, the `inverse_of` declaration is no longer required in the examples used above with the `Post` and `Comment` model.

I'll first discuss how the automatic inverse detection works, then talk about its weaknesses.

## Heuristics, Baby!

If you look at the [commit][commit] made to the Rails master branch, you can see exactly what was done. The meat of the logic lies in `set_automatic_inverse_of` method. The other changes in the commit are related to caching the inverse association (whether or not it is found).

The simple heuristic used by the code is to take the downcased name of the Active Record relation and see if there exist any associations of the same name. The simplified code is here:

{% highlight ruby %}
def set_automatic_inverse_of
  if can_find_inverse_of_automatically?(self)
    inverse_name = active_record.name.downcase.to_sym

    begin
      reflection = klass.reflect_on_association(inverse_name)
    rescue NameError
      # No reflection found, perform logic for caching this negative result
    end

    # Perform logic for caching the reflection 
  end
end
{% endhighlight %}

This super simple heuristic works almost all of the time, since it is likely that if you are defining an inverse relationship, then you will be defining the inverse on the Active Record class you are currently a part of. For example:

{% highlight ruby %}
class Car < ActiveRecord::Base
  belongs_to :police_officer, inverse_of: :car
end
{% endhighlight %}

Here, the downcased name of the Active Record relation `Car` provides the exact name of the inverse relationship. The `set_automatic_inverse_of` method is only called once (as the result is cached for future use). This is especially important because of the `to_sym` method called on the downcased Active Record relation (which is expensive when called too many times).

The rest of the logic makes sure that this simple heuristic will be guaranteed to work. The `can_find_inverse_of_automatically?` method makes sure that the options specified by the method are indeed valid for automatic inverse finding. If they are not, then the method falls back to not setting the inverse.

## Drawbacks to Simple Heuristics

The main drawback with this architecture is that it doesn't account for more complicated relationships. The `can_find_inverse_of_autoamtically?` method returns false if the association it is looking at is `polymorphic`. This is because complicated naming structures simply aren't accounted for.

The design tradeoff was simplicity over complexity. However, the simplicity of the heuristic meant that fewer edge cases could be handled (hence the inability to handle associations with `:conditions`, `:through`, `:polymorphic`, or `:foreign_key` options).

I would argue that this was a good tradeoff because developers who are creating more complicated associations would probably be aware of the manual ability to set `inverse_of`. The automatic inverse detection was mostly for a fast, simple optimization that wouldn't make too much of an up front performance hit.

[commit]: https://github.com/rails/rails/commit/26d19b4661f3d89a075b5f05d926c578ff0c730f
