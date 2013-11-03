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
  node1, node2 = vertex1, vertex2
  while node1 != node2:
    node1 = vertex1.parent
    node2 = vertex2.parent

  return node1
{% endhighlight %}

A second simplification to this problem is as follows: given a finite tree, find the lowest common ancestor of any two nodes. Notice that we've dropped the constraint of finding the lowest common ancestor of two nodes that are on the same level in the tree. This simplification isn't too hard to solve given our previous knowledge.

We just need to turn the first problem into the second. We can do that by climbing all the way up to the root, figuring out the height of `vertex1` and `vertex2`, setting the heights the same, then calling `lca_same_level` on the resulting nodes. Let's write this up:

{% highlight python %}
def find_height(vertex):
  node = vertex
  height = 0
  while node.parent != None:
    height += 1
    node = node.parent

  return height

def climb_parents(vertex, num_parents):
  node = vertex
  for i in xrange(num_parents):
    if node.parent != None:
      node = node.parent

  return node

def lca_finite_tree(vertex1, vertex2):
  h1, h2 = find_height(vertex1), find_height(vertex2)

  if h2 < h1:
    vertex2 = climb_parents(vertex2, h1 - h2)
  elif h2 > h1:
    vertex1 = climb_parents(vertex1, h2 - h1)

  return lca_same_level(vertex1, vertex2)
{% endhighlight %}
