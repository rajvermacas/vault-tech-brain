---
title: "MSAL"
type: entity
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---Entra-ID-Audience-Scopes-Deep-Dive]]"
tags:
  - azure
  - authentication
  - library
---

# MSAL (Microsoft Authentication Library)

A JavaScript library installed in the React frontend (or any client-side app) that handles all communication with [[Microsoft-Entra-ID]] on behalf of the developer. When Mrinal clicks Login on `https://crick-info-buzz.com`, it is MSAL — not hand-written code — that constructs and sends the authorize request.

## What MSAL Does

- Constructs the authorize request with correct parameters (`client_id`, `scope`, `redirect_uri`, `state`, [[PKCE]] challenge)
- Redirects the browser to Entra ID's login page
- Handles the `/callback` route — reads the auth code from the URL, verifies the `state` parameter (CSRF check)
- Exchanges the auth code for tokens (or passes the code to the backend to do so)
- Caches tokens in memory and silently renews them using the refresh token before they expire
- Exposes the access token to your app code so it can be attached to API requests

## Key Point: MSAL Is the "Someone" Asking for Tokens

When discussing who sends token requests to Entra ID — the answer is MSAL. The developer configures MSAL once with `clientId` and `scope`. From that point on, MSAL handles all token lifecycle management automatically. No user sees this. No human types these requests.

```javascript
// Developer configures MSAL once
const msalConfig = {
    auth: {
        clientId: "a7f3c921-4d82-4b1e-9c3a-f28d01e5b6c4",  // frontend App Registration
        authority: "https://login.microsoftonline.com/{tenant-id}"
    }
};

// MSAL request includes the scope pointing to the backend
const tokenRequest = {
    scopes: ["api://crick-info-buzz-backend/Scores.Read"]
};
```

## Connections

- [[App-Registration]] — MSAL is configured with the frontend App Registration's `client_id`
- [[OAuth-2.0-Authorization-Code-Flow]] — MSAL orchestrates the entire 10-step flow from the frontend side
- [[PKCE]] — MSAL generates and manages the `code_verifier` / `code_challenge` pair automatically
- [[Scope]] — developer configures desired scopes in MSAL; MSAL includes them in every authorize request
- [[Redirect-URI]] — MSAL sends the `redirect_uri` in requests and handles the callback at that route
- [[Microsoft-Entra-ID]] — MSAL's primary counterpart; all requests go to Entra ID endpoints
- [[Public-vs-Confidential-Client]] — MSAL is used by public clients (SPAs); confidential clients use server-side SDKs
