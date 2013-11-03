---
layout: post
title: "Problem 2 Solution"
date: 2013-11-3 16:43:00
categories: problems, trees, algorithms
---

# Introduction

I'm going to first give some simplifications to solve this problem. There are a number of ways to think about solving this problem, so we'll first get some intuition for solving it through these simplifications.

After the simplifications, we'll apply some concepts that are used a lot in solving computer science problems to help us solve this problem.

# Simplifications

The simplest problem that we could solve would probably be a finite tree with our two vertices `v~1~` and `v~2~` being on the same level of the tree. In this case, you can just sequentially walk up with each pointer until the two pointers are on the same element. Assuming we are given two vertices which must be in the same tree, at some point the two pointers will eventually be the same. Some python pseudocode is given below:

{% highlight python %}
def lca_same_level(vertex1, vertex2):
  pointer1, pointer2 = vertex1, vertex2
  while pointer1 != pointer2:
    pointer1 = vertex1.parent
    pointer2 = vertex2.parent

  return pointer1
{% endhighlight %}

