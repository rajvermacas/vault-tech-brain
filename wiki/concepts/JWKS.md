---
title: "JWKS"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---Entra-ID-OAuth-Reference]]"
tags:
  - authentication
  - security
  - cryptography
---

# JWKS (JSON Web Key Set)

A public endpoint that serves the cryptographic public keys needed to verify [[JWT]] token signatures. Published by [[Microsoft-Entra-ID]] and used by backend servers for local token validation.

## Endpoint

```
GET https://login.microsoftonline.com/{tenant-id}/discovery/v2.0/keys
```

**Completely public. No authentication required.** This is by design — any server receiving Microsoft-issued tokens needs to verify them.

## Response Structure

```json
{
  "keys": [
    {
      "kty": "RSA",
      "kid": "abc123",     // matches "kid" in JWT header
      "n": "sKey9x3...",   // RSA modulus
      "e": "AQAB"          // RSA exponent
    }
  ]
}
```

Microsoft maintains multiple keys simultaneously during key rotation.

## Backend Validation Steps

1. **Fetch & cache** public keys once (not per request)
2. **Verify signature** — recompute SHA-256 hash, decrypt signature with public key, compare
3. **Validate claims** — check `iss`, `aud`, `exp`, `nbf`, `tid`
4. **Authorize** — check `scp` and `roles` for required permissions

If a token arrives with an unknown `kid`, re-fetch keys and update cache (handles key rotation).

## Security Properties

- Knowing the public key gives an attacker nothing — cannot derive the private key, can only verify (not create) signatures.
- Enables **zero-network-hop validation** — microsecond latency per request after initial key fetch.

## Related

Used in the [[OAuth-2.0-Authorization-Code-Flow]] at Step 10 (server-side validation). The signature scheme is RS256 — see [[JWT]] for the full signing/verification process.
