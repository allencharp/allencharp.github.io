---
layout: post
title:  "Cookie SameSite Attribute"
date:   2020-11-17 19:56:00 +0800
---

# What are first-party and third-party cookies?
* Cookies that match the domain of the current site, i.e. what's displayed in the browser's address bar, are referred to as <b>first-party cookies</b>. 
* Cookies from domains other than the current site are referred to as <b>third-party cookies</b>.

# SameSite attribute
* Strict: If you set SameSite to <b>Strict</b>, your cookie will only be sent in a first-party context. In user terms, the cookie will only be sent if the site for the cookie matches the site currently shown in the browser's URL bar. 
* Lax: When you set a cookie' SameSite attribute to Lax, the cookie will be sent along with the GET request initiated by third party website.
<br>
The important point here is that, to send a cookie with a GET request, GET request being made must cause a top level navigation. Only in this way, the cookie set as LAX will be sent. Let me explain more.

