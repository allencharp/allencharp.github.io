---
layout: post
title:  "Server Side Request Forgery"
date:   2020-01-18 16:24:00 +0800
author: allencharp
tags: [web-security, ssrf, pentest]
---

# What is SSRF

* Server-side request forgery (also known as SSRF) is a web security vulnerability that allows an attacker to induce the server-side application to make HTTP requests to an arbitrary domain of the attacker's choosing.
* In typical SSRF examples, the attacker might cause the server to make a connection back to itself, or to other web-based services within the organization's infrastructure, or to external third-party systems.
* A successful SSRF attack can often result in unauthorized actions or access to data within the organization, either in the vulnerable application itself or on other back-end systems that the application can communicate with. In some situations, the SSRF vulnerability might allow an attacker to perform arbitrary command execution.

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

* **Input validation and whitelist**  
  Use **regex** to ensure that the received data is valid from a security point of view, if the input data has a simple format. After ensuring the validity of the incoming IP address, a second layer of validation is applied: a whitelist is created after determining all the IP addresses (IPv4 and IPv6, to avoid bypasses) of the identified and trusted applications. The valid IP is cross-checked against that list (strict string comparison, case sensitive) to ensure communication with the internal application.

* **Disable unnecessary URL schemes**  
  If your application only uses HTTP or HTTPS, then disable unnecessary URL schemes, so the web application cannot be used to make requests using potentially dangerous schemes such as **file://**, **dict://**, **ftp://** and **gopher://**.

* **Authentication on internal services**  
  By default, services such as Memcached, Redis, Elasticsearch, and MongoDB do not require authentication. An attacker can use SSRF vulnerabilities to access these services without any authentication. Therefore, to ensure web application security, it is best to enable authentication wherever possible, even for services on the local network.

[reference](https://www.acunetix.com/blog/articles/server-side-request-forgery-vulnerability/)
