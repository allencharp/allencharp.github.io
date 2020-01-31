---
layout: post
title:  "ASP.NET Security Feature"
date:   2019-12-22 20:04:34 +0800
---

# Code Best Practices
[reference](https://github.com/DevExpress/aspnet-security-bestpractices/tree/master/SecurityBestPractices.WebForms) <br>
<br>
### Good approach when handle the upload file.
* Handle DDoS
{% highlight C# %}
using(var stream = file.FileContent)
{
    DoProcessing(stream);
}
{% endhighlight %}
Bad approach, when using <strong>FileBytes</strong>, it will read all the file content into member, which is possible for Denial of Service.<br>
{% highlight C# %}
DoProcessing(file.FileBytes) // Bad approach sample
{% endhighlight %}

* Validate the upload extension name, to prevent the malicious file attack
{% highlight aspx-cs %}
<validationsettings allowedfileextensions=".jpg,.png"></validationsettings>
{% endhighlight %}
<br>
### Secure the way when displaying binary images
* Handle the <strong>ContentType</strong> properly
{% highlight C# %}
Response.ContentType = "image/jpeg"; #specify content-type to prevent the vulnerability
Response.Headers.Add("X-Content-Type-Options", "nosniff");
{% endhighlight %}
Typciall the jpg xss attack:
![jpg xss attack]({{site.baseurl}}/assets/images/jpg-xss.jpg)
<br>
### Prevent Open Redirect
* IsLocalUrl Validation
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

* LocalRedirect
{% highlight C# %}
public IActionResult SomeAction(string redirectUrl)
{
    return LocalRedirect(redirectUrl);
}
{% endhighlight %}
<br>

### Enable CSRF token
ASP.net sample:
{% highlight C# %}ßßßßßß
[ValidateAntiForgeryToken]
{% endhighlight %}
<br>


### Path Manipulation and Path.Combine()
{% highlight C# %}
public static bytes[] getFile(String filename) {
  // Disallow filenames with the dot-dot path traversal sequence.
  if (filename.Contains("..") {
    throw new ArgumentException("error");
  }

  String imageDir = "C:\\Image\\";
  filepath = Path.Combine(imageDir, filename);
  var ext = Path.GetExtension(filePath);
  return File.ReadAllBytes(filepath);
}
{% endhighlight %}
The security issue of the above code is using <b>Path.Combine()</b> to generate the path string. However, if the second parameter *filename* is using absolute path, then the first parameter *imageDir* will be ignored<br>
From [MS Doc](https://docs.microsoft.com/en-us/dotnet/api/system.io.path.combine?view=netframework-4.8) it says:
> If path2 does not include a root (for example, if path2 does not start with a separator character or a drive specification), the result is a concatenation of the two paths, with an intervening separator character. If path2 includes a root, path2 is returned.

> The parameters are not parsed if they have white space. Therefore, <b>if path2 includes white space (for example, " \file.txt "), the Combine method appends path2 to path1 instead of returning only path2.</b>

