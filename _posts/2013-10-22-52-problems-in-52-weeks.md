---
layout: post
title: "52 Problems in 52 Weeks"
date: 2013-10-22 14:47:00
categories: problems
---

Today I've decided to start an endeavor to write about 52 computer science problems in 52 weeks. I will provide a problem statement and attempt to provide a solution to the problem. I'll be selecting problems that I think are interesting/fun or have some unique solution and try to explain them as clearly as possible.

My first problem will be an old problem that I love telling other people.

# Problem 1: The Yellow Submarine

The Rolling Stones have learned that their rival band, The Beatles, have an invisible yellow submarine that they've been hiding in. This yellow submarine lives on the integer number line and has some constant integer velocity `v` with some integer starting position `p`. At each discrete time step, the yellow submarine moves by the velocity `v`.

The Rolling Stones would like to destroy the yellow submarine. At each time step, the Stones have a bomb they can detonate on one integer on the number line. If the Rolling Stones detonate the bomb on the integer which the submarine currently resides, then they win! Otherwise, they don't get any information if they miss the submarine.

Unfortunately, the Rolling Stones don't know `p` or `v`, they just know that both `p` and `v` are integers. Help the Stones devise an algorithm that will be guaranteed to destroy the yellow submarine in finite time.

For example, let's say the the yellow submarine has `p = 3, v = 5`. Then at time step `t = 0`, the submarine is on the integer `3`. At time step `t=1`, the submarine is on integer `8`. At time step `t=2`, the submarine is on integer `13`, etc. If the Rolling Stones' algorithm was to always shoot the integer 13, then the bomb would miss on timesteps `t=0,1` but the bomb would destroy the submarine on timestep `t=2`. Obviously this algorithm doesn't work for all values of `p` and `v` (consider any starting position `p != 13` with zero velocity `v = 0`).
