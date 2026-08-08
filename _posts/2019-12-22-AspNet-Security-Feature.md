---
layout: post
title:  "ASP.NET Security Feature"
date:   2019-12-22 20:04:34 +0800
author: allencharp
tags: [web-security, aspnet, hardening]
---

# Code Best Practices

[reference](https://github.com/DevExpress/aspnet-security-bestpractices/tree/master/SecurityBestPractices.WebForms)

### Good approach when handling file upload

* Handle memory usage (prevent DoS):

{% highlight C# %}
using(var stream = file.FileContent)
{
    DoProcessing(stream);
}
{% endhighlight %}

The bad approach below uses **FileBytes**, which reads the whole file content into memory and can lead to a Denial of Service:

{% highlight C# %}
DoProcessing(file.FileBytes) // Bad approach sample
{% endhighlight %}

* Validate the upload extension to prevent malicious file attacks:

{% highlight aspx-cs %}
<validationsettings allowedfileextensions=".jpg,.png"></validationsettings>
{% endhighlight %}

### Secure the way binary images are displayed

* Handle the **ContentType** properly:

{% highlight C# %}
Response.ContentType = "image/jpeg"; // specify content-type to prevent the vulnerability
Response.Headers.Add("X-Content-Type-Options", "nosniff");
{% endhighlight %}

Typically, the JPG XSS attack:

![jpg xss attack]({{site.baseurl}}/assets/images/jpg-xss.jpg)

### Prevent Open Redirect

* IsLocalUrl validation:

{% highlight C# %}
private IActionResult RedirectToLocal(string returnUrl)
{
    if (Url.IsLocalUrl(returnUrl))
    {
        return Redirect(returnUrl);
    }
    else
    {
        return RedirectToAction(nameof(HomeController.Index), "Home");
    }
}
{% endhighlight %}

* LocalRedirect:

{% highlight C# %}
public IActionResult SomeAction(string redirectUrl)
{
    return LocalRedirect(redirectUrl);
}
{% endhighlight %}

### Enable CSRF token

ASP.NET sample:

{% highlight C# %}
[ValidateAntiForgeryToken]
{% endhighlight %}

### Path Manipulation and Path.Combine()

{% highlight C# %}
public static byte[] GetFile(String filename) {
  String imageDir = "C:\\Image\\";
  filepath = Path.Combine(imageDir, filename);

  return File.ReadAllBytes(filepath);
}
{% endhighlight %}

The security issue in the code above is using **Path.Combine()** to generate the path string. If the second parameter *filename* uses an absolute path, the first parameter *imageDir* is ignored.  
From [MS Doc](https://docs.microsoft.com/en-us/dotnet/api/system.io.path.combine?view=netframework-4.8):

> If path2 does not include a root (for example, if path2 does not start with a separator character or a drive specification), the result is a concatenation of the two paths, with an intervening separator character. If path2 includes a root, path2 is returned.

> The parameters are not parsed if they have white space. Therefore, **if path2 includes white space (for example, " \file.txt "), the Combine method appends path2 to path1 instead of returning only path2.**

Prevention: use **Path.GetFileName()** to get the "base name" of the parameter:

{% highlight C# %}
public static byte[] GetFile(String filename) {

  if (string.IsNullOrEmpty(filename) || Path.GetFileName(filename) != filename)
  {
    throw new ArgumentNullException("error");
  }

  String filepath = Path.Combine("\\FILESHARE\images", filename);
  return File.ReadAllBytes(filepath);
}
{% endhighlight %}
