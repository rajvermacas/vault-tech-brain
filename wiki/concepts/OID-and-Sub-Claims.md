---
title: "OID and Sub Claims"
type: concept
created: 2026-04-13
updated: 2026-04-13
sources:
  - "[[Source---Auth-Flows-Delegated-OID-Sub-Session]]"
tags:
  - jwt
  - identity
  - entra-id
---

# OID and Sub Claims

Two [[JWT]] claims that both identify a user, but with different scopes of uniqueness. Only present in tokens from user-authenticated flows ([[Delegated-vs-Application-Permissions|delegated permissions]]); absent in Client Credentials tokens.

## `oid` — Object ID

- The user's **immutable, tenant-wide** unique identifier in [[Microsoft-Entra-ID]].
- Same value across every application the user signs into within the same tenant.
- Does not change if the user changes their email, username, or display name.
- Use as the **primary key** when storing user records in a database.

```json
"oid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
```

## `sub` — Subject

- The user's **application-scoped** unique identifier.
- Same user signing into two different apps gets a **different** `sub` in each app's token.
- Exists for privacy — prevents apps from correlating users across services by comparing identifiers.
- Useful only when building something isolated to a single app.

```json
"sub": "xQ9kP2rT7mLwVnA3dGsE8fYhZcBjIoUv"
```

## Comparison

| | `oid` | `sub` |
|---|---|---|
| Uniqueness scope | Tenant-wide | Per app (per `client_id`) |
| Same across apps? | ✅ Yes | ❌ No — different per app |
| Changes on rename? | No | No |
| Primary use | DB primary key, cross-app identity | Single-app session identity |
| Privacy design | Correlatable across apps | Prevents cross-app correlation |

## Rule of Thumb

- **Use `oid`** when you need to reliably identify the same human across your system or store a persistent user record.
- **Use `sub`** only when building a single-app feature where cross-app identity correlation is explicitly not needed.

> [!warning]
> Never use `sub` as a database primary key if you might ever need to correlate users across multiple apps or services in the same tenant. Two different apps will see two different `sub` values for the same user.

## Connections

- [[JWT]] — both are payload claims in the access token and id token
- [[Delegated-vs-Application-Permissions]] — `oid`/`sub` only appear in delegated (user-present) tokens
- [[Microsoft-Entra-ID]] — issues and populates both claims
- [[App-Registration]] — the `client_id` determines the `sub` value (per-app scoping)
- [[Source---Auth-Flows-Delegated-OID-Sub-Session]] — session where this was defined
