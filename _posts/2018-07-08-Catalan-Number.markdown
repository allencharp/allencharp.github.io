---
layout: post
title:  "Catalan Number"
date:   2018-07-08 20:34:34 +0800
author: allencharp
tags: [math, algorithms, combinatorics]
---

## Description

The [Catalan number](https://en.wikipedia.org/wiki/Catalan_number) is a sequence of natural numbers that appears in many counting problems, such as the number of structurally unique binary search trees (BSTs) with *n* nodes.

The first few Catalan numbers are: 1, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, 16796, 58786, ...

![Catalan formula](https://wikimedia.org/api/rest_v1/media/math/render/svg/34d4f28865115a05a806649a40f84e1bbc736320)

The *n*-th Catalan number can be computed as:

```
C(n) = (2n)! / ((n + 1)! * n!)
```

A simple iterative implementation in Python:

{% highlight python %}
def catalan_number(n):
    nm = dm = 1
    for k in range(2, n + 1):
        nm, dm = (nm * (n + k), dm * k)
    return nm // dm
{% endhighlight %}

## LeetCode Practice

1. [Unique Binary Search Trees](https://github.com/allencharp/LeetCode/blob/master/UniqueBinarySearchTrees.py) — how many structurally unique BSTs store *n* nodes?
2. [Different Ways to Add Parentheses](https://github.com/allencharp/LeetCode/blob/master/DifferentWaystoAddParentheses.py)
