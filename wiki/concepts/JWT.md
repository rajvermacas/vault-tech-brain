---
title: "JWT"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---Entra-ID-OAuth-Reference]]"
tags:
  - authentication
  - security
  - token-format
---

# JWT (JSON Web Token)

A compact, URL-safe token format consisting of three Base64url-encoded parts separated by dots. Used by [[Microsoft-Entra-ID]] as the format for access tokens and ID tokens in [[OAuth-2.0-Authorization-Code-Flow]].

## Structure

```
HEADER.PAYLOAD.SIGNATURE
```

### Header

Metadata about the token:
- `typ`: "JWT"
- `alg`: Signing algorithm (e.g. "RS256")
- `kid`: Key ID — tells the verifier which [[JWKS]] public key to use

### Payload (Claims)

Identity and authorization data:
- `iss`: Issuer (Entra ID URL + tenant ID)
- `aud`: Audience — **must match your API's app ID** (critical security check)
- `sub` / `oid`: User object ID
- `tid`: Tenant ID
- `exp`: Expiry (Unix timestamp)
- `scp`: Delegated scopes granted
- `roles`: App roles assigned

### Signature

The cryptographic seal — the entire 3rd segment IS the encrypted hash:

```
RSA_encrypt(SHA256(header + "." + payload), microsoft_private_key)
```

## Key Properties

- **Signed, not encrypted** — anyone can decode and read the payload, but nobody can tamper with it.
- **Self-contained** — all claims needed for authorization are inside the token.
- **Locally verifiable** — no call to the issuer needed; just the public key from [[JWKS]].

## RS256 Signing Process

**Microsoft signs** (private key in HSM):
1. `message = Base64url(header) + "." + Base64url(payload)`
2. `hash = SHA-256(message)` → 32-byte digest
3. `signature = RSA_encrypt(hash, private_key)`

**Your backend verifies** (public key from [[JWKS]]):
1. Recompute: `your_hash = SHA-256(received header + "." + payload)`
2. Decrypt: `their_hash = RSA_decrypt(signature, public_key)`
3. Compare: match → trust; mismatch → reject

## The `aud` Claim

The #1 defense against **token confusion attacks**. A token minted for App A cannot be used against App B because `aud` will mismatch. Always validate `aud` even if the signature is valid.
