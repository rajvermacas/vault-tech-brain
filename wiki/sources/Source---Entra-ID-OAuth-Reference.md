---
title: "Source - Entra ID OAuth Reference"
type: source
created: 2026-04-10
updated: 2026-04-10
sources:
  - "raw/rajvermacasconcepts at master.md"
tags:
  - authentication
  - oauth
  - azure
  - security
---

# Source - Entra ID OAuth Reference

## Summary

A comprehensive FAQ-style revision guide covering [[Microsoft-Entra-ID]] and [[OAuth-2.0-Authorization-Code-Flow]] for a real application called crick-info-buzz. Written as a deep-dive learning document with ASCII diagrams, step-by-step flows, and concrete examples.

## Main Claims

- [[Microsoft-Entra-ID]] is the sole Authorization Server — your app never handles user credentials.
- Exactly **4 runtime actors** exist: Browser, Frontend, Backend, Entra ID. The [[App-Registration]] is a config record inside Entra ID, not a separate actor.
- [[App-Registration]] is the blueprint (class definition); [[Service-Principal]] is the runtime instance (one per tenant).
- The `client_id` is globally unique across all Microsoft tenants — issued solely by Microsoft.
- The `client_secret` lives on the App Registration, never touches the browser, and is used only in the server-to-server token exchange (Step 6 of the traffic path).
- [[JWT]] tokens are **signed, not encrypted** — anyone can read the payload, but nobody can tamper with it.
- Backend token validation is entirely local using cached [[JWKS]] public keys — zero network calls per request.
- The `aud` claim is the #1 defense against token confusion attacks.
- Permissions follow a **requested → granted** model: requested on the App Registration (wishlist), granted on the Service Principal (stamp).
- In multi-tenant apps, each tenant's admin independently controls what permissions they grant via their own SPN.

## Key Takeaways

- App Registration = class definition (global, one only); Service Principal = instance (one per tenant that uses the app).
- The 10-step OAuth traffic path is the core mental model for understanding the auth flow end to end.
- The consent screen is the exact moment "requested → granted" transition happens — and may also create the SPN.
- RS256 signing: Microsoft signs with private key (in HSM), your backend verifies with public key (from JWKS endpoint).
- Refresh tokens silently renew access tokens (up to 90 days with continuous use), avoiding repeated user logins.
- Client secret expiry silently breaks login flows — set calendar reminders.

## Connections

- [[Microsoft-Entra-ID]] — the authorization server at the center of this flow
- [[OAuth-2.0-Authorization-Code-Flow]] — the specific grant type described
- [[JWT]] — token format used for access and ID tokens
- [[Service-Principal]] — runtime identity, one per tenant
- [[App-Registration]] — blueprint/config record for the application
- [[JWKS]] — public key endpoint for local token verification
