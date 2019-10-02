---
layout: post
title:  "CORS Header Misconfiguration"
date:   2019-09-22 22:31:34 +0800
---

# What is CORS Header
A request for a resource (like an image or a font) outside of the origin is known as a cross-origin request. <br>
CORS (cross-origin resource sharing) manages cross-origin requests.
<br>
<br>
The CORS standard is needed because it allows servers to specify not just who can access its assets, but also how the assets can be accessed.<br>
For example, it is likely that server A does not want servers B, C, or D to edit or delete its assets.
<br>
<br>

# CORS Header
The most important header is *Access-Control-Allow-Origin*, which is determined by server, which host could access the resource. <br>
* Access-Control-Allow-Origin
* Access-Control-Allow-Credentials
* Access-Control-Allow-Headers
* Access-Control-Allow-Methods
* Access-Control-Expose-Headers
* Access-Control-Max-Age
* Access-Control-Request-Headers
* Access-Control-Request-Method
* Origin
<br>
<br>

# CORS Misconfiguration
* Set "\*" to *Access-Control-Allow-Origin* header, which contains the sensitive inforamtion
* Poor Validation on *Origin* header
* Directly reflect the *Origin* header to *Access-Control-Allow-Origin*

# CORS Scan Tool
[CORScanner](https://github.com/chenjj/CORScanner)
