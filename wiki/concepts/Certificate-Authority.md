---
title: "Certificate Authority"
type: concept
created: 2026-04-14
updated: 2026-04-14
sources:
  - "[[Source---Understanding-CSR-and-CER]]"
tags:
  - pki
  - tls
  - security
---

# Certificate Authority

A **CA** is a trusted third party that validates identity and signs [[X509-Certificate|digital certificates]]. It is the "notary" in [[PKI]]: it stamps documents so that parties who have never met can trust each other.

## Role in the Trust Chain

```
Root CA (pre-installed in OS/browser trust store)
  └── Intermediate CA (signed by Root)
        └── Leaf Certificate (signed by Intermediate, installed on server)
```

Browsers trust a small set of Root CAs. Intermediate CAs are not trusted directly — the server must send the full chain so clients can walk up to a trusted root.

## What a CA Does

1. Receives a [[CSR]] from a server.
2. Validates the applicant's identity (domain validation, OV, EV).
3. Builds the certificate: `[server_public_key + metadata + SANs]`.
4. Signs it: `signature = encrypt(SHA256(cert_fields), CA_private_key)`.
5. Returns the signed [[X509-Certificate]].

The CA is **only involved at issuance**. All subsequent [[TLS]] verification is local on the client.

## Public vs Self-Signed CA

| Type | Examples | Client trust |
|---|---|---|
| **Public CA** | DigiCert, Let's Encrypt, GlobalSign | Pre-installed in OS/browser — automatic |
| **Self-signed CA** | HashiCorp Vault default, internal PKI | Unknown to clients — must distribute `ca.crt` manually |

## Self-Signed CA Distribution

When using a self-signed CA (e.g., [[HashiCorp-Vault]]):
- Every client must receive `ca.crt` and configure it as trusted.
- Methods: `VAULT_CACERT=/path/to/ca.crt`, `--cacert ca.crt` (curl), `update-ca-certificates` (system-wide Linux), Kubernetes Secret mount.

## Vault as a CA

[[HashiCorp-Vault]] can act as a CA itself (PKI secrets engine) — issuing certificates to other services. In that role Vault plays two parts simultaneously: a TLS server (needs its own cert) and a CA that signs certs for others. These are separate cert hierarchies.

## See Also

- [[PKI]] — the system the CA operates within
- [[X509-Certificate]] — what the CA produces
- [[CSR]] — what the CA receives
- [[TLS]] — where certificates are consumed
- [[Self-Signed-Certificate]] — when there is no third-party CA
- [[HashiCorp-Vault]] — practical self-signed CA scenario
