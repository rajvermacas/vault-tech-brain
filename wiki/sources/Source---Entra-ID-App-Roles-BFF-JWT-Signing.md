---
title: "Source - Entra ID App Roles, BFF Pattern, JWT Signing"
type: source
created: 2026-04-11
updated: 2026-04-12
sources:
  - "raw/entra-id-app-roles-bff-jwt-signing.md"
tags:
  - authentication
  - oauth
  - azure
  - security
  - app-roles
  - bff
  - jwt
---

# Source - Entra ID App Roles, BFF Pattern, JWT Signing

> [!warning]
> Fact-check correction (2026-04-12): this source session correctly captured the definition-vs-assignment model for app roles, but it overstated token issuance behavior. See [[Source---Microsoft-Learn-Entra-ID-App-Roles-Fact-Check]] and [[App-Roles]] for the corrected wording around **Assignment required?**, omitted `roles` claims, and API/framework enforcement.

## Summary

A Socratic deep dive covering all properties of [[App-Registration]], tenant/subscription cardinality, [[App-Roles]] mechanics end-to-end, the two OAuth token patterns ([[OAuth-2.0-Authorization-Code-Flow#Pattern-1-SPA|Pattern 1 SPA]] vs [[BFF-Pattern|Pattern 2 BFF]]), XSS attack vectors against browser-stored tokens, and the correct mechanics of [[JWT]] RS256 signature verification.

## Main Claims

- **App Role definition fields** (confirmed via Microsoft docs): Display name, Allowed member types, Value, Description, State. There is NO `group_assignment` property on the App Role definition itself.
- **Group-to-role mapping lives on the Enterprise Application (Service Principal)**, not the App Registration. Path: Enterprise Applications → Users and Groups → Add Assignment.
- **Unassigned principals often receive tokens without a `roles` claim**, but Entra ID can also deny token issuance earlier when the Enterprise Application is configured with **Assignment required? = Yes**.
- **`roles` claim is typically omitted (not emitted as an empty array)** when no applicable role assignment is present.
- **Two valid OAuth patterns exist for SPA + backend:** Pure SPA (MSAL exchanges token in browser, no client_secret) and BFF (backend exchanges token using client_secret, browser only gets an HttpOnly session cookie).
- **Microsoft docs explicitly state:** "Public clients, which include native applications and single page apps, must not use secrets or certificates when redeeming an authorization code."
- **XSS can steal tokens via monkey-patching `window.fetch`** even when MSAL stores tokens in memory (not localStorage), by intercepting the Authorization header on outgoing requests.
- **HttpOnly cookie flag** makes the session cookie invisible to JavaScript entirely — `document.cookie` does not show it — which is the core security property of the BFF pattern.
- **JWT signature mechanics (RS256):** `signature = encrypt(SHA256(header.payload), private_key)`. The private key is NOT an input to the hash function. Hash first, then encrypt the hash output.
- **Tenant cardinality clarified:** 1 tenant → MANY App Registrations. 1 App Registration → 1 tenant (home tenant). These are two independent directional statements.

## Key Takeaways

- App Roles define RBAC within your app (`roles` JWT claim). Scopes define delegated API permissions (`scp` JWT claim). Both are checked by the backend — Entra ID only checks existence and consent at issuance.
- The Enterprise Application (Service Principal) is the assignment layer — where users/groups get mapped to App Roles. This is separate from the App Registration (definition layer).
- BFF pattern security advantage: access token never reaches browser. Browser holds only an opaque HttpOnly session cookie. XSS scripts cannot read HttpOnly cookies via `document.cookie`.
- JWT signature = encrypt(hash(header.payload), private_key). Verification = decrypt(signature, public_key) and compare. `kid` header in JWT identifies which JWKS public key to use.
- Home tenant = where you created the App Registration. Other tenants where the app runs get guest SPNs with independent granted permissions.

## Connections

- [[App-Registration]] — updated: full property breakdown across all 8 sections
- [[App-Roles]] — new concept: RBAC mechanism, definition vs assignment, roles claim
- [[Service-Principal]] — updated: cardinality table, home vs guest SPN distinction
- [[BFF-Pattern]] — new concept: backend exchanges token, HttpOnly cookie, XSS protection
- [[JWT]] — updated: RS256 signing mechanics corrected (hash then encrypt, not hash with key)
- [[JWT-Signature-Verification]] — new concept: RS256 step-by-step mechanics
- [[XSS]] — new concept: cross-site scripting, fetch monkey-patching attack vector
- [[Microsoft-Entra-ID]] — tenant/subscription cardinality, home tenant definition
- [[PKCE]] — code_verifier sent to backend in BFF pattern alongside auth code
- [[Scope]] — contrasted with App Roles (scp vs roles claim)
- [[OAuth-2.0-Authorization-Code-Flow]] — two-pattern distinction (SPA vs BFF)
- [[Source---Microsoft-Learn-Entra-ID-App-Roles-Fact-Check]] — official-doc correction for assignment-required and claim-emission nuance
