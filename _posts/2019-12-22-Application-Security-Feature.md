---
layout: post
title:  "Application Security Feature"
date:   2019-12-22 20:04:34 +0800
---

# Code Best Practices
[reference](https://www.youtube.com/watch?v=V_LFLjnFnis&t=657s) <br>
### Good approach when handle the upload file.
* Handle DDoS
{% highlight C# %}
using(var stream = file.FileContent)
{
    DoProcessing(stream);
}
{% endhighlight %}
Bad approach, when using <strong>FileBytes</strong>, it will read all the file content into member, which is possible for Denial of Service.<br>
Sample:
{% highlight csharp %}
DoProcessing(file.FileBytes)
{% endhighlight %}

* Validate the upload extension name, to prevent the malicious file attack
{% highlight aspx-cs %}
<validationsettings allowedfileextensions=".jpg,.png"></validationsettings>
{% endhighlight %}

### Secure the way when displaying binary images
* Handle the <strong>ContentType</strong> properly
{% highlight C# %}
Response.ContentType = "image/jpeg"; #specify content-type to prevent the vulnerability
Response.Headers.Add("X-Content-Type-Options", 'nosniff");
{% endhighlight %}
Typciall the jpg xss attack:
![jpg xss attack]({{site.baseurl}}/assets/images/jpg-xss.jpg)
