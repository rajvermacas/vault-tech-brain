---
title: "X.509 Certificate"
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

# X.509 Certificate

A **digital certificate** — the signed artifact returned by a [[Certificate-Authority]] after processing a [[CSR]]. It proves that a public key belongs to a specific identity.

## What It Contains

- The server's **public key** (in plaintext — visible to anyone)
- **Metadata**: subject (domain/org), issuer (CA), validity period (not before / not after)
- **Subject Alternative Names (SAN)**: domains and IPs this cert covers
- The **CA's signature**: `encrypt(SHA256(all_cert_fields), CA_private_key)`

The public key is **never hidden** — the signature just proves it hasn't been tampered with and was issued by a trusted CA.

## File Extensions

`.cer` and `.crt` are effectively the same thing — both contain an X.509 certificate:

| Extension | Common Platform | Encoding |
|---|---|---|
| `.cer` | Windows, IIS | PEM or DER |
| `.crt` | Linux, Nginx, Apache | Usually PEM |

Check with `file cert.cer` or look for `-----BEGIN CERTIFICATE-----` to determine encoding.

## Verification by Client

During [[TLS]] handshake:
1. Server sends the certificate.
2. Client reads the CA's signature from the cert.
3. Client decrypts: `expected_hash = decrypt(signature, CA_public_key)`.
4. Client computes: `actual_hash = SHA256(cert_fields)`.
5. If `expected_hash == actual_hash` → cert is authentic.

## Related Formats

| Format | Contains | Use |
|---|---|---|
| `.pem` | One or more PEM-encoded items | Linux, Let's Encrypt |
| `.der` | Binary single cert | Java, older systems |
| `.pfx`/`.p12` | Key + cert + chain (PKCS#12) | Windows, Azure import |
| `.p7b`/`.p7c` | Cert chain only, no private key (PKCS#7) | Windows cert chains |

> Rule: `.pem` is a text wrapper. `.crt` can often be renamed to `.pem`. `.pfx` is a password-protected bundle of everything.

## Common Failures

- **Missing intermediate chain**: browser shows "cert not trusted" even with a valid cert — include the full CA bundle.
- **Expired intermediate cert**: entire chain breaks even if the leaf cert is valid.
- **SAN not in CSR**: cert will be rejected by modern browsers.

## See Also

- [[PKI]] — the trust system this certificate operates within
- [[CSR]] — the request that produces this certificate
- [[Certificate-Authority]] — the issuer
- [[TLS]] — the protocol that uses this certificate
- [[Self-Signed-Certificate]] — a certificate signed by its own private key
- [[PEM-and-DER-Formats]] — encoding details
- [[HashiCorp-Vault]] — practical example of internal CA + certificate distribution
