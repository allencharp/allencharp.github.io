---
layout: post
title:  "Cookie SameSite Attribute"
date:   2020-11-17 19:56:00 +0800
author: allencharp
tags: [web-security, cookies, samesite]
---

# What are first-party and third-party cookies?

* Cookies that match the domain of the current site (what's displayed in the browser's address bar) are referred to as **first-party cookies**.
* Cookies from domains other than the current site are referred to as **third-party cookies** (e.g. analytics cookies when you visit a site that embeds a tracker from another domain).

# SameSite attribute

The `SameSite` attribute (part of [RFC 6265bis](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis)) controls when cookies are sent on cross-site requests. It takes three values:

## Strict

If you set SameSite to **Strict**, your cookie will only be sent in a first-party context. In user terms, the cookie will only be sent if the site for the cookie matches the site currently shown in the browser's URL bar. Even top-level navigation from an external link will not include the cookie — which can hurt UX (a link from an email won't keep the user logged in).

## Lax

When you set a cookie's SameSite attribute to **Lax**, the cookie will be sent along with GET requests initiated by third-party websites. The important point is that the GET request must cause a top-level navigation (e.g. clicking a link) for the cookie to be sent. Subresource requests and fetch/XHR calls are still blocked.

Lax is a good default: it preserves the "logged in via link" experience while blocking most cross-site request forgery.

## None

Cookies with `SameSite=None` are sent in all contexts, including cross-site requests, but the cookie must also be marked **Secure** so it is only sent over HTTPS. Browsers reject `SameSite=None` without `Secure`.

# SameSite and CSRF

Cross-Site Request Forgery (CSRF) works because browsers automatically attach cookies to requests made from any site. Setting `SameSite=Lax` (or `Strict`) on session cookies makes the browser withhold those cookies on cross-site POST requests and XHR/fetch, which breaks the most common CSRF delivery paths.

Important nuance: `SameSite` is a solid mitigation but **not a complete CSRF defense** — a strict allowlist of origins plus CSRF tokens is still recommended for sensitive actions. Also note the Lax exception for top-level GET navigations can still be abused in some scenarios.

# Browser defaults

Modern browsers (Chrome 80+, Edge, Firefox 115+) have moved toward treating cookies **without a SameSite attribute as `Lax` by default**. This means the *absence* of the attribute is no longer "send everywhere" — a meaningful security improvement, but it also means developers should set the attribute explicitly rather than relying on defaults.

# Summary

`SameSite` is a cheap and effective CSRF mitigation: pick `Lax` as the default for session cookies, use `Strict` for highly sensitive cookies, and only use `None` (with `Secure`) when cross-site cookie delivery is genuinely required.
