---
layout: post
title:  "Securing Nexus Against Third-Party Supply Chain Poisoning"
date:   2026-08-09 09:00:00 +0800
author: allencharp
tags: [supply-chain-security, nexus, devsecops, repository, security-design]
---

# Introduction

The real attack surface is supply-chain poisoning. A single `pip install` typed by a developer can unknowingly pull a poisoned package into the whole CI/CD pipeline; once installed, the malicious package can be triggered and cause real damage across our environment.

This post describes how we use Nexus Repository Manager as the single choke point for third-party packages, and the configuration baseline we apply to keep poisoning attacks out.

# Nexus Configuration Baseline

We run two Nexus environments that host the plugin/package artifacts our builds depend on: one for the test environment and one for the online (production) environment. The baseline below turns each of them from a simple mirror into a barrier against third-party poisoning.

## Route All Resolution Through the Test Nexus Proxy

Create one proxy repository per format (`pypi-proxy`, `npm-proxy`, `maven-central`), then point every client at Nexus and nowhere else:

All test servers resolve exclusively through the test Nexus — replace `nexus` in the examples below with your test Nexus hostname.

```ini
# ~/.pip/pip.conf
[global]
index-url = http://nexus:8081/repository/pypi-proxy/simple
trusted-host = nexus
```

```ini
# ~/.npmrc
registry = http://nexus:8081/repository/npm-proxy/
```

```xml
<!-- maven settings.xml -->
<mirrors>
  <mirror>
    <id>nexus</id>
    <mirrorOf>*</mirrorOf>
    <url>http://nexus:8081/repository/maven-central/</url>
  </mirror>
</mirrors>
```

That means, in the test environment, every `pip install` or `npm update` resolves through the test Nexus. If a package does not exist there, the proxy repository pulls it from the third-party repository, caches it in the test Nexus, and serves it to the requesting server.

## Poisoning Scan on the Test Nexus

Once a component is pulled from a third-party repository, a poisoning scan runs on the test Nexus to flag any poisoned packages or plugins before they can spread further. It checks new components against known-malicious sources (Sonatype IQ / Repository Firewall when licensed, or OSS alternatives such as OWASP Dependency-Check, `pip-audit` and Trivy); anything flagged is quarantined, reported, and kept out of the online Nexus.

## Block Third-Party URLs at the Test VPC Firewall

Block the third-party repository URLs (e.g. `pypi.org`, `registry.npmjs.org`) at the test environment's VPC firewall. Only the test Nexus host is allowed outbound to those domains, so test servers have exactly one path for packages: the test Nexus. If a test server tries to reach a public registry directly, the firewall drops the request.

## Online Nexus

The online Nexus does not talk to third-party repositories directly. A scheduled job copies approved packages from the test Nexus to the online Nexus, but only when two conditions are met:

- **The package passed the poisoning scan** on the test Nexus.
- **The package is at least 30 days old**, counted from its publish date in the upstream registry. This grace period is deliberate: most poisoned packages are discovered and removed from public registries within days or weeks, so by the time a package reaches production the known-bad versions have usually been purged upstream.

A crontab entry for the sync job could look like this:

```bash
# nightly sync: only packages that passed the scan and are older than 30 days
0 2 * * * /usr/local/bin/nexus-sync --from test-nexus --to online-nexus --min-age 30d
```

The sync script lists components from the test Nexus REST API, filters them by scan status and age, and promotes the matching ones to the online Nexus. The 30-day window means new releases never reach production immediately — a real constraint to communicate to developers, but a cheap price for keeping a poisoned package out of the online environment.

## Block Third-Party URLs for the Online Nexus

The online Nexus runs no proxy repositories — it never fetches from the internet on its own. Its only write path is the nightly sync: the sync job uploads approved packages from test Nexus into the online Nexus repositories, and everything else is denied.

# Architecture at a Glance

```
                     test VPC: firewall blocks direct 3rd-party access

   ┌───────────────┐      ┌──────────────────┐      ┌──────────────────┐
   │ dev / test    │─────▶│   test Nexus     │─────▶│ third-party repos│
   │ servers       │      │  (proxy repos)   │      │  pypi / npm / ...│
   └───────────────┘      └────────┬─────────┘      └──────────────────┘
                                   │
                                   ▼
                      ┌────────────────────────┐
                      │ ① poisoning scan       │── hit ─▶ quarantine + alert
                      └────────────────────────┘         (never reaches online)
                                   │ passed
                                   ▼
                      ┌────────────────────────┐
                      │ ② 30-day observation   │  upstream removes bad versions
                      └────────────────────────┘
                                   │ nightly cron sync
                                   ▼
                     online VPC: no public internet

   ┌───────────────┐      ┌──────────────────┐
   │  production   │◀─────│  online Nexus    │   no proxy repos —
   │  servers      │      │  (sync only)     │   never fetches on its own
   └───────────────┘      └──────────────────┘
```

Every package travels the same path: pulled into the test Nexus from a third-party repository, scanned, held for 30 days, and only then synced to the online Nexus — which never touches the public internet.
