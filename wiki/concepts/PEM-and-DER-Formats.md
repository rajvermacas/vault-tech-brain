---
title: "PEM and DER Formats"
type: concept
created: 2026-04-14
updated: 2026-04-14
sources:
  - "[[Source---Understanding-CSR-and-CER]]"
tags:
  - pki
  - certificates
  - file-formats
---

# PEM and DER Formats

File format differences for [[PKI]] artifacts — these are **encoding/container** differences, not different underlying concepts. The same certificate can be stored in any of these formats.

## Format Reference

| Format | Encoding | Contains | Typical Use |
|---|---|---|---|
| **.pem** | Base64 text (`-----BEGIN...-----`) | Anything — key, cert, chain | Linux, Let's Encrypt, Nginx |
| **.der** | Binary | Single cert | Java, older systems |
| **.crt** | Usually PEM | Certificate | Nginx, Apache |
| **.cer** | PEM or DER | Certificate | Windows, IIS |
| **.csr** | PEM | Signing request | Sent to CA |
| **.key** | PEM | Private key | Stays on server |
| **.pfx / .p12** | Binary (PKCS#12) | Key + cert + chain bundled | Windows, Azure import |
| **.p7b / .p7c** | PEM or DER (PKCS#7) | Cert chain only, no private key | Windows cert chains |

## Rules of Thumb

- `.pem` is a **text wrapper**. A `.crt` file can often be renamed to `.pem` and it will work.
- `.pfx` is a **password-protected bundle** of private key + certificate + chain — the "all-in-one" format.
- Use **PFX** when importing into Windows (IIS, Azure App Gateway, Key Vault).
- Use **PEM** for Linux (Nginx, Apache, Kubernetes secrets).
- Convert between them with `openssl`.

## Common Conversions

```bash
# PEM cert → DER
openssl x509 -in cert.pem -outform DER -out cert.der

# DER cert → PEM
openssl x509 -in cert.der -inform DER -out cert.pem

# PEM key + cert → PFX
openssl pkcs12 -export -out bundle.pfx -inkey server.key -in server.crt -certfile ca.crt

# PFX → PEM (extract cert)
openssl pkcs12 -in bundle.pfx -nokeys -out cert.pem

# PFX → PEM (extract key)
openssl pkcs12 -in bundle.pfx -nocerts -nodes -out key.pem
```

## Detecting Format

```bash
file cert.cer          # reports "PEM certificate" or "data" (binary DER)
head -1 cert.cer       # "-----BEGIN CERTIFICATE-----" = PEM
```

## See Also

- [[PKI]] — the system these formats serve
- [[X509-Certificate]] — the certificate artifact stored in these formats
- [[CSR]] — the signing request format
- [[TLS]] — where these files are consumed
- [[HashiCorp-Vault]] — practical example: `ca.crt` (PEM), `vault.key` (PEM)
