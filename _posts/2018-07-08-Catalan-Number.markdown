---
layout: post
title:  "Catalan Number"
date:   2018-07-08 20:34:34 +0800
author: allencharp
tags: [math, algorithms, combinatorics]
---

# Description

The [Catalan number](https://en.wikipedia.org/wiki/Catalan_number) is a sequence of natural numbers that appears in many counting problems, such as the number of structurally unique binary search trees (BSTs) with *n* nodes.

The first few Catalan numbers are: 1, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, 16796, 58786, ...

![Catalan formula](https://wikimedia.org/api/rest_v1/media/math/render/svg/34d4f28865115a05a806649a40f84e1bbc736320)

# Recurrence and closed form

The *n*-th Catalan number satisfies the recurrence

```
C(0) = 1
C(n) = Σ C(i) · C(n-1-i)   for i = 0 .. n-1
```

which follows from choosing the root of a BST and counting the left and right subtrees independently. The closed form is

```
C(n) = (2n)! / ((n + 1)! · n!) = (1 / (n + 1)) · C(2n, n)
```

# Where Catalan numbers appear

* Number of structurally unique BSTs with *n* nodes
* Number of valid parentheses combinations with *n* pairs
* Number of ways to triangulate a convex polygon
* Number of full binary trees with *n+1* leaves
* Number of possible stack pop sequences of length *n*

# Implementation

A simple iterative implementation in Python:

```python
def catalan_number(n):
    nm = dm = 1
    for k in range(2, n + 1):
        nm, dm = (nm * (n + k), dm * k)
    return nm // dm
```

Analysis: the loop runs `n-1` times with constant work per iteration, so the time complexity is **O(n)** and space is **O(1)** (ignoring the size of the big integers). `//` keeps integer division, which is exact here.

# LeetCode Practice

1. [Unique Binary Search Trees](https://github.com/allencharp/LeetCode/blob/master/UniqueBinarySearchTrees.py) — how many structurally unique BSTs store *n* nodes?
2. [Different Ways to Add Parentheses](https://github.com/allencharp/LeetCode/blob/master/DifferentWaystoAddParentheses.py) — a Catalan-flavored divide-and-conquer problem.

# Summary

Catalan numbers bridge recursion, combinatorics and dynamic programming: the recurrence maps directly to DP solutions (BST counting, parentheses generation), and the O(n) iterative formula is a neat optimization when only the final count is needed.
