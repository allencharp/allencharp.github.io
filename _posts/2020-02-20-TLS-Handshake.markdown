---
layout: post
title:  "TLS Handshake"
date:   2020-02-20 21:16:00 +0800
author: allencharp
tags: [network-security, tls, crypto]
---

# Introduction

TLS (Transport Layer Security) secures nearly all modern web traffic. The handshake is the phase where the client and server authenticate each other and derive session keys; after that, data flows with symmetric encryption.

This post walks through the **TLS 1.2** handshake with an ephemeral Diffie-Hellman (DHE/ECDHE) cipher suite, then contrasts it with TLS 1.3.

# Steps of the TLS 1.2 Handshake (DHE cipher suite)

* **Client hello**: The client sends a client hello message with the protocol version, the client random, and a list of cipher suites.
* **Server hello**: The server replies with its SSL certificate, its selected cipher suite, and the server random. With an ephemeral Diffie-Hellman (DHE) cipher suite, the server also includes the following (step 3):
* **Server's digital signature**: The server uses its private key to *sign* (not encrypt) the client random, the server random, and its DH parameter. This signed data acts as the server's digital signature, proving that the server holds the private key matching the public key in the SSL certificate.
* **Digital signature confirmed**: The client verifies the server's digital signature with the public key, confirming that the server controls the private key and is who it says it is.
* **Client DH parameter**: The client sends its DH parameter to the server.
* **Client and server calculate the premaster secret**: Both sides compute the same shared premaster secret from their own DH private value and the peer's DH parameter. With ECDHE this is the ephemeral ECDH shared secret — ephemeral keys are discarded after the handshake.
* **Session keys created**: The client and server derive session keys from the premaster secret, the client random and the server random.
* **Client is ready**: The client sends a ChangeCipherSpec message followed by an encrypted Finished message.
* **Server is ready**: The server sends its ChangeCipherSpec message and encrypted Finished message.
* Secure symmetric encryption is achieved.

# Forward secrecy

With **ephemeral** Diffie-Hellman (DHE/ECDHE), each handshake uses fresh, short-lived key material. Even if a server's long-term private key leaks later, previously recorded sessions cannot be decrypted — this property is called **forward secrecy**. Non-ephemeral key exchange (plain RSA key exchange) does not provide it, which is why DHE/ECDHE suites are the modern standard.

# Certificate chain and CA validation

The server's certificate is signed by a Certificate Authority (CA). The client validates the chain:

1. The certificate is signed by a trusted root/intermediate CA (or chains up to one).
2. The certificate covers the hostname being contacted (validity period and SAN entries are checked).
3. The certificate is not revoked or expired.

A browser's trust store decides which roots are trusted — which is why MITM proxies (like the ZAP CA in my ZAP post) must be explicitly imported into the trust store.

# TLS 1.3 in one glance

TLS 1.3 (RFC 8446) simplifies the handshake dramatically:

* The key exchange is completed in **one round trip** (0-RTT for resumed sessions) by sending the client's key share in the ClientHello.
* The RSA key exchange and all non-forward-secret suites were removed; forward secrecy is mandatory.
* Handshake messages after the ClientHello are encrypted, hiding certificates from passive observers.

# Summary

The TLS handshake authenticates the server (certificate + signature), establishes a shared secret (DHE/ECDHE), and derives session keys. Ephemeral key exchange gives forward secrecy, and TLS 1.3 makes the whole process faster and more secure by default.
