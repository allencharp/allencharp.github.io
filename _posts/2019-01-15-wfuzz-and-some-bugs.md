---
layout: post
title:  "wfuzz and some bugs"
date:   2019-01-15 22:31:34 +0800
---

# Don't set Content-Type Header in Wfuzz

If use postdata to send HTTP Post, don't change the http content-type.
The origin source code in src/wfuzz/externals/reqresp/Request.py
{% highlight python %}
elif name == "postdata":
    if self.ContentType == "application/x-www-form-urlencoded":
            return self.__variablesPOST.urlEncoded()
    elif self.ContentType == "multipart/form-data":
            return self.__variablesPOST.multipartEncoded()
    else:
            return self.__uknPostData
{% %}
If we change the ContentType to applicaton/json, then the post data will be corrupted to __uknPostData