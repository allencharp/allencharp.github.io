---
layout: post
title:  "wfuzz and some bugs"
date:   2019-01-15 22:31:34 +0800
---

# Wuff is a cool brute force tool, currently I integrate python Wuff into ZAPv2 library could do a lot of amazing job. <br>
# However, Wfuzz also contains some bugs that we should avoid 

* Couldn't set Content-Type Header in Wfuzz when we post data

If use postdata to send HTTP Post, don't change the http content-type. The origin source code in <br>
[src/wfuzz/externals/reqresp/Request.py](https://github.com/xmendez/wfuzz/blob/master/src/wfuzz/externals/reqresp/Request.py)
{% highlight python %}
elif name == "postdata":
    if self.ContentType == "application/x-www-form-urlencoded":
            return self.__variablesPOST.urlEncoded()
    elif self.ContentType == "multipart/form-data":
            return self.__variablesPOST.multipartEncoded()
    else:
            return self.__uknPostData
{% endhighlight %}
If we change the ContentType to applicaton/json, then the post data will be corrupted to __uknPostData