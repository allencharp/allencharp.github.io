---
layout: post
title:  "Container Image Signing and Verification"
date:   2026-08-11 09:00:00 +0800
author: allencharp
tags: [supply-chain-security, containers, cosign, sigstore, security-design]
---

# Introduction

In the previous post we locked down third-party packages behind Nexus. Container images are the same attack surface, in a different packaging: an attacker who compromises a registry account, overwrites a tag, or tricks a build pipeline can ship arbitrary code to every environment that pulls that image. A tag alone proves nothing — tags are mutable pointers.

Signing and verification fix this with cryptography: an image is signed by its builder, and every consumer (CI, or the production runtime) verifies the signature before trusting it. This post covers what image signing actually protects, the two mainstream toolchains — **Cosign** (sigstore) and **Notation** (Notary v2) — and how to enforce verification at runtime.

![Container image signing and verification flow](/assets/images/container-image-signing-flow.svg)

# What Signing Actually Covers

## Tags Are Names, Digests Are Content

A tag like `app:latest` can be moved to any image. The **digest** (`sha256:...`) is content-addressed — it changes if and only if the content changes. Signing therefore binds to the digest, never to the tag:

{% raw %}
```bash
# resolve a tag to an immutable digest first
docker pull registry.example.com/app:latest
docker image inspect --format '{{index .RepoDigests 0}}' registry.example.com/app:latest
# → registry.example.com/app@sha256:3f9a...
```
{% endraw %}

## The Sign / Verify Model

- **Sign**: the builder signs the image manifest digest with a private key. The signature (a small artifact) is stored back in the registry, associated with the digest.
- **Verify**: the consumer checks the signature against a trusted public key and confirms the digest matches. If the image content or the signature is tampered with, verification fails.

Two things must be protected outside the image itself: the **private key** (only the builder has it) and the **trust anchor** (which public keys your environment accepts).

# Tooling Overview

- **Cosign** (sigstore project, part of CNCF): the most common choice. Supports classic key pairs and **keyless** signing via OIDC identity, Fulcio certificates and the Rekor transparency log.
- **Notation** (Notary v2, CNCF): OCI-native signatures stored as manifest artifacts; designed for enterprise key management (KMS / HSM), with a policy-based verification model.
- **Docker Content Trust** (Notary v1): the old generation, tied to Docker Hub's legacy UCP; effectively superseded. Not covered here.

# Signing with Cosign

## Key-Based Workflow

```bash
# 1. generate a key pair (keep cosign.key secret, publish cosign.pub)
cosign generate-key-pair

# 2. sign the image digest
cosign sign --key cosign.key \
  registry.example.com/app@sha256:3f9a...

# 3. verify with the public key
cosign verify --key cosign.pub \
  registry.example.com/app@sha256:3f9a...
```

The `cosign.pub` public key becomes your trust anchor. Distribute it out-of-band (Git repo, configmap, secrets) and rotate it like any credential.

## Keyless Workflow

Keyless signing removes the long-lived private key from CI — the risk of key exfiltration:

```bash
# CI: sign with the runner's OIDC identity (GitHub Actions / GitLab / ...)
cosign sign registry.example.com/app@sha256:3f9a...

# verify against the expected identity, not a static key
cosign verify \
  --certificate-identity "https://github.com/your-org/.github/workflows/build.yml@refs/heads/main" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  registry.example.com/app@sha256:3f9a...
```

Keyless requires the sigstore infrastructure (Fulcio / Rekor). In air-gapped or enterprise environments you either self-host sigstore or fall back to Notation with your own KMS.

# Signing with Notation

Notation is the OCI-native option: the signature is stored as an OCI artifact referenced from the image manifest, so it works with any OCI 1.1 registry.

```bash
# 1. add your trust anchor (a CA or leaf certificate)
notation cert add --type ca --name my-ca trust.pem

# 2. sign with a configured key (can be backed by AWS KMS / Azure Key Vault)
notation sign --key my-signing-key \
  --signature-manifest imageManifest \
  registry.example.com/app@sha256:3f9a...

# 3. verify against the trust policy
notation verify registry.example.com/app@sha256:3f9a...
```

Because keys live in your KMS, the private key never leaves infrastructure — a strong fit for regulated environments.

# Enforcing Verification at Runtime

Signing only pays off if something refuses unsigned images. Two layers:

**1. Verify before promotion (CI)** — mirror the Nexus model: the test environment signs, CI verifies, and only verified digests are promoted to the production registry.

**2. Admission control (runtime)** — the cluster refuses unsigned images:

```yaml
# Kyverno ClusterPolicy: only accept images with a valid cosign signature
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-image-signature
spec:
  validationFailureAction: Enforce
  rules:
    - name: cosign-verify
      match:
        any:
          - resources:
              kinds:
                - Pod
      verifyImages:
        - image: "registry.example.com/*"
          key: |-
            -----BEGIN PUBLIC KEY-----
            ...your cosign.pub...
            -----END PUBLIC KEY-----
```

The equivalent with OPA Gatekeeper + `sigstore` constraint library achieves the same goal for clusters that already run Gatekeeper.

![Kubernetes admission control image signature verification flow](/assets/images/k8s-admission-verify-flow.svg)

# Pitfalls

- **Docker pull does not verify**: `docker pull` fetches the image blindly — signature verification is a separate, manual step with `cosign verify` or `notation verify`. The image is already on disk before you know whether it is trusted. **Podman** is the exception: with a configured trust policy (`/etc/containers/policy.json`) it rejects unsigned images at pull time, closing the gap between "downloaded" and "verified."
- **Verifying tags instead of digests**: re-resolve the digest at deploy time, or an attacker who can move the tag can bypass a check done earlier.
- **Trust anchor hygiene**: a leaked or overly-broad public key is a backdoor. Pin keys to repositories/namespaces and rotate on a schedule.
- **Rollback protection**: a signature proves origin, not freshness. Combine with age/expiry policy so an old signed-but-vulnerable image cannot be quietly redeployed.
- **Registry support**: OCI referrers (or cosign's fallback tag storage) must be enabled on the registry, or signature artifacts will not be discoverable.
- **Keyless dependencies**: if Fulcio/Rekor are unreachable, verification fails — decide beforehand whether you self-host sigstore or use Notation with KMS.

# Conclusion

Package repositories (previous post) and container images are two halves of the same supply-chain boundary. Signing gives you *who built it and that it is unchanged*; verification at admission time gives you a place to say no. Together with the Nexus-style staged promotion, an attacker has to win every layer — while you only have to hold one.

# References

- sigstore/cosign — <https://github.com/sigstore/cosign>
- Notation (Notary v2) — <https://notaryproject.dev/>
- Kyverno verifyImages — <https://kyverno.io/docs/writing-policies/verify-images/>
- OPA Gatekeeper + sigstore — <https://github.com/sigstore/gatekeeper-plugin>
- OCI Image Spec / referrers — <https://github.com/opencontainers/image-spec>
