---
layout: post
title: "Problem 2"
date: 2013-11-3 16:13:00
categories problems, trees, algorithms
---

# Introduction

As part of my goal to finish [52 computer science problems in 52 weeks][52probs], I'm going to start on the second problem.

This problem is more of an algorithms interview question that is concerned with navigating around trees.

# Setup

Denote a tree `T` as a graph where any two vertices are connected by exactly one simple path. Suppose that each vertex `v` in the tree has a pointer to its parent and also to each of its children. Note that this tree does not necessarily need to be a binary tree, or a search tree. 

An ancestor of two nodes `v_1` and `v_2` is a node `a` such that `a` can be reached from both `v_1` and `v_2` by recursively following parent pointers. The lowest common ancestor of `v_1` and `v_2` is the ancestor `a` such the distance `d(a, v_1) + d(a, v_2)` is minimized, where the distance is defined as the number of edges traversed to get between two vertices.

# Problem 2

The problem is now as follows: you are given two vertices `v_1` and `v_2` which you know reside in a tree `T` which has an infinite number of nodes in it. Find the lowest common ancestor of `v_1` and `v_2`.
