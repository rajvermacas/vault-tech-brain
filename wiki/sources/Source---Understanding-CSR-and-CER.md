---
title: "Source - Understanding CSR and CER"
type: source
created: 2026-04-14
updated: 2026-04-14
sources: []
tags:
  - tls
  - pki
  - certificates
  - security
  - cryptography
---

# Source - Understanding CSR and CER

A Claude conversation explaining TLS/SSL certificate lifecycle artifacts: CSR, CER/CRT, private keys, and file formats. The session culminates in a practical walkthrough of how to handle certificates when connecting to a self-hosted HashiCorp Vault.

## Main Claims

- PKI uses asymmetric cryptography and a chain of trust: private keys, public keys, and certificates each have distinct roles.
- A **CSR** (Certificate Signing Request) is a request sent to a CA containing your public key + identity info. The CA signs it and returns a **certificate** (.cer/.crt).
- The CA **signs** — it does not encrypt — the server's public key. The public key is in plaintext inside the cert; the CA appends a signature (`encrypt(SHA256(cert_fields), CA_private_key)`).
- Clients verify by decrypting the signature with the CA's public key and comparing hashes — this is signature verification, not decryption.
- File formats (.pem, .der, .pfx, .p7b) are encoding/container differences, not different concepts.
- The private key **never leaves the server** — loss requires certificate reissuance.

## Key Evidence / Steps

**Certificate lifecycle (chronological):**
1. Server generates private key + public key.
2. Server creates CSR (public key + subject info), signs CSR with own private key.
3. CSR submitted to CA.
4. CA verifies identity (domain validation, OV, EV).
5. CA signs: `signature = encrypt(SHA256(cert_fields), CA_private_key)` → produces `.cer`/`.crt`.
6. Server installs: private key + certificate + CA chain.
7. TLS handshake: server sends cert; client verifies signature using CA public key pre-installed in OS/browser.

**Vault-specific scenario:**
- Vault is the server machine, typically using a **self-signed CA** (not a public CA).
- Three artifacts produced at setup: `ca.crt`, `vault.crt`, `vault.key`.
- Clients reject Vault's cert by default because the self-signed CA is unknown to their OS cert store.
- Fix: distribute `ca.crt` to every client and configure it as a trusted CA (`VAULT_CACERT`, `--cacert`, `update-ca-certificates`).

## Failure Points Documented

| Problem | Fix |
|---|---|
| CSR/private key mismatch | Always generate CSR from the same key |
| Missing intermediate chain | Include full CA bundle in server config |
| PFX exported without chain | Re-export PFX with full chain |
| SAN not in CSR | Always add `subjectAltName` to CSR |
| Private key wrong format | Convert with `openssl pkcs8` or `openssl rsa` |
| Expired intermediate cert | Replace intermediate even if leaf cert is valid |

## Key Takeaways

- **Signing ≠ encrypting.** The CA signs (hashes then encrypts hash with CA private key); the server's public key is always in plaintext.
- **Verification = local.** The CA is only involved at issuance. All TLS verification afterward is purely local using the CA public key bundled in the OS/browser.
- **Self-signed CA = you are the CA.** No browser/client trusts it by default — you must distribute `ca.crt` yourself.
- **PFX for Windows/Azure, PEM for Linux.** Convert with `openssl`.
- **SAN is mandatory.** Chrome dropped CN-only cert support in 2017.
- **`VAULT_SKIP_VERIFY=true` is a security anti-pattern.** Never use in production.

## Connections

- [[PKI]] — the overarching trust system this describes
- [[TLS]] — the protocol these certificates enable
- [[Certificate-Authority]] — the signing party in the flow
- [[CSR]] — the signing request artifact
- [[X509-Certificate]] — the signed certificate artifact
- [[PEM-and-DER-Formats]] — file encoding formats covered
- [[HashiCorp-Vault]] — practical application discussed in the session
- [[Self-Signed-Certificate]] — used by Vault; requires manual CA distribution
