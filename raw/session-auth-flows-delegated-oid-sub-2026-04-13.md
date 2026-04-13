---
title: "Session — Auth Flows Taxonomy, Delegated Permissions, OID/Sub Claims"
source: "cowork-session"
author: "Mrinal + Claude"
published:
created: 2026-04-13
description: "Deep dive session covering: corrected 4-flow auth taxonomy, what delegated means on the resource server, BFF subsequent call path, scp absence in client credentials (Microsoft-doc fact-checked), delegated vs application core distinction, oid vs sub claims, BFF ASCII correction."
tags:
  - entra-id
  - oauth
  - jwt
  - bff
  - auth-flows
---

## Auth Flows Taxonomy — Corrected

### Original (incorrect) framing

User proposed 4 flows:
- a. User login
  - a.1 With scope, without app roles
  - a.2 Without scope, with app roles
- b. Service-to-service (client id + secret)
  - b.1 With scope, without app roles
  - b.2 Without scope, with app roles

### Corrections

**1. Scope and App Roles are not mutually exclusive.**
A single JWT can carry both `scp` and `roles` claims simultaneously. The framing "with scope, WITHOUT app roles" implies they are mutually exclusive — they are not. The correct framing is: what claims does the backend check to authorize the request?

**2. Client Credentials flow does not produce `scp` claim.**
The `scp` claim only appears in tokens from user-authenticated flows (Authorization Code, Device Code, ROPC, On-Behalf-Of). In Client Credentials, there is no user context, so there are no delegated permissions and no `scp`. The request uses `scope=api://your-api/.default` as required syntax, but the token carries `roles` (if app roles are assigned to the calling SPN), not `scp`.

This was fact-checked against Microsoft's official documentation:
- Access Token Claims Reference: "scp — Only included for user tokens."
- Application and Delegated Permissions article: "The scp claim is absent in application permission tokens."

**3. Corrected taxonomy:**

| | User-authenticated (Auth Code Flow) | Service-to-service (Client Credentials) |
|---|---|---|
| Fine-grained authz via scope (`scp`) | ✅ delegated permissions, user/admin consent | ❌ scp does not appear |
| Fine-grained authz via App Roles (`roles`) | ✅ user/group assigned to role on Enterprise App | ✅ service principal assigned to app role |
| Coarse authz (identity only, no claim check) | ✅ authenticate user, trust fully | ✅ authenticate SPN, trust fully |

---

## What "Delegated" on Resource Server App Registration Means to Entra ID

### The declaration

When the API owner defines a delegated scope on the resource server's App Registration ("Expose an API → Add a scope"), they are declaring: "This API can be accessed on behalf of a signed-in user." Entra ID uses this as a lookup at token issuance time.

### What Entra ID does with it

When the client's `/authorize` request includes `scope=api://resource-server/Scores.Read`, Entra ID:
1. Checks: does `Scores.Read` exist as a delegated scope on that resource server? If no → reject.
2. Checks: has the user consented to this client using this scope? If no → show consent screen.
3. If both pass → mints `access_token` with `scp: "Scores.Read"` + user identity claims.

### What "delegated" does NOT indicate

- It does not specify which grant flow the client uses (Auth Code, Device Code, ROPC, etc.).
- It does not indicate anything about PKCE, client_secret, or auth code mechanics.
- It does not gate which client app can call the API (that's handled by API Permissions tab on the client's App Registration).

### What the claims carry

- `scp` — what action is permitted
- `azp`/`appid` — which app is calling
- `oid`/`sub` — whose identity is present (the user's)

---

## BFF Subsequent Calls — Entra ID Not Involved

After the initial authentication in the BFF pattern:

**Every subsequent UI → Backend call:**
- Browser sends HttpOnly session cookie
- Backend looks up session, retrieves stored `access_token`
- Backend validates JWT locally using cached JWKS public keys (zero network hops to Entra ID)
- Backend forwards Bearer token to resource server
- Resource server also validates locally

**Entra ID re-enters only in two cases:**
1. Token expiry (~every 1 hour): backend silently calls Entra ID's `/token` endpoint with the stored `refresh_token` to get a fresh `access_token`. User doesn't notice.
2. Continuous Access Evaluation (CAE): if something significant changes mid-session (account disabled, password reset, Conditional Access policy change), Entra ID can push a revocation signal.

```
Every UI → Backend call:    Session cookie → local JWT check (no Entra ID)
Every ~1 hour (silently):   Backend → Entra ID (refresh token exchange)
On security events:         Entra ID → Backend (CAE revocation signal)
```

### Corrected BFF ASCII Diagram

```
                    AUTHENTICATION (once)

Browser          BFF Backend          Entra ID        Resource Server
   │                  │                   │                  │
   │──/authorize─────────────────────────►│                  │
   │                  │                   │ login UI         │
   │◄─────────────────────────────────────│                  │
   │──user credentials──────────────────►│                  │
   │◄─────────────────auth code──────────│                  │
   │──auth code──────►│                  │                  │
   │                  │──code+secret────►│                  │
   │                  │◄─access_token────│                  │
   │                  │  (scp, oid, azp) │                  │
   │                  │──store token (session store)         │
   │◄──HttpOnly cookie│                  │                  │

                    SUBSEQUENT CALLS (every request)

Browser          BFF Backend          Entra ID        Resource Server
   │                  │                   │                  │
   │──session cookie─►│                  │                  │
   │                  │ lookup token      │                  │
   │                  │ validate JWT locally (JWKS cached)   │
   │                  │──Bearer token───────────────────────►│
   │                  │                  │                  │ check scp/roles
   │◄─────────────────────────────────response──────────────│
```

---

## Delegated vs Application — The Core Distinction

### Common misconception

User's initial summary: "Delegated and Application both try to solve one problem: who provides consent — user or admin."

### Correction

Consent is a downstream consequence. The actual problem both answer is: **whose identity is embedded in the token — is a user present or not?**

- **Delegated** → user is present → token carries user identity (`oid`, `sub`) → API knows whose data is being touched → consent from the user makes sense because a real person is authorizing access to their own data.
- **Application** → no user → token carries only the app's identity → API knows which service is calling → admin-only consent because the permission is tenant-wide and potentially touches everyone's data.

**One-line summary**: Delegated vs Application answers "is a user present?" — consent rules, token claims, and authorization behavior all follow from that single answer.

---

## OID vs Sub Claims

Both are user identifiers in the JWT, but serve different purposes.

### `oid` — Object ID
- The user's **immutable, tenant-wide** unique ID in Entra ID.
- Same value across every app the user signs into within the same tenant.
- Use as primary key when storing user records in a database — never changes even if the user changes email or username.

### `sub` — Subject
- The user's **app-specific** unique ID.
- Same user signing into two different apps gets a *different* `sub` in each app's token.
- Exists for privacy — so apps can't correlate users across services by comparing identifiers.

| | `oid` | `sub` |
|---|---|---|
| Scope | Tenant-wide | Per app |
| Stable across apps? | Yes | No |
| Use for | DB primary key, cross-app identity | Single-app session identity |
| Changes if user renamed? | No | No |

**Rule of thumb**: Use `oid` to reliably identify the same human across the system. Use `sub` only for single-app isolation where cross-app correlation is not needed.

---

## Fact-Check Results

Claim checked: "scp does not appear in client credentials tokens"
Source: Microsoft Learn — Access Token Claims Reference + Application and Delegated Permissions article
Result: **CONFIRMED**
- "scp — Only included for user tokens." (Access Token Claims Reference)
- "The scp claim is absent in application permission tokens." (Application and Delegated Permissions)
- URLs: https://learn.microsoft.com/en-us/entra/identity-platform/access-token-claims-reference and https://learn.microsoft.com/en-us/troubleshoot/entra/entra-id/app-integration/application-delegated-permission-access-tokens-identity-platform
