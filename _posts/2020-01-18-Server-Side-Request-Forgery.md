---
layout: post
title:  "Server Side Request Forgery"
date:   2020-01-18 16:24:00 +0800
---

# What is SSRF
* Server-side request forgery (also known as SSRF) is a web security vulnerability that allows an attacker to induce the servier-side application to make HTTP requests to an arbitrary domain of the attacker's choosing
* In typical SSRF examples, the attacker might cause the server to make a connection back to itself, or to other web-based services within the organization's infrastrcture, or to external third-party systems
* A successful SSRF attack can often result in unauthorized actions or access to data within the organization, either in the vulnerable application itself or on other back-end systems that the applciation can communicate with. In some situations, the SSRF vulnerabilitiy might allow an attacker to perfor arbitrary command execution. 

# Sample Malicious Code
{% highlight php %}
<?php

/**
* Check if the 'url' GET variable is set
* Example - http://localhost/?url=http://testphp.vulnweb.com/images/logo.gif
*/
if (isset($_GET['url'])){
$url = $_GET['url'];

/**
* Send a request vulnerable to SSRF since
* no validation is being done on $url
* before sending the request
*/
$image = fopen($url, 'rb');

/**
* Send the correct response headers
*/
header("Content-Type: image/png");

/**
* Dump the contents of the image
*/
fpassthru($image);}
{% endhighlight %}

# Mitigating Server Side Request Forgery
* Disable URL schema
If your application only uses HTTP or HTTPS, then disable unnecessary URL schemas, it will be unable to use the web application to make requests using potentially dangerous schemas such as <b>file://</b>, <b>dict://</b>, <b>ftp://</b> and <b>gopher://</b>.

