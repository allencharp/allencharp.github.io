---
layout: post
title:  "CORS Header Misconfiguration"
date:   2019-09-22 22:31:34 +0800
author: allencharp
tags: [web-security, cors, misconfiguration]
---

# What is CORS

A request for a resource (like an image or a font) from outside the origin is known as a cross-origin request.  
CORS (Cross-Origin Resource Sharing) manages cross-origin requests.

The CORS standard is needed because it allows servers to specify not just *who* can access its assets, but also *how* the assets can be accessed.  
For example, server A probably does not want servers B, C or D to edit or delete its assets.

# CORS Headers

The most important header is **Access-Control-Allow-Origin**, which is decided by the server and controls which hosts can access the resource:

* Access-Control-Allow-Origin
* Access-Control-Allow-Credentials
* Access-Control-Allow-Headers
* Access-Control-Allow-Methods
* Access-Control-Expose-Headers
* Access-Control-Max-Age
* Access-Control-Request-Headers
* Access-Control-Request-Method
* Origin

# CORS Misconfiguration

* Setting `*` in the *Access-Control-Allow-Origin* header when the response contains sensitive information
* Poor validation of the *Origin* header
* Directly reflecting the *Origin* header into *Access-Control-Allow-Origin*

# CORS Scan Tool

[CORScanner](https://github.com/chenjj/CORScanner)
