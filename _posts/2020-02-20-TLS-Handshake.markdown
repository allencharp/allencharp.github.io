---
layout: post
title:  "TLS Handshake"
date:   2020-02-20 21:16:00 +0800
author: allencharp
tags: [network-security, tls, crypto]
---

# Steps of the TLS 1.2 Handshake (DHE cipher suite)

* **Client hello**: The client sends a client hello message with the protocol version, the client random, and a list of cipher suites.
* **Server hello**: The server replies with its SSL certificate, its selected cipher suite, and the server random. With an ephemeral Diffie-Hellman (DHE) cipher suite, the server also includes the following (step 3):
* **Server's digital signature**: The server uses its private key to sign the client random, the server random, and its DH parameter. This signed data acts as the server's digital signature, proving that the server holds the private key matching the public key in the SSL certificate.
* **Digital signature confirmed**: The client verifies the server's digital signature with the public key, confirming that the server controls the private key and is who it says it is.
* **Client DH parameter**: The client sends its DH parameter to the server.
* **Client and server calculate the premaster secret**: Both sides compute the same shared premaster secret from their own DH private value and the peer's DH parameter.
* **Session keys created**: The client and server derive session keys from the premaster secret, the client random and the server random.
* **Client is ready**: Same as an RSA handshake — the client sends a ChangeCipherSpec message followed by an encrypted Finished message.
* **Server is ready**: The server sends its ChangeCipherSpec message and encrypted Finished message.
* Secure symmetric encryption is achieved.
