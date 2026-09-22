---
layout: post
title:  "API Security: The Six Gates"
date:   2026-09-04 15:00:00 +0800
author: allencharp
tags: [api-security, security-design]
---


# Where to Start

There is no single answer to "how do I secure this API" — the right set of controls depends entirely on what the API does and who can reach it. Three questions settle it:

- **Who calls it** — first-party browser, mobile app, server-to-server, or a third party?
- **What does it touch** — public catalogue data, or payments and PII?
- **Where does it sit** — public internet, or inside a service mesh?

Those answers decide how much of what follows is worth building, and in what depth. The rest of this note walks a single request from the socket to the database, describing one control per hop — an ordering that keeps a security review from collapsing into an unordered list of vulnerabilities.

# Six Gates

Every request crosses the same six boundaries. Each one answers a single question, and none of them substitutes for another.

![Six gates an API request passes through](/assets/images/api-security-six-layers.svg)

**1. Transport.** TLS on every hop, including the internal ones — "it is behind the VPN" stops being true the day someone adds a second ingress. HSTS, no silent downgrade, and mTLS between services inside the mesh.

**2. Authentication — who are you.** For users, OAuth 2.1 / OIDC with short-lived access tokens and rotating refresh tokens. For services, mTLS or signed assertions; an API key alone carries no identity and no proof of possession. Secrets live in a KMS or secrets manager, never in the repo, never in an image layer.

**3. Authorization — may you touch *this* object.** One layer below, and the one that actually gets breached. See the next section.

**4. Input and output.** Schema validation on an allowlist, not a denylist. Parameterised queries. Mass-assignment protection — a client that can PUT `{"role":"admin"}` will. Response redaction, and error bodies that carry a code rather than a stack trace.

**5. Abuse controls.** Rate limits keyed on user, key and endpoint together — IP alone is useless against a distributed client and punishes everyone behind a NAT. Brute-force lockout with backoff on login and OTP endpoints. `Idempotency-Key` on anything that moves money. Signature plus nonce plus timestamp window if replay matters.

**6. Audit.** An accurate API inventory, because you cannot secure an endpoint you forgot existed. Shadow and zombie APIs retired on a schedule. Authn, rate limiting and access logs centralised at the gateway, with alerting on anomalous call patterns.

# Layer Three Is Where Breaches Happen

Most real API breaches aren't crypto failures — they're requests that authenticated fine, then touched data they shouldn't. The fix is one line of server-side ownership code that keeps getting skipped, because middleware auth feels like it covered this.

![Broken object level authorization versus a proper ownership check](/assets/images/api-bola-ownership-check.svg)

The 2023 OWASP API Security Top 10, reduced to the actual fix:

| Risk | The actual fix |
| --- | --- |
| BOLA — object level | Ownership check on every request for every object id |
| Broken authentication | Short tokens, rotation, one-time codes |
| BOPLA — property level | Allowlist writable fields; never return internal fields |
| Unrestricted resource consumption | Depth/complexity/pagination caps, timeouts |
| BFLA — function level | Enforce roles server-side on every admin and write route |
| Sensitive business flows | Rate limit the *flow*, not the endpoint |
| SSRF | Allowlist outbound destinations; block link-local metadata |
| Misconfiguration | No directory listing, verbose errors, default creds |
| Improper inventory | Retire old versions; document every route |
| Unsafe consumption | Validate third-party API responses too |

**Scanners miss business logic.** Never trust amounts, stock, or coupon state from the client — the server decides what they're worth. Lock concurrency; validate state machines on every transition.
