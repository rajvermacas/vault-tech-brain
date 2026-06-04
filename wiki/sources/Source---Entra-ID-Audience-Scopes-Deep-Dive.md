---
title: "Source - Entra ID Audience Scopes Deep Dive"
type: source
created: 2026-04-10
updated: 2026-04-10
sources:
  - "raw/entra-id-audience-scopes-deep-dive.md"
tags:
  - authentication
  - oauth
  - azure
  - security
  - scopes
  - pkce
---

# Source - Entra ID Audience Scopes Deep Dive

## Summary

A Socratic Q&A deep dive into how [[Microsoft-Entra-ID]] constructs the `aud` (audience) claim in [[JWT]] tokens, what `api://` identifiers actually are, how [[Scope]] works end-to-end, the distinction between [[Public-vs-Confidential-Client]], [[PKCE]] as an alternative to client secrets, [[Redirect-URI]] security, and the two-[[App-Registration]] architecture for a frontend + backend app. All concepts illustrated concretely using the **crick-info-buzz** cricket scores app.

## Main Claims

- The `aud` claim is derived directly from the **Application ID URI** set in the backend App Registration under "Expose an API" — not computed at runtime.
- `api://` is NOT a network protocol. It is a URI-shaped name badge — a globally unique string identifier used by Entra ID's registry. Nothing connects to it.
- A scope request (`api://crick-info-buzz-backend/Scores.Read`) encodes both the resource (`aud`) and the permission (`scp`) in a single string — Entra ID splits on `/`.
- **Two separate App Registrations** are required for a frontend + backend app (confirmed via Microsoft official docs). The frontend is a public client (no secret, uses [[PKCE]]); the backend is a confidential client (has `client_secret`).
- Scope is checked at **two separate moments**: by Entra ID at token issuance (does the scope exist? did the user consent?), and by the backend middleware on every request (does this token carry the right scope for this endpoint?).
- The **Redirect URI** is a security whitelist. Entra ID refuses to send the auth code to any URI not pre-registered in the App Registration — blocking auth code interception by attackers.
- **PKCE** (Proof Key for Code Exchange) replaces `client_secret` for public clients using a hash-based challenge/response proof.

## Key Takeaways

- `aud` = Application ID URI of the target API. You set it once in Azure portal. It never changes per token request.
- `api://crick-info-buzz-backend` is just the backend's name tag in Entra ID — not a URL, not a protocol.
- `client_id` in the token request = who is asking. `aud` in the token = who the token is for. Two different App Registrations, two different GUIDs.
- The frontend App Registration has NO secret, NO Application ID URI, NO exposed scopes — it only lists the backend's scopes under API Permissions.
- The backend App Registration has a secret, an Application ID URI, and defined scopes — but NO redirect URIs and NO public client flows.
- Scope enforcement is split: Entra ID checks existence + consent; your backend checks endpoint-level authorization. Neither alone is sufficient.
- Redirect URI whitelist prevents auth code interception — an attacker cannot redirect the auth code to their own server.

## Connections

- [[Microsoft-Entra-ID]] — the authorization server that mints tokens and enforces the aud/scope chain
- [[JWT]] — token format carrying `aud`, `scp`, `iss`, `exp` claims
- [[App-Registration]] — updated: now covers two-registration pattern and public vs confidential distinction
- [[OAuth-2.0-Authorization-Code-Flow]] — the flow within which all these mechanisms operate
- [[PKCE]] — the mechanism public clients use instead of client_secret
- [[Scope]] — new concept page: named permission unit in Entra ID
- [[Public-vs-Confidential-Client]] — new concept page: the fundamental split driving the two-registration pattern
- [[Redirect-URI]] — new concept page: security whitelist for auth code delivery
- [[MSAL]] — new entity page: the JavaScript library that executes token requests in the React frontend
