---
layout: post
title: "Problem 1 Solution"
date: 2013-10-22 15:14:00
categories: problems problem1
---

# Introduction

In this post, I'm going to provide to solution to [problem 1][prob1].

The tricky part of this solution is that the set of all possible velocities and positions are integer numbers (both positive, negative, and zero). This means that moving back and forth with a bomb (like bombing integers 0, 1, -1, 2, -2, etc.) doesn't work because you could start with a high velocity and outpace the bomb.

Alternatively, just sitting on one value with the bomb doesn't work because the submarine could either skip over that position, have zero velocity, or just never approach that position at all.

# Simplifications

In order to solve the problem, I find it useful to first simplify the problem into something that is easy to solve. You can make two easy simplifications. You want to destroy the submarine (you still have 1 bomb per time step), under the following conditions:

  1. Submarine has `v=0` and unknown `p`.
  2. Submarine has `p=0` and unknown `v` (and you start shooting at timestep `t=1`).


## No Velocity, Unknown Position

In the first case, we can come up with a pretty simple strategy. We just need to enumerate all possible integers. In other words, we need to get an algorithm that will eventually hit every integer in finite time.

This algorithm is relatively simple: just send the bomb to the integers in the following sequence:

  0, 1, -1, 2, -2, 3, -3, ...

If you do this, then you will be guaranteed to hit every integer. This is because you're slowly moving out from 0 in both the positive and negative directions.

Now that we've solved the problem for this simple case, we have a lot more intuition for what we have to do to solve the problem more generally.

## Known Starting Position, Unknown Velocity

The next simplification we can solve is when we know the starting position, but don't know the velocity (we can't shoot on timestemp `t=0` otherwise the algorithm would be trivial and we could just shoot the submarine at it starting position - this doesn't really give us any intuition for the general answer). This case is really similar to the previous simplification. In fact, you can see it as just a transformation of variables.

To shoot down the submarine at time `t`, you just need to shoot the integer `tv`. We know this because the position of the submarine at time `t` is determined exactly by the starting position of 0 and the velocity.

Therefore, in order to solve this case of the problem, we want to iterate over all possible velocities in the following sequence:

  0, 1, -1, 2, -2, 3, -3, ...

For example, at time `t=1`, we would try to shoot down the submarine if it had velocity `v=0`, so we would shoot `1*0 = 0`. At time `t=2`, we would try to shoot down the submarine if it had velocity `v=1`, so we would shoot `2*1 = 1`. We try all the possible velocities just like how in the previous problem we tried all the possible positions.

We will eventually blow up the submarine because for every possible velocity `v`, we will eventually try to blow up the position that the submarine would have with that velocity.

# Synthesis of Knowledge

Now that we've solved those two simpler problems, we can try to synthesize our knowledge to solve the more general problem. In particular, we've noticed a couple of things:

    1. We can iterate over all possible integers in such a way that we will reach any arbitrary integer in finite time. We just need to use the sequence [0, 1, -1, 2, -2, ...].

    2. We can test to see if the submarine has a particular starting position and velocity at any timestep `t`, by bombing the integer `p + vt`. If the submarine has starting position `p` and velocity `v`, then the submarine will have been destroyed.

# Full Solution

These two observations give us all we need in order to solve the problem. In each of our simplifications, we were only iterating over one variable (either starting position or velocity). In our general problem, we need to iterate over two variables. In other words, we need to get all possible pairs `(p,v)` and iterate over all of them in such a way that we will reach any arbitrary pair `(p,v)` in finite time.

The next logical step, then, is to create a two-variable analog to our sequence of [0, 1, -1, 2, -2, ...]. Then, we can use our testing mechanism of shooting `p + tv` to check if the submarine indeed started out with the pair of conditions `(p,v)`.

To get the two-variable analog, just think about what we did with one-variable. We iterated back and forth on the number line, getting further and further out from 0. In two variables, instead of a single number line, we now have a plane with two axes. One axis represents starting position and the second axis represents velocity. We just need to iterate over all pairs.

It's now clear that we can just start at the tuple (0,0) and make concentric squares about the origin. As time increases, you trace out parts of these concentric squares which are further and futher out from the origin. Tracing out these concentric squares will guarantee that you eventually bomb any arbitrary tuple `(p,v)` in finite time, which solves the problem. In fact, the only thing the algorithm needs to do is iterate over all possible pairs `(p,v)` in two dimensions, so other shapes like a spiral about the origin would also work.

[prob1]: ({% post_url 2013-10-22-52-problems-in-52-weeks %})
