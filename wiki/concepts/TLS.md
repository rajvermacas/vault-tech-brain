---
title: "TLS"
type: concept
created: 2026-04-14
updated: 2026-04-14
sources:
  - "[[Source---Understanding-CSR-and-CER]]"
tags:
  - security
  - networking
  - cryptography
---

# TLS

**Transport Layer Security** — the cryptographic protocol that secures communications over a network (HTTPS, mTLS, etc.). Relies on [[PKI]] for identity verification.

## TLS Handshake (simplified)

```
Client                          Server
------                          ------
ClientHello ─────────────────►
                                ServerHello
              ◄─────────────── Certificate (.cer/.crt)
Verify cert:
  - decrypt signature with CA public key
  - compare hash to cert fields
  - ✅ if match → server is who it claims to be

Key exchange ◄──────────────►
Encrypted session established
```

## Certificate Verification (local, no CA involved)

The [[Certificate-Authority]] is only involved **once** — at certificate issuance. After that, all TLS verification is local:

1. Client reads the [[X509-Certificate]] sent by server.
2. Extracts the CA's signature from the cert.
3. Decrypts the signature using the CA's **public key** (pre-installed in OS/browser trust store).
4. Computes `SHA256(cert_fields)`.
5. Compares: if hashes match → cert is authentic and untampered.

This is **signature verification**, not decryption of the server's public key (which is in plaintext in the cert).

## mTLS (Mutual TLS)

Both parties present certificates. Used for service-to-service authentication (e.g., inside Kubernetes, [[HashiCorp-Vault]] clients).

## Use Cases

- HTTPS (browser → web server)
- mTLS (service → service)
- [[HashiCorp-Vault]] client connections
- Azure App Gateway / AKS ingress termination
- Code signing, S/MIME email encryption

## See Also

- [[PKI]] — the trust infrastructure TLS relies on
- [[X509-Certificate]] — the certificate exchanged during handshake
- [[Certificate-Authority]] — who signs the certificate
- [[CSR]] — how certificates are requested
- [[Self-Signed-Certificate]] — when no public CA is used
