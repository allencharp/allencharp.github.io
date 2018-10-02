---
layout: post
title:  "Catalan Number"
date:   2018-07-08 20:34:34 +0800
---

<h1>Description</h1>

About Catalan Number [here]({https://en.wikipedia.org/wiki/Catalan_number}) is the description

The array of the number is: 1, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, 16796, 58786....

<image src="https://wikimedia.org/api/rest_v1/media/math/render/svg/34d4f28865115a05a806649a40f84e1bbc736320" />
<hr>
{%highlight python%}
def catalan_number(n):
    nm = dm = 1
    for k in range(2, n+1):
      nm, dm = ( nm*(n+k), dm*k )
    return nm/dm
{% endhighlight %}

<h1>Leetcode Test</h1>
1. [Binary Search Tree]({ https://github.com/allencharp/LeetCode/blob/master/UniqueBinarySearchTrees.py })<br>
About how many structurally unique BST(binary search trees) that store value O(n)?

2. [Different Ways to Add Parentheses]({ https://github.com/allencharp/LeetCode/blob/master/DifferentWaystoAddParentheses.py })

