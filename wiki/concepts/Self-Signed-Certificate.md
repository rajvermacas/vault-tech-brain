---
title: "Self-Signed Certificate"
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

# Self-Signed Certificate

A certificate where the issuer and the subject are the same — you act as your own [[Certificate-Authority]] and sign your own [[CSR]]. No external trust, so clients reject it by default.

## How It Works

Instead of sending the CSR to a public CA, you sign it with your own private key:

```
openssl req -x509 -newkey rsa:4096 -keyout ca.key -out ca.crt -days 365
```

The resulting `ca.crt` is both the CA certificate and the root of trust. Any certificate signed by this CA (`vault.crt`, etc.) is only trusted by clients that have been explicitly given `ca.crt`.

## When to Use

| Context | Use self-signed? |
|---|---|
| Public-facing HTTPS | No — browsers reject it |
| Internal services (AKS, mTLS) | Yes — distribute `ca.crt` to known clients |
| Local dev | Yes — or use `VAULT_SKIP_VERIFY=true` (dev only) |
| [[HashiCorp-Vault]] in production | Yes — but properly distribute `ca.crt` |

## The Distribution Problem

Since the self-signed CA is unknown to OS/browser trust stores:

```
Client: "Who signed this cert?"
OS cert store: "Never heard of this CA" ❌
→ SSL error / connection rejected
```

**Fix**: give every client `ca.crt` and register it as trusted. For Vault specifically:

```bash
export VAULT_CACERT=/path/to/ca.crt         # Vault CLI
curl --cacert ca.crt https://vault:8200/...  # curl
# Linux system-wide:
cp ca.crt /usr/local/share/ca-certificates/
update-ca-certificates
```

## Anti-Pattern: Skipping Verification

```bash
export VAULT_SKIP_VERIFY=true   # ❌ never in production
curl -k https://vault:8200/...  # ❌ same
```

Disabling verification defeats TLS entirely — you have encryption but no authentication, leaving you open to MITM attacks.

## See Also

- [[PKI]] — the broader trust infrastructure
- [[Certificate-Authority]] — what you become when you self-sign
- [[X509-Certificate]] — the artifact produced
- [[TLS]] — the protocol that consumes these certificates
- [[HashiCorp-Vault]] — primary practical use case in this wiki
