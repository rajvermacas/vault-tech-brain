---
title: "Source - Auth Flows Delegated OID Sub Session"
type: source
created: 2026-04-13
updated: 2026-04-13
sources:
  - "raw/session-auth-flows-delegated-oid-sub-2026-04-13.md"
tags:
  - entra-id
  - oauth
  - jwt
  - bff
  - auth-flows
---

# Source — Auth Flows, Delegated Permissions, OID/Sub Session

A deep-dive session (2026-04-13) covering corrections and new concepts across auth flow taxonomy, delegated permission semantics, BFF runtime behavior, and JWT identity claims.

## Main Claims

- The scope/App Roles split is not mutually exclusive — a token can carry both `scp` and `roles` simultaneously.
- The `scp` claim is **absent** in Client Credentials tokens; confirmed by Microsoft's official documentation. Only `roles` appears when app roles are assigned.
- "Delegated" on the resource server App Registration is a declaration to Entra ID that this API can be accessed on behalf of a signed-in user — it says nothing about which grant flow the client uses.
- In the BFF pattern, subsequent UI→Backend calls involve **no Entra ID network hop** — JWT validation is done locally via cached JWKS keys. Entra ID re-enters only for token refresh (~1 hour) and CAE revocation events.
- Delegated vs Application permissions answer one question: **is a user present?** Consent rules follow from this, not the other way around.
- `oid` is a tenant-wide stable user identifier; `sub` is app-scoped and different per client app.

## Key Evidence

- Microsoft Access Token Claims Reference explicitly states `scp` is "only included for user tokens."
- Microsoft Application and Delegated Permissions article explicitly states "The scp claim is absent in application permission tokens."
- The `roles` claim documentation confirms it is used "in place of user scopes for application tokens" in Client Credentials.

## Limitations

- This session focuses on the BFF pattern. Pure SPA (Pattern 1) behavior covered in earlier sessions.
- CAE behavior discussed conceptually; not implemented or tested against a real tenant.

## Key Takeaways

- **Auth flow taxonomy**: two dimensions — who authenticates (user vs service) × what claim carries authorization (`scp` vs `roles` vs neither). Not a clean 2×2 because `scp` cannot appear in Client Credentials.
- **Delegated on resource server** = "user must be present" signal to Entra ID. Scope existence check + consent check happen at issuance. Backend enforces at endpoint level.
- **BFF subsequent calls** = Entra ID completely off the hot path. Only session cookie → local JWT check → resource server.
- **oid vs sub**: use `oid` for DB keys; `sub` is per-app private identifier.

## Connections

- [[OAuth-2.0-Authorization-Code-Flow]] — the user-authenticated flow producing `scp` tokens
- [[Scope]] — `scp` claim, delegated permissions, two-stage enforcement
- [[App-Roles]] — `roles` claim, the service-to-service authorization mechanism
- [[BFF-Pattern]] — subsequent calls section and corrected ASCII diagram
- [[JWT]] — where `scp`, `roles`, `oid`, `sub`, `azp` live
- [[Service-Principal]] — SPN assigned to app roles in Client Credentials
- [[Consent]] — downstream of user-presence question, not the root distinction
- [[Delegated-vs-Application-Permissions]] — new concept page synthesizing the core distinction
- [[OID-and-Sub-Claims]] — new concept page
- [[Auth-Flows-Taxonomy]] — new analysis page with corrected taxonomy table
