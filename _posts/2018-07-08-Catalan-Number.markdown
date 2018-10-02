---
layout: post
title:  "Catalan Number"
date:   2018-07-08 20:34:34 +0800
---

Catalan number:

how many conditions are there to combine a binary search tree with O(n)

<image src="https://wikimedia.org/api/rest_v1/media/math/render/svg/34d4f28865115a05a806649a40f84e1bbc736320" />
<hr>
{% highlight python %}
def catalan_number(n):
    nm = dm = 1
    for k in range(2, n+1):
      nm, dm = ( nm*(n+k), dm*k )
    return nm/dm
{% endhighlight %}

the first catalan numbers from n=0,1,2,3,... are
1,1,2,