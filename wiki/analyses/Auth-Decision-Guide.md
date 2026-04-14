---
title: "Auth Decision Guide"
type: analysis
created: 2026-04-14
updated: 2026-04-14
sources:
  - "[[Source---Entra-ID-OAuth-Reference]]"
  - "[[Source---Entra-ID-Audience-Scopes-Deep-Dive]]"
  - "[[Source---Entra-ID-App-Roles-BFF-JWT-Signing]]"
  - "[[Source---Auth-Flows-Delegated-OID-Sub-Session]]"
tags:
  - oauth
  - architecture
  - entra-id
  - decision-guide
---

# Auth Decision Guide

Navigational reference — common architecture decisions in Entra ID auth with direct answers and pointers to deeper pages.

---

## Should I use SPA or BFF pattern?

```
Does your app have a backend server you control?
├── No (static frontend only, no server)  → SPA Pattern
└── Yes (any backend: Node, .NET, Python)  → BFF Pattern (strongly preferred)
```

**Choose BFF unless you have no backend.**

| | SPA Pattern | BFF Pattern |
|---|---|---|
| Token storage | Browser memory (MSAL) | Server-side only |
| XSS exposure | Access token stealable via fetch monkey-patching | No token in browser — only HttpOnly cookie |
| Refresh tokens | Stored in browser (higher risk) | Stored server-side |
| [[PKCE]] required | Yes (public client) | Yes (for auth code exchange) |
| Complexity | Lower (no backend auth logic) | Higher (backend manages token lifecycle) |
| Recommendation | Use only for truly serverless apps | Default for all apps with a backend |

See [[BFF-Pattern]], [[XSS]], [[Public-vs-Confidential-Client]].

---

## Should I use App Roles or Scopes?

```
Is a user present in the flow?
├── No (service-to-service, background job) → App Roles only (roles claim)
└── Yes (user signed in)
      │
      ├── Does your API need to know WHAT the user can do?
      │   └── Yes → App Roles (coarse RBAC: admin/reader/etc.)
      │
      ├── Does your API need to know WHAT DATA the user consented to share?
      │   └── Yes → Scopes (fine-grained: Scores.Read, Profile.Write)
      │
      └── Both? → Use both: roles for authorization, scp for consent boundaries
```

| Mechanism | Claim | User required | Consent | Use for |
|---|---|---|---|---|
| **[[Scope]]** | `scp` | Yes (delegated) | User or admin | Data access consent, fine-grained operations |
| **[[App-Roles]]** | `roles` | No (application) or Yes | Admin only | RBAC role assignment, service permissions |

See [[Delegated-vs-Application-Permissions]], [[Auth-Flows-Taxonomy]].

---

## How do I protect my SPA/app from XSS stealing tokens?

```
Using SPA pattern (tokens in browser)?
└── Attacker can inject JS → monkey-patch fetch → steal Bearer token from every request
   Fix: move to BFF Pattern (no token in browser)

Using BFF pattern (HttpOnly cookie)?
└── JS cannot read HttpOnly cookies → token is safe from XSS
   Remaining risk: CSRF (mitigate with SameSite=Strict or CSRF token)
```

See [[XSS]], [[BFF-Pattern]].

---

## When does Entra ID re-enter the picture after first sign-in?

In the [[BFF-Pattern]], Entra ID is **off the hot path** for all requests between refreshes:

```
Every request:   Browser → Backend (session cookie) → Resource API (access token)
                 ← Entra ID NOT involved

Every ~1 hour:   Backend → Entra ID (refresh token exchange) → new access token
                 ← Entra ID re-enters briefly

Security event:  Entra ID → Backend (CAE revocation signal)
                 ← Entra ID pushes proactively
```

See [[Token-Expiry-and-Refresh]], [[BFF-Pattern]].

---

## What claim should I use as a user identifier in my database?

```
Use oid (Object ID), not sub.

oid = stable across all apps in the tenant → safe DB foreign key
sub = scoped per app → changes if app registration changes
```

| Claim | Scope | Stable? | Use for |
|---|---|---|---|
| `oid` | Tenant-wide | Yes | Database user IDs, audit logs |
| `sub` | Per app | Yes (but per-app) | Privacy-preserving pairwise ID only |

See [[OID-and-Sub-Claims]].

---

## Do I need one App Registration or two?

```
Does your app have a frontend AND a backend API?
├── Frontend is a public client (SPA, mobile) → Two registrations
│     Registration 1: Frontend (public client, no secret)
│     Registration 2: Backend API (exposes scopes)
│
└── Backend-only API with machine clients → One registration per service
```

The two-registration pattern separates concerns: the frontend registration is a public client with no secret; the backend registration owns the scope definitions. Mixing them into one registration creates security and lifecycle problems.

See [[App-Registration]], [[Public-vs-Confidential-Client]], [[Scope]].

---

## Single-tenant or multi-tenant app?

```
Will users from other organizations sign in?
├── No (internal tool, one company) → Single-tenant
└── Yes (SaaS product, shared service) → Multi-tenant

Single-tenant: simpler — one SPN, known issuer, no cross-tenant consent complexity
Multi-tenant: guest SPNs created per external tenant; must validate iss/tid in tokens
```

See [[Multi-Tenant-and-Guest-SPN]], [[Service-Principal]], [[Consent]].

---

## Connections

- [[Auth-Flows-Taxonomy]] — the full matrix of Entra ID auth flows
- [[BFF-Pattern]] — recommended frontend auth architecture
- [[OAuth-2.0-Authorization-Code-Flow]] — the primary grant type
- [[Delegated-vs-Application-Permissions]] — the core permission split
- [[App-Roles]] — RBAC mechanism
- [[Scope]] — consent-based permission mechanism
- [[Token-Expiry-and-Refresh]] — how tokens renew
- [[Multi-Tenant-and-Guest-SPN]] — cross-tenant scenarios
- [[OID-and-Sub-Claims]] — user identity in tokens
- [[XSS]] — the threat model driving BFF recommendation
