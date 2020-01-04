---
layout: post
title:  "ASP.NET Security Feature"
date:   2019-12-22 20:04:34 +0800
---

# Code Best Practices
### Good approach when handle the upload file.
##### Handle DDoS
{% highlight csharp %}
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

##### Validate the upload extension name
{% highlight aspx-cs %}
<validationsettings allowedfileextensions=".jpg,.png"></validationsettings>
{% endhighlight %}