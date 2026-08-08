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

# Why SSRF is dangerous

The key to SSRF is that the request is made by the **server**, not the client. This means:

* The server's internal network becomes reachable: `localhost`, `127.0.0.1`, internal IP ranges (e.g. `10.x.x.x`, `172.16-31.x.x`, `192.168.x.x`), cloud metadata endpoints, and other back-end services that are not exposed to the internet.
* Cloud metadata is a classic target: on AWS, `http://169.254.169.254/latest/meta-data/` exposes temporary credentials and instance data; a single SSRF can leak the cloud account's secrets.
* SSRF can also be chained to reach internal admin panels, Redis/Memcached, or Elasticsearch instances and turn into RCE (e.g. via Redis + cron).

# Sample Malicious Code

```php
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
```

Here `$url` is taken directly from user input and passed to `fopen()`. An attacker only needs to change the parameter, e.g. `?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/`, to read the cloud metadata.

# Common Bypass Techniques

Filters are often imperfect. Common bypasses include:

* **IP confusion**: `2130706433` (decimal), `0x7f000001` (hex), `0177.0.0.1` (octal) or `127.1` can represent `127.0.0.1`.
* **Redirects**: the URL points to an allowed host, but the server follows a redirect to an internal address.
* **DNS rebinding**: a domain resolves to a public IP on the first lookup and to an internal IP on the second.
* **Encoding tricks**: `http://user@host/`, unusual ports, or IPv6 literals like `[::1]`.

# Mitigating Server Side Request Forgery

* **Input validation and whitelist**  
  Use **regex** to ensure that the received data is valid from a security point of view, if the input data has a simple format. After ensuring the validity of the incoming IP address, a second layer of validation is applied: a whitelist is created after determining all the IP addresses (IPv4 and IPv6, to avoid bypasses) of the identified and trusted applications. The valid IP is cross-checked against that list (strict string comparison, case sensitive) to ensure communication with the internal application.

* **Disable unnecessary URL schemes**  
  If your application only uses HTTP and HTTPS, then disable unnecessary URL schemes, so the web application cannot be used to make requests using potentially dangerous schemes such as **file://**, **dict://**, **ftp://** and **gopher://**.

* **Authentication on internal services**  
  By default, services such as Memcached, Redis, Elasticsearch, and MongoDB do not require authentication. An attacker can use SSRF vulnerabilities to access these services without any authentication. Therefore, to ensure web application security, it is best to enable authentication wherever possible, even for services on the local network.

* **Network-level defense**  
  In addition to application-level validation, block internal ranges at the egress firewall/proxy level (deny requests to private IP ranges from the application tier), and disable DNS rebinding where possible.

# Summary

SSRF is a server-side trust issue: the server blindly follows attacker-controlled URLs. Defense in depth — strict allowlists, scheme restrictions, authentication on internal services, and network segmentation — is required because any single bypass defeats a purely input-based filter.

[reference](https://www.acunetix.com/blog/articles/server-side-request-forgery-vulnerability/)
