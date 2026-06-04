---
title: "Delegated vs Application Permissions"
type: concept
created: 2026-04-13
updated: 2026-04-13
sources:
  - "[[Source---Auth-Flows-Delegated-OID-Sub-Session]]"
  - "[[Source---Microsoft-Learn-Permissions-and-Consent-Overview]]"
tags:
  - oauth
  - authorization
  - entra-id
---

# Delegated vs Application Permissions

The fundamental split in [[Microsoft-Entra-ID]] between two types of access: whether a human user is present in the flow or not. Everything else — consent rules, token claims, grant flows — follows from this single distinction.

## The Core Question

> **Is a user present in this authentication flow?**

- **Yes** → Delegated permissions. The client app acts *on behalf of* the user.
- **No** → Application permissions. The client app acts *as itself*.

## Common Misconception

> "Delegated vs Application is about who provides consent — user or admin."

Consent rules are a *consequence* of the user-presence question, not the root distinction:

- Delegated → user is present → user can consent to low-risk permissions; admin must consent to high-risk ones.
- Application → no user → admin must always consent, because the permission is tenant-wide and potentially touches everyone's data.

The consent difference exists *because* of the user-presence difference.

## Token Claims

| | Delegated | Application |
|---|---|---|
| User present? | ✅ Yes | ❌ No |
| `scp` claim | ✅ Present — the delegated scope | ❌ Absent |
| `roles` claim | ✅ If app roles are assigned to the user | ✅ If app roles are assigned to the calling SPN |
| `oid` / `sub` | ✅ The user's identity | ❌ Not present (or refers to the app's SPN) |
| `azp` / `appid` | ✅ The client app | ✅ The client app |
| Grant flows | Auth Code, Device Code, ROPC, On-Behalf-Of | Client Credentials only |

> [!important]
> The `scp` claim is **absent** in application permission tokens. Confirmed by Microsoft's official documentation: "The scp claim is absent in application permission tokens." — Microsoft Learn.

## What Each Means on the Resource Server

When the resource server's [[App-Registration]] exposes a **delegated** scope, it tells [[Microsoft-Entra-ID]]:
- This API can be called on behalf of a signed-in user.
- A user must be present for Entra ID to mint this token.
- Entra ID will check: (1) does the scope exist? (2) has the user consented?

When the resource server exposes an **application** permission (via [[App-Roles]] with `Allowed member types = Applications`), it tells Entra ID:
- This API can be called by a service acting as itself with no user.
- Admin must have granted the permission to the calling SPN.

## Practical Split

```
User logs in via browser         → Delegated → scp in token
Service calls API with secret    → Application → roles in token (if assigned)
```

## Connections

- [[Scope]] — the delegated permission mechanism; produces the `scp` claim
- [[App-Roles]] — the application permission mechanism; produces the `roles` claim
- [[Consent]] — downstream consequence of this distinction
- [[JWT]] — where both `scp` and `roles` live
- [[OAuth-2.0-Authorization-Code-Flow]] — the primary delegated flow
- [[Service-Principal]] — what holds application permission grants
- [[Microsoft-Entra-ID]] — enforces the distinction at token issuance
- [[Source---Microsoft-Learn-Permissions-and-Consent-Overview]] — official doc grounding
- [[Auth-Flows-Taxonomy]] — corrected taxonomy showing how delegated vs application maps to scp/roles/neither
