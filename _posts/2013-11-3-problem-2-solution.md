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

The simplest problem that we could solve would probably be a finite tree with our two vertices `v~1~` and `v~2~` being on the same level of the tree. In this case, you can just sequentially walk up with each reference until the two references are on the same object. Assuming we are given two vertices which must be in the same tree, at some point the two references will necessarily be the same. Some python pseudocode is given below:

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
    vertex2 = climb_parents(vertex1, h1 - h2)
  elif h2 > h1:
    vertex1 = climb_parents(vertex2, h2 - h1)

  return lca_same_level(vertex1, vertex2)
{% endhighlight %}

# Full Solution

Now we're ready to tackle the original problem. Given an infinite tree and any two vertices in that tree, get the lowest common ancestor of the two vertices.

We could try to use our solution for the finite tree, but we can quickly see that it won't work. This is because it might take infinitely long for us to calculate the height of each vertex. Obviously, this is unacceptable.

However, let's think about what we did in our previous solutions that might help us. We basically held references to two nodes in the tree, and we checked if those were equal. When we started on the same level, we could reason pretty easily that we'd eventually have references to the same node.

Now, let's think about how we can eventually get references to the same node to happen in an infinite tree. Well, we know that if we just walk for a long time with our two vertices, both of our references will eventually be to ancestors (albiet potentially not the same ancestor).

Now, we can ask ourselves, is there a way to get two references for the same node? If there is, then we automatically know that this node is an ancestor. Moreover, if we were counting how many parent pointers each reference climbed to get to the ancestor, we would be able to find the lowest common ancestor easily. We just need to set references to the same level in the tree, then climb until we reach the lowest common ancestor (much like what we did in our simplification).

{% highlight python %}
def climb_to_lca(vertex1, vertex2, height1, height2):
  if height1 > height2:
    return climb_parents(vertex1, height1 - height2)
  elif height1 < height2:
    return climb_parents(vertex2, height2 - height1)
  else:
    return vertex1
{% endhighlight %}

Now, the only question is, how do we get to an ancestor? If we climb the parents of the references in a uniform fashion (i.e. climb parents for node 1 and 2 at the same time), we will only converge on an ancestor if both starting vertices were at the same level in the tree. What we would really want to do is to climb to one node 1's parent, then scan over all parents on node 2 until we reach the root. After that, node 1 can climb up a parent again, and node 2 can scan all of its parents again. We can continue this until nodes 1 and 2 refer to the same node. This algorithm will work for finding an ancestor in a finite tree. However, it is not possible to scan through an infinite tree. Thus, we need a smarter way to do this scanning.

One way we can do this by using an age old computer science concept: galloping. The idea behind galloping is that you can search for numbers by increasing the index of the number you are checking according to a geometric progression. For example, if you were searching a sorted list for a particular number, you could search indices 0, 1, 2, 4, 8, 16, etc. until you have narrowed down the range of numbers you need to check.

So how can we apply galloping in this problem? We notice that climbing parent references at a constant rate won't work -- therefore we might want to apply some sort of galloping for climbing parent references. Imagine you got the parent of the references sequentially in a galloping format. For example, let's use this progression:

(n1, 1), (n2, 2), (n1, 4), (n2, 8), (n1, 16), ...

In the above, `node1` will climb 1 parent reference while `node2` doesn't change its reference. Then, `node2` will climb 2 parent references while `node1` doesn't change its reference, etc. Each time a reference changes, we will check to see if the references are the same. If they are, we can stop climbing because we've found an ancestor.

The pseudocode for the algorithm is below:

{% highlight python %}
def find_ancestor(vertex1, vertex2, step = 2):
  climbing_node, resting_node = vertex1, vertex2
  count1, count2 = 0, 0
  num_steps = 0

  while True:
    moves_to_make = step ** (num_steps)
    for i in xrange(moves_to_make):
      if climbing_node == resting_node:
        return climb_to_lca(vertex1, vertex2, count1, count2)
      climbing_node = climbing_node.parent

    if num_steps % 2 == 0:
      count1 += moves_to_make
    else:
      count2 += moves_to_make

    climbing_node, resting_node = resting_node, climbing_node
    num_steps += 1
{% endhighlight %}

This algorithm will find the lowest common ancestor, and will do so in `O(k)` time, where `k = max(d(v~1~, l), d(v~2~, l))` is the maximum height between a starting vertex and the lowest common ancestor `l`. The correctness of the algorithm comes from the fact that at some point, the difference between the two vertices will be covered.

Let's look at correctness a little more. Let `j = ceiling(lg(2k))` be the smallest number such that `2^j^ > 2k`. After we have done `j` alternating gallops, it must be the case that we'll find an ancestor on the next gallop (in fact most likely before, but all we need to do is show that we'll eventually find an ancestor). This is because in the next gallop, the node which is moving must at some point reach the stationary node. Note that the difference in the levels of vertex 1 and 2 is at most `k` so that the two vertices are within `k` nodes of each other. Since the `j`th gallop covers `2k` nodes, it must be the case that during the `j`th gallop, the two nodes meet. This works because there is the invariant that the nodes are never more than `2^j^` further apart from each other than they originally started.

This analysis also shows that the runtime is `O(k)` because you never do more than `O(2^j+1^) = O(k)` parent traversals.

# Connections
[Timsort][timsort] (the sorting mechanism behind Python) actually uses galloping when merging two sorted lists.

[timsort]: http://en.wikipedia.org/wiki/Timsort
