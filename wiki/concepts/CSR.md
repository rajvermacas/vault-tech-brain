---
title: "CSR"
type: concept
created: 2026-04-14
updated: 2026-04-14
sources:
  - "[[Source---Understanding-CSR-and-CER]]"
tags:
  - pki
  - tls
  - certificates
---

# CSR

**Certificate Signing Request** — a message you send to a [[Certificate-Authority]] asking it to sign your public key and produce a [[X509-Certificate]].

## What It Contains

- Your **public key**
- **Subject identity info**: Common Name (CN), Organization (O), Country (C)
- **Subject Alternative Names (SAN)**: list of domains/IPs the cert should cover
- A **self-signature**: signed with your private key to prove you hold it

## File Format

`.csr` — typically PEM-encoded (text, starts with `-----BEGIN CERTIFICATE REQUEST-----`).

## Why SAN Matters

Modern browsers (Chrome since 2017) reject certificates that rely only on the CN field. The CSR **must** include `subjectAltName` with all intended domains/IPs, or clients will reject the issued certificate.

## Chronological Role

```
1. Generate private key + public key
2. Create CSR: embed public key + identity + SANs, sign with private key
3. Submit CSR to CA
4. CA validates identity, signs, returns .cer/.crt
```

## Common Failure

**CSR and private key mismatch**: if you generate a new private key after creating the CSR, the modulus won't match and certificate installation will fail. Always generate the CSR from the same private key.

## See Also

- [[PKI]] — the broader trust system
- [[X509-Certificate]] — what the CA returns after signing the CSR
- [[Certificate-Authority]] — the party that signs the CSR
- [[PEM-and-DER-Formats]] — encoding details
- [[Self-Signed-Certificate]] — skipping the CA: you sign your own CSR
