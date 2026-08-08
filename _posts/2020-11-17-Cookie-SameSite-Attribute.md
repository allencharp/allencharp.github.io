---
layout: post
title:  "Cookie SameSite Attribute"
date:   2020-11-17 19:56:00 +0800
author: allencharp
tags: [web-security, cookies, samesite]
---

# What are first-party and third-party cookies?

* Cookies that match the domain of the current site (what's displayed in the browser's address bar) are referred to as **first-party cookies**.
* Cookies from domains other than the current site are referred to as **third-party cookies**.

# SameSite attribute

* **Strict**: If you set SameSite to **Strict**, your cookie will only be sent in a first-party context. In user terms, the cookie will only be sent if the site for the cookie matches the site currently shown in the browser's URL bar.
* **Lax**: When you set a cookie's SameSite attribute to **Lax**, the cookie will be sent along with GET requests initiated by third-party websites. The important point is that the GET request must cause a top-level navigation (e.g. clicking a link) for the cookie to be sent.
* **None**: Cookies with `SameSite=None` are sent in all contexts, including cross-site requests, but the cookie must also be marked **Secure** so it is only sent over HTTPS.
