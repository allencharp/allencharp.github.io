---
layout: post
title:  "wfuzz and some bugs"
date:   2019-01-15 22:31:34 +0800
author: allencharp
tags: [pentest, wfuzz, fuzzing]
---

# Wfuzz

[Wfuzz](https://github.com/xmendez/wfuzz) is a cool brute-force / fuzzing tool. I also integrated Python Wfuzz into the ZAPv2 library, which can do a lot of amazing jobs.

However, Wfuzz also contains some bugs that we should be aware of.

## Can't set Content-Type header when posting data

When using `postdata` to send an HTTP POST, don't change the content-type. Looking at the source in [src/wfuzz/externals/reqresp/Request.py](https://github.com/xmendez/wfuzz/blob/master/src/wfuzz/externals/reqresp/Request.py):

{% highlight python %}
elif name == "postdata":
    if self.ContentType == "application/x-www-form-urlencoded":
            return self.__variablesPOST.urlEncoded()
    elif self.ContentType == "multipart/form-data":
            return self.__variablesPOST.multipartEncoded()
    else:
            return self.__uknPostData
{% endhighlight %}

If we change the ContentType to `application/json`, the post data will be corrupted to `__uknPostData`.
