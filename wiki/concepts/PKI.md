---
title: "PKI"
type: concept
created: 2026-04-14
updated: 2026-04-14
sources:
  - "[[Source---Understanding-CSR-and-CER]]"
tags:
  - cryptography
  - security
  - tls
---

# PKI

**Public Key Infrastructure** — the system of roles, policies, hardware, software, and procedures needed to create, manage, distribute, use, store, and revoke [[X509-Certificate|digital certificates]].

## Core Idea

PKI solves the problem of trust at scale: how can a client trust a server it has never met before? Answer: a **chain of trust** rooted in Certificate Authorities whose public keys are pre-installed in operating systems and browsers.

## Components

| Component | Role |
|---|---|
| **Private Key** | Held only by the key owner; used to sign (never shared) |
| **Public Key** | Shared freely; embedded in certificates |
| **[[CSR]]** | Certificate Signing Request — how you ask a CA to sign your public key |
| **[[X509-Certificate]]** | Signed certificate — proof your public key is vouched for by a CA |
| **[[Certificate-Authority]]** | Trusted third party that validates identity and signs certificates |
| **CA Chain** | Intermediate + Root CA certs that form the trust path |

## Trust Chain

```
Root CA (in OS/browser trust store)
  └── Intermediate CA (signed by Root)
        └── Server Certificate (signed by Intermediate)
```

Clients walk up this chain until they reach a Root CA they already trust.

## Asymmetric Cryptography Basis

PKI relies on asymmetric key pairs:
- **Sign**: `signature = encrypt(hash(data), private_key)`
- **Verify**: `hash = decrypt(signature, public_key)` — compare to `hash(data)`

The CA's signature on a certificate proves the certificate hasn't been tampered with and was issued by a trusted party.

## See Also

- [[TLS]] — the protocol that uses PKI for secure connections
- [[Certificate-Authority]] — the signing party
- [[CSR]] — how a certificate request is formed
- [[X509-Certificate]] — the resulting signed certificate
- [[Self-Signed-Certificate]] — PKI without a third-party CA
