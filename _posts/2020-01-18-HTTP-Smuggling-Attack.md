---
layout: post
title:  "HTTP Smuggling Attack"
date:   2020-01-18 16:24:00 +0800
---

# What is SSRF
* Server-side request forgery (also known as SSRF) is a web security vulnerability that allows an attacker to induce the servier-side application to make HTTP requests to an arbitrary domain of the attacker's choosing
* In typical SSRF examples, the attacker might cause the server to make a connection back to itself, or to other web-based services within the organization's infrastrcture, or to external third-party systems
* A successful SSRF attack can often result in unauthorized actions or access to data within the organization, either in the vulnerable application itself or on other back-end systems that the applciation can communicate with. In some situations, the SSRF vulnerabilitiy might allow an attacker to perfor arbitrary command execution. 