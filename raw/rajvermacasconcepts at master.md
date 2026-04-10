---
title: "rajvermacas/concepts at master"
source: "https://github.com/rajvermacas/concepts/blob/master/azure/entra-id-oauth-reference.md"
author:
published:
created: 2026-04-10
description: "concepts. Contribute to rajvermacas/concepts development by creating an account on GitHub."
tags:
  - "clippings"
---
## Microsoft Entra ID + OAuth 2.0 — Complete Reference

### A FAQ-style revision guide covering authentication, JWT, SPNs, permissions, and the full traffic path

---

## Table of Contents

1. [The Big Picture — What Is Entra ID?](#1-the-big-picture)
2. [The 4 Runtime Actors](#2-the-4-runtime-actors)
3. [App Registration vs Service Principal (SPN)](#3-app-registration-vs-service-principal)
4. [Client ID & Client Secret](#4-client-id--client-secret)
5. [The Complete Traffic Path](#5-the-complete-traffic-path)
6. [JWT Deep Dive — Structure & Signature](#6-jwt-deep-dive)
7. [Server-Side Validation](#7-server-side-validation)
8. [Permissions — Requested vs Granted](#8-permissions--requested-vs-granted)
9. [Single Tenant vs Multi-Tenant](#9-single-tenant-vs-multi-tenant)
10. [Key Mental Models Summary](#10-key-mental-models-summary)

---

## 1\. The Big Picture

### Q: What is Microsoft Entra ID and what role does it play?

Microsoft Entra ID (formerly Azure Active Directory) is the **Authorization Server**. It is the single authority that:

- Verifies who users are (authentication)
- Issues cryptographically signed tokens as proof
- Maintains app registrations, permissions, and consent records
- Publishes public keys so anyone can verify its tokens

Your app never handles user credentials. You delegate that entirely to Entra ID.

```
┌─────────────────────────┐
                    │    Microsoft Entra ID    │
                    │                         │
                    │  • Verifies identity    │
                    │  • Issues JWT tokens    │
                    │  • Holds App Registrations│
                    │  • Holds Service Principals│
                    │  • Publishes public keys │
                    └─────────────────────────┘
                              ▲     │
              "who is this?"  │     │  "here's the proof"
                              │     ▼
              ┌───────────────────────────────┐
              │         Your App              │
              │  (crick-info-buzz frontend    │
              │   + backend)                  │
              └───────────────────────────────┘
```

---

## 2\. The 4 Runtime Actors

### Q: How many systems are actually talking to each other at runtime?

Exactly **4 actors**. Not 3, not 5.

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│   1. Browser          (user's browser / client)      │
│   2. Your Frontend    (crick-info-buzz React/SPA)    │
│   3. Your Backend     (crick-info-buzz API server)   │
│   4. Microsoft Entra ID  (auth server)               │
│                                                      │
│   NOTE: App Registration is NOT a 5th actor.         │
│   It is a config record that lives INSIDE Entra ID.  │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### Q: Where does the App Registration fit then?

It is a **configuration record** stored inside Entra ID — not a separate runtime server. Think of it as a security profile that Entra ID checks internally when your app shows up. There is no separate server for it.

---

## 3\. App Registration vs Service Principal

### Q: What is an App Registration?

The **blueprint / definition** of your application. You create it once in the Azure portal. It lives in your (the developer's) home tenant.

```
App Registration holds:
├── client_id (appId)          ← globally unique GUID, issued by Microsoft
├── client_secret / certificates
├── redirect URIs
├── Requested API permissions   ← the wishlist, not yet granted
└── Exposed API scopes
```

### Q: What is a Service Principal (SPN)?

The **runtime instance** of the App Registration inside a specific tenant. It is the live identity object that Entra ID uses when your app actually runs.

```
Service Principal holds:
├── appId                       ← references the App Registration
├── Object ID                   ← SPN's own unique ID in this tenant
├── Actual granted permissions  ← what was actually consented to
├── User & group assignments
├── Sign-in logs                ← every login attempt by users in this tenant
├── Conditional Access policies applied
└── Admin consent records
```

### Q: What is the relationship between the two?

```
App Registration  =  class definition  (global, one only)
Service Principal =  instance of that class  (one per tenant)

App Registration ──── appId (shared) ────► Service Principal
                                           (links them together)
```

- App Registration has a **one-to-one** relationship with the software
- App Registration has a **one-to-many** relationship with Service Principals

### Q: Is an SPN required for single-tenant apps too, or only multi-tenant?

**Required in both cases.** No app can function without an SPN.

```
SINGLE TENANT:
  Register app in portal
       ↓ (automatic, instant)
  App Registration + SPN created together in your tenant

MULTI TENANT:
  Register app in portal
       ↓ (automatic)
  App Registration + SPN in YOUR tenant

       Later, when external tenants use the app:
       ↓ (on first consent)
  SPN created in Company A's tenant
  SPN created in Company B's tenant
```

If there is no SPN in a tenant, Entra ID returns:

```
401 Unauthorized
"The client application {appId} is missing a service principal in tenant {tenantId}"
```

### Q: 1 App Registration → Multiple SPNs? Or 1 SPN → Multiple App Registrations?

```
1 App Registration → MANY SPNs    ✔  (one per tenant that uses the app)
1 SPN → MANY App Registrations    ✗  NEVER. A SPN always maps to exactly one App Registration.
```

### Q: What triggers SPN creation in an external tenant?

Three ways — whichever comes first:

```
1. First user logs in interactively
   → consent screen appears → user clicks Accept → SPN created

2. Admin pre-consents via consent URL
   → SPN created immediately for whole tenant

3. Manually via PowerShell / MS Graph / Azure CLI
   New-MgServicePrincipal -AppId "your-client-id"
   az ad sp create --id "your-client-id"
```

### Q: Is the appId (client\_id) unique only within a tenant, or globally?

**Globally unique across the entire Microsoft identity platform** — across all tenants, everywhere.

Microsoft is the sole authority that generates and issues appIds. No tenant can claim or replicate one. This is what makes the multi-tenant SPN model work: Company A and Company B can each hold an SPN with the same `appId` without conflict, because the `appId` is a globally unique pointer back to your single App Registration.

```
Microsoft Identity Platform (flattened view)
├── appId: "a1b2c3d4..."  → crick-info-buzz        (your app)
├── appId: "b2c3d4e5..."  → some other app
└── appId: "c3d4e5f6..."  → yet another app

Tenants are just "views" over this global registry.
Each tenant holds SPNs that REFERENCE these global appIds.
```

### Q: When people say "SPN credentials" or "the SPN's client secret" — is that correct?

Technically loose language. The `client_id` and `client_secret` live on the **App Registration**, not the SPN. But since the SPN is the runtime identity doing the acting, engineers colloquially attribute the credentials to it. If you look for `client_secret` on the SPN object in the portal — you won't find it there.

```
Technically correct:
  App Registration  →  WHERE credentials are stored (client_id, secret)
  Service Principal →  WHO is acting at runtime using those credentials
```

### Q: What are "logs" on the SPN?

**Sign-in logs** — a record of every authentication event tied to this app in this tenant. Each entry captures:

```
Who:              mrinal@companyA.com
When:             2026-04-09 10:32:14 UTC
From where:       IP: 103.x.x.x, Location: Mumbai
MFA used:         Yes — Microsoft Authenticator
Result:           Success / Failure
Failure reason:   (if failed) wrong password / MFA timeout
Conditional Access: which policy evaluated, passed or blocked
```

This is visible to Company A's IT admin in their Azure portal → Enterprise Applications → crick-info-buzz → Sign-in logs. They see only their own users' activity. Full isolation per tenant.

---

## 4\. Client ID & Client Secret

### Q: Why do we need client\_id and client\_secret at all?

Every time your backend calls Entra ID's token endpoint, Microsoft needs to answer:

```
Q1: Which application is asking?   → answered by client_id
Q2: Is it really that application? → answered by client_secret
```

Without these, any random server on the internet could hit the token endpoint and claim to be your app.

### Q: What exactly is the client\_id?

A **public, non-secret UUID** that uniquely identifies your App Registration. It appears in browser URLs, frontend code, everywhere. That is completely fine — it is not a secret.

```
Example: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

### Q: What exactly is the client\_secret?

A **password** Microsoft generates for your app. Only your backend server knows it. It is the proof that the entity making the token exchange request is genuinely your app and not an impersonator.

### Q: Where does each one live in the codebase?

```
Frontend (browser — public, safe to expose):
  REACT_APP_CLIENT_ID = "a1b2c3d4-..."     ✔

Backend (server — private):
  CLIENT_ID     = "a1b2c3d4-..."           ✔ (same value)
  CLIENT_SECRET = "xK9~abc...Qr7"         ← NEVER in frontend
  TENANT_ID     = "your-tenant-id"        ← needed for Entra ID URLs
```

### Q: How do we create them?

```
Step 1: Azure Portal → Microsoft Entra ID → App Registrations → New Registration
        Name: crick-info-buzz
        Supported accounts: Single or Multi tenant
        Redirect URI: https://yourapp.com/callback
        → Click Register → client_id appears immediately on overview page

Step 2: App Registration → Certificates & Secrets → New Client Secret
        Description: "prod-backend-secret"
        Expiry: 6 / 12 / 24 months
        → Click Add → COPY THE VALUE IMMEDIATELY
          Microsoft shows it only once. It is gone after you leave the page.
```

### Q: What happens when the client secret expires?

The token exchange (Step 6 in the traffic path) silently starts returning `401` errors and your entire login flow breaks. Set a calendar reminder before expiry and rotate the secret proactively.

---

## 5\. The Complete Traffic Path

### Q: What are the exact steps, in chronological order, for a user logging into crick-info-buzz?

```
ACTORS:
  [B]  = Browser (user's browser)
  [FE] = crick-info-buzz Frontend
  [BE] = crick-info-buzz Backend
  [EID]= Microsoft Entra ID

─────────────────────────────────────────────────────────────────
STEP 1  [B] ──────────────────────────────────► [EID]
        Browser redirected to /authorize endpoint

        GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize
          ?client_id=<app-id>
          &response_type=code
          &redirect_uri=https://yourapp.com/callback
          &scope=openid profile email User.Read
          &state=<csrf-token>
          &code_challenge=<PKCE-hash>
          &code_challenge_method=S256

─────────────────────────────────────────────────────────────────
STEP 2  [EID] ────────────────────────────────► [B]
        Entra ID serves Microsoft login UI to browser

─────────────────────────────────────────────────────────────────
STEP 3  [B] ──────────────────────────────────► [EID]
        User submits credentials
        Entra ID handles: password, MFA, SSO, Conditional Access
        Your app never sees credentials.

─────────────────────────────────────────────────────────────────
STEP 4  [EID] ────────────────────────────────► [B]
        Redirect back to your frontend with short-lived auth code

        GET https://yourapp.com/callback
          ?code=0.AXoA_short_lived_code...
          &state=<original-csrf-token>

        ⚠ Frontend MUST verify state matches → prevents CSRF

─────────────────────────────────────────────────────────────────
STEP 5  [B/FE] ───────────────────────────────► [BE]
        Frontend ships the auth code to your backend

        POST /auth/callback
        { code: "0.AXoA..." }

─────────────────────────────────────────────────────────────────
STEP 6  [BE] ─────────────────────────────────► [EID]
        ★ THE SECURITY HEART ★
        Server-to-server. client_secret never touches the browser.

        POST /oauth2/v2.0/token
          client_id=<app-id>
          &client_secret=<secret>     ← stays on server only
          &grant_type=authorization_code
          &code=0.AXoA...
          &redirect_uri=https://yourapp.com/callback
          &code_verifier=<original-PKCE-value>

─────────────────────────────────────────────────────────────────
STEP 7  [EID] ────────────────────────────────► [BE]
        Tokens returned

        {
          "access_token":  "eyJ0...",   ← JWT, for calling APIs
          "id_token":      "eyJ0...",   ← JWT, for knowing who the user is
          "refresh_token": "0.ARo...",  ← opaque, for silent renewal
          "expires_in":    3600
        }

─────────────────────────────────────────────────────────────────
STEP 8  [BE] ─────────────────────────────────► [B]
        Backend sets a session cookie.
        Raw JWT never touches the browser.

─────────────────────────────────────────────────────────────────
STEP 9  [B] ──────────────────────────────────► [BE]
        Every subsequent API call carries the session cookie

─────────────────────────────────────────────────────────────────
STEP 10 [BE] ─────────────────────────── (no outbound call)
        Backend validates JWT locally using cached public keys.
        Zero network hops. Microseconds. See Section 7.

─────────────────────────────────────────────────────────────────
```

### Q: Why does the token exchange (Step 6) happen on the backend and not the frontend?

Because it requires the `client_secret`. The secret must never touch the browser. If it did, any user could open DevTools and steal it, then impersonate your entire application.

### Q: What is the refresh token and when is it used?

Access tokens expire in ~1 hour. When they do, your backend silently exchanges the refresh token for a new one without the user logging in again.

```
POST /oauth2/v2.0/token
  grant_type=refresh_token
  &refresh_token=0.ARo...
  &client_id=...
  &client_secret=...

→ New access_token + new refresh_token returned
```

Refresh tokens can last up to 90 days with continuous use.

---

## 6\. JWT Deep Dive

### Q: What does a JWT look like?

Three parts, separated by dots. Each part is Base64url encoded.

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6ImFiYzEyMyJ9
.
eyJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20v...
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

│──── HEADER ────│──────────── PAYLOAD ────────────│── SIGNATURE ──│
   (Base64url)           (Base64url)                  (Base64url)
```

### Q: What is in each part?

**Header** — metadata about the token itself:

```
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "abc123"    ← Key ID: tells verifier which public key to use
}
```

**Payload** — claims about the user and the token:

```
{
  "iss": "https://login.microsoftonline.com/{tenant-id}/v2.0",
  "aud": "api://your-api-id",     ← must match YOUR API's app ID
  "sub": "user-object-id",
  "oid": "user-object-id",
  "tid": "tenant-id",
  "exp": 1712700000,              ← expiry (Unix timestamp)
  "iat": 1712696400,              ← issued at
  "scp": "User.Read",             ← delegated scopes granted
  "roles": ["Admin"],             ← app roles
  "name": "Mrinal",
  "preferred_username": "mrinal@company.com"
}
```

**Signature** — the cryptographic seal:

```
NOT a JSON object. No property name. No fields.
The entire 3rd segment IS the encrypted hash.

= RSA_encrypt(
    SHA256(Base64url(header) + "." + Base64url(payload)),
    microsoft_private_key
  )
```

### Q: The payload is visible to anyone — is that a security problem?

No. The payload is **signed, not encrypted**. Anyone can decode and read it — but nobody can tamper with it. If even one character changes in the payload, the signature verification fails instantly. The signing is what provides the security guarantee, not secrecy.

### Q: Where exactly is "the hash Microsoft originally computed" stored inside the JWT?

There is **no property** that holds it. The **entire 3rd segment** IS the hash, RSA-encrypted with Microsoft's private key.

```
part1 . part2 . part3
               ──────
               This IS it.
               RSA_encrypt(SHA256(part1 + "." + part2), private_key)

When you RSA_decrypt(part3, public_key)
→ you get back the original 32-byte SHA-256 hash Microsoft computed.
```

### Q: Which token should be used for what?

```
access_token   → Call APIs (put in Authorization: Bearer header)
                 Audience = your API or Microsoft Graph
                 TTL: ~1 hour

id_token       → Know who the user is (UI display, session identity)
                 Audience = your app's client_id
                 TTL: ~1 hour
                 ⚠ NEVER use id_token to authorize API calls

refresh_token  → Silently get new access tokens
                 Audience = token endpoint only
                 TTL: up to 90 days with continuous use
```

---

## 7\. Server-Side Validation

### Q: Does the backend call Microsoft on every API request to validate the token?

**No.** Validation is done entirely locally. The backend uses Microsoft's cached public keys to verify the signature — zero outbound network calls per request.

### Q: What is the JWKS endpoint? Does it require authentication?

```
GET https://login.microsoftonline.com/{tenant-id}/discovery/v2.0/keys
```

**Completely public. No authentication required.** This is by design — any server that receives Microsoft-issued tokens needs to verify them, and Microsoft has no way to know in advance which servers those are. The public key is *meant* to be shared openly.

Knowing the public key gives an attacker nothing — you cannot derive the private key from it, and you can only verify signatures with it, not create them.

### Q: What does the JWKS endpoint return?

```
{
  "keys": [
    {
      "kty": "RSA",
      "kid": "abc123",     ← matches "kid" in token header
      "n": "sKey9x3...",   ← RSA modulus (the actual key material)
      "e": "AQAB"          ← RSA exponent
    },
    {
      "kid": "def456"      ← Microsoft keeps multiple keys during rotation
    }
  ]
}
```

### Q: What are the exact validation steps the backend performs?

```
STEP 1 — Fetch & cache public keys (once, not per request)
  GET /discovery/v2.0/keys
  Cache the keys in memory.
  If a token arrives with unknown "kid" → re-fetch keys, update cache.

STEP 2 — Verify the signature
  recomputed_hash = SHA256(header + "." + payload)
  decrypted_hash  = RSA_decrypt(signature, public_key matching kid)
  recomputed_hash === decrypted_hash?
    ✔ match    → token is untampered
    ✘ no match → REJECT immediately. Stop here. Return 401.

STEP 3 — Validate claims
  iss === "https://login.microsoftonline.com/{tenant-id}/v2.0"  ?
  aud === "api://your-api-app-id"   ← CRITICAL. Your #1 defense.
  exp > now()                       ← not expired
  nbf <= now()                      ← already active
  tid === your-tenant-id            ← especially for multi-tenant

STEP 4 — Authorize
  scp contains required scope?   → User.Read, etc.
  roles contains required role?  → Admin, etc.
  ✔ all pass → allow request
  ✗ any fail → 403 Forbidden
```

### Q: Why is the aud claim so critical?

It prevents **token confusion attacks**. If App A and App B both use Entra ID, a token minted for App A cannot be used against App B — because `aud` will mismatch. Always check `aud` even if the signature is valid.

### Q: What is the RS256 signing process in plain English?

```
MICROSOFT SIGNS (private key — never leaves Microsoft's HSM):
  1. message   = Base64url(header) + "." + Base64url(payload)
  2. hash      = SHA-256(message)          → 32-byte digest
  3. signature = RSA_encrypt(hash, private_key)  → becomes 3rd segment

YOUR BACKEND VERIFIES (public key — downloaded from JWKS, cached):
  1. Recompute: your_hash = SHA-256(received header + "." + payload)
  2. Decrypt:   their_hash = RSA_decrypt(signature, public_key)
  3. Compare:   your_hash === their_hash?
                  ✔ → Microsoft sealed this exact content. Trust it.
                  ✗ → Something was tampered with. Reject.
```

---

## 8\. Permissions — Requested vs Granted

### Q: What is the difference between requested and granted permissions?

```
REQUESTED (lives on App Registration):
  The wishlist. What your app claims it might need.
  Declared by you, the developer, at registration time.
  Nothing is granted yet — like a visa application listing activities.

GRANTED (lives on Service Principal):
  The actual stamp. What has been approved in a specific tenant.
  Written to the SPN at consent time.
  A token only carries granted permissions, not requested ones.
```

### Q: Who grants permissions — and does it depend on the type of permission?

```
┌───────────────────────────────────────────────────────────────┐
│  Permission Type              Who Can Grant                   │
│  ─────────────────────────────────────────────────           │
│  Low-risk delegated           The USER themselves             │
│  (User.Read, profile, email)  → the consent screen they see  │
│                                                               │
│  High-risk delegated          TENANT ADMIN only               │
│  (Mail.Read, Files.ReadWrite) → "Grant for all users" button  │
│                                                               │
│  Application permissions      TENANT ADMIN only. Always.      │
│  (User.Read.All, Mail.Read.All) No exceptions.                │
└───────────────────────────────────────────────────────────────┘
```

### Q: That consent screen I've seen when signing into apps — what exactly is that moment?

That is the **exact moment** the "requested → granted" transition happens. When you click Accept:

1. Your consent is written as a grant onto the SPN in your tenant
2. If the SPN didn't exist yet — it is created right now
3. The token issued after this contains only the scopes you just granted

### Q: What does individual user consent vs admin consent look like on the SPN?

```
INDIVIDUAL USER CONSENT:
  SPN (Company A's tenant)
  └── Consent grants:
      ├── User A → User.Read ✔   (User A clicked Accept)
      ├── User B → (nothing)     (hasn't logged in yet)
      └── User C → (nothing)     (hasn't logged in yet)
  → Each user sees consent screen on their first login

ADMIN CONSENT:
  SPN (Company A's tenant)
  └── Consent grants:
      └── All users → User.Read ✔  (admin consented once for everyone)
  → No user ever sees a consent screen
```

### Q: Does the SPN need to exist before a user can log in?

For **interactive OAuth login** (Authorization Code Flow): No. The consent screen creates the SPN on the spot.

For **non-interactive flows** (client credentials, direct API calls): Yes. The SPN must pre-exist. Without it you get a 401 error.

---

## 9\. Single Tenant vs Multi-Tenant

### Q: What does single tenant vs multi-tenant mean for crick-info-buzz?

```
SINGLE TENANT:
  Only users in YOUR company's tenant can log in.
  Use case: internal tools built for your own org.

  App Registration (your tenant)
       └── SPN (your tenant)    ← 1 SPN, auto-created at registration

MULTI TENANT:
  Users from ANY Microsoft tenant can log in.
  Use case: SaaS products sold to external companies (like crick-info-buzz).

  App Registration (your tenant)
       ├── SPN (your tenant)          ← auto-created at registration
       ├── SPN (Company A's tenant)   ← created when they first consent
       └── SPN (Company B's tenant)   ← created when they first consent
```

### Q: Can Company A's admin control what permissions they grant, independently of Company B?

Yes. Each tenant's admin independently decides what to approve. Their decisions are stored on their own SPN and have no effect on any other tenant's SPN.

```
App Registration: requests User.Read + Mail.Read.All

Company A's SPN:  User.Read ✔   Mail.Read.All ✔   (approved both)
Company B's SPN:  User.Read ✔   Mail.Read.All ✗   (approved only one)
```

---

## 10\. Key Mental Models Summary

### The class / instance model

```
App Registration  =  class definition   (one, global, yours)
Service Principal =  instance           (one per tenant)
```

### The blueprint / live identity model

```
App Registration  =  the blueprint      (what the app IS)
Service Principal =  the live identity  (what the app DOES at runtime)
```

### The credentials / actor model

```
App Registration  =  WHERE credentials live  (client_id, client_secret)
Service Principal =  WHO acts at runtime     (permissions, logs, policies)
```

### The wishlist / stamp model

```
Requested permissions  =  the wishlist       (App Registration)
Granted permissions    =  the approved stamp (Service Principal)
```

### The global / local model

```
appId / client_id  =  globally unique, issued by Microsoft, immutable
Object ID (SPN)    =  local to a tenant, different per tenant
```

### The sign / verify model

```
Microsoft's private key  =  signs tokens     (never leaves Microsoft)
Public key (JWKS)        =  verifies tokens  (published openly, anyone can use)
```

### The authentication / authorization model

```
Authentication  =  proving WHO you are     (valid JWT, correct signature)
Authorization   =  proving WHAT you can do (correct scp/roles in JWT)
A valid token proves identity — not permission. Always check both.
```

---

*Document compiled from a deep-dive learning session on Microsoft Entra ID + OAuth 2.0 / JWT authentication flows.* *Last updated: April 2026*