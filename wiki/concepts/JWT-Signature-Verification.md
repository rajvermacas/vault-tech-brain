---
title: "JWT Signature Verification"
type: concept
created: 2026-04-11
updated: 2026-04-11
sources:
  - "[[Source---Entra-ID-App-Roles-BFF-JWT-Signing]]"
tags:
  - jwt
  - security
  - cryptography
  - authentication
---

# JWT Signature Verification

The process by which a backend validates that a [[JWT]] was genuinely issued by [[Microsoft-Entra-ID]] and has not been tampered with. Uses RS256 (RSA + SHA-256) asymmetric signing.

## Signing (Microsoft's side — at token issuance)

```
Step 1 — Hash (no key involved)
  digest = SHA256(base64(header) + "." + base64(payload))
  This is deterministic. Anyone can compute it.

Step 2 — Encrypt the digest (private key used here only)
  signature = RSA_encrypt(digest, private_key)

Full formula:
  signature = RSA_encrypt( SHA256(header.payload), private_key )
```

Microsoft's private key never leaves their HSM (Hardware Security Module).

## Verification (Backend's side — on every request)

```
Step 1 — Decrypt the signature
  expected_digest = RSA_decrypt(signature, public_key)

Step 2 — Hash independently
  computed_digest = SHA256(header.payload)

Step 3 — Compare
  if expected_digest == computed_digest → signature valid
  else → token tampered with or forged → reject
```

The public key is fetched from Entra ID's [[JWKS]] endpoint and cached locally. Backend uses the `kid` (key ID) field in the JWT header to select the correct public key from the JWKS response.

## Common Misconception

> [!warning]
> **Wrong model:** `signature = hash(private_key, header.payload)`
>
> The private key is **not** an input to the hash function. The hash (SHA-256) is computed independently of any key. The key is only used in the second step to encrypt the hash output.

## Why Two Separate Operations

- **Hash (SHA-256)** ensures **integrity** — any change to the header or payload produces a completely different digest, making tampering detectable.
- **Asymmetric encryption (RSA)** ensures **authenticity** — only Microsoft, who holds the private key, could have produced a valid signature. Anyone with the public key can verify it, but nobody can forge it.

Fusing the key into the hash function would conflate two distinct security guarantees.

## Key Lookup via `kid`

The JWT header contains a `kid` (key ID) field. Entra ID rotates its signing keys periodically. The `kid` tells the backend exactly which key in the [[JWKS]] response was used to sign this token:

```
JWT Header: { "alg": "RS256", "kid": "abc123..." }
                                         ↓
JWKS endpoint returns N public keys → pick the one with matching "kid"
```

## Connections

- [[JWT]] — the token format being verified
- [[JWKS]] — public key endpoint; backend caches keys from here
- [[Microsoft-Entra-ID]] — holds the private key, performs the signing
- [[BFF-Pattern]] — backend performs this verification in both SPA and BFF patterns
- [[Source---Entra-ID-App-Roles-BFF-JWT-Signing]] — source session
