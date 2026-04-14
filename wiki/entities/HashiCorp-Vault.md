---
title: "HashiCorp Vault"
type: entity
created: 2026-04-14
updated: 2026-04-14
sources:
  - "[[Source---Understanding-CSR-and-CER]]"
tags:
  - secrets-management
  - infrastructure
  - security
---

# HashiCorp Vault

A secrets management and encryption-as-a-service tool. In the context of [[TLS]] and [[PKI]], Vault is a **server machine** that clients connect to over HTTPS — and can also act as its own [[Certificate-Authority]] via the PKI secrets engine.

## Vault's Role in PKI

### As a TLS Server

Vault terminates TLS for all incoming connections. When deployed with TLS (strongly recommended in production), it uses a standard [[X509-Certificate]] setup:

```
ca.crt      → The CA that signed Vault's cert (often self-signed)
vault.crt   → Vault's server certificate
vault.key   → Vault's private key (never leaves the Vault machine)
```

Because Vault typically uses a **self-signed CA**, clients will reject connections by default — the CA is unknown to their OS cert store.

### As a CA (PKI Secrets Engine)

Vault can also act as a [[Certificate-Authority]] to issue certificates to other services. This is a separate role from Vault's own TLS server cert. Two hierarchies:
1. **Vault's own TLS cert** — how clients trust Vault's HTTPS endpoint.
2. **Vault-as-CA** — how Vault issues certs to other services.

## Connecting to Vault: What to Do with Certificates

| Client | How to trust Vault's CA |
|---|---|
| Vault CLI | `export VAULT_CACERT=/path/to/ca.crt` |
| Python/Go/Java SDK | Pass `ca.crt` path in Vault client config |
| Kubernetes pod | Mount `ca.crt` as a Secret, reference in env/config |
| curl (testing) | `curl --cacert ca.crt https://vault:8200/v1/sys/health` |
| Linux system-wide | Copy to `/usr/local/share/ca-certificates/`, run `update-ca-certificates` |

## Anti-Pattern

```bash
export VAULT_SKIP_VERIFY=true  # ❌ never in production
```

Disabling TLS verification removes authentication entirely — you have encryption but no guarantee you're talking to the real Vault. See [[Self-Signed-Certificate]].

## Common Operational Pitfalls

- **Losing `ca.crt`**: clients can no longer verify Vault. Store it in a safe location.
- **Rotating certs without updating clients**: if the CA changes during renewal, all clients with the old `ca.crt` will break.
- **PFX exported without chain**: Azure Key Vault / App Gateway imports will fail.

## See Also

- [[PKI]] — the trust system Vault participates in
- [[TLS]] — the protocol Vault uses for connections
- [[Certificate-Authority]] — what Vault becomes with the PKI engine
- [[X509-Certificate]] — the cert Vault presents during handshake
- [[Self-Signed-Certificate]] — Vault's typical CA setup
- [[PEM-and-DER-Formats]] — format reference for Vault cert files
