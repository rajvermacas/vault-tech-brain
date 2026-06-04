---
title: "Redirect URI"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---Entra-ID-Audience-Scopes-Deep-Dive]]"
tags:
  - authentication
  - oauth
  - security
---

# Redirect URI

A pre-registered URL in the [[App-Registration]] that tells [[Microsoft-Entra-ID]] where it is allowed to send the auth code after a user logs in. It functions as a **security whitelist** — Entra ID will only redirect to addresses you have explicitly registered.

## Why It Exists

After Mrinal types his credentials on the Microsoft login page, Entra ID needs to send an authorization code back to the crick-info-buzz React app. But how does Entra ID know where to send it?

The [[MSAL]] library includes a `redirect_uri` in every authorize request:
```
&redirect_uri=https://crick-info-buzz.com/callback
```

Entra ID checks this against the list of registered Redirect URIs in the App Registration. If it matches → the auth code is sent there. If it does not match → the request is rejected.

## Concrete Flow (crick-info-buzz)

**Step 1** — MSAL redirects the browser to Entra ID:
```
https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/authorize
  ?client_id=a7f3c921-4d82-4b1e-9c3a-f28d01e5b6c4
  &redirect_uri=https://crick-info-buzz.com/callback
  &response_type=code
  &scope=api://crick-info-buzz-backend/Scores.Read
  &state=abc123
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cg
```

**Step 2** — Mrinal logs in. Entra ID verifies credentials.

**Step 3** — Entra ID checks: is `https://crick-info-buzz.com/callback` in the registered Redirect URIs of app `a7f3c921-...`? Yes → redirects browser to:
```
https://crick-info-buzz.com/callback
  ?code=0.AX4Tj9mK3p...
  &state=abc123
```

**Step 4** — React `/callback` route receives this. MSAL reads the `code` from the URL. It also verifies the `state` parameter matches what it originally sent — this prevents CSRF attacks. Then it sends the code to the Node.js backend for the token exchange.

## The Attack It Prevents

An attacker registers a different app and crafts a URL with:
```
&redirect_uri=https://evil-site.com/steal
```

Entra ID looks up app `a7f3c921-...` and checks its registered Redirect URIs. `https://evil-site.com/steal` is not in the list. Request rejected. The auth code never reaches the attacker's server.

## What Is Registered Here

Only the **frontend** [[App-Registration]] has Redirect URIs. The backend registration has none — the Node.js server never receives a browser redirect. It receives the auth code directly from the frontend over a server-to-server call (Step 5/6 of [[OAuth-2.0-Authorization-Code-Flow]]).

## Connections

- [[App-Registration]] — Redirect URIs are a property of the frontend registration only
- [[OAuth-2.0-Authorization-Code-Flow]] — Redirect URI is used in Steps 1 and 4 of the flow
- [[MSAL]] — includes `redirect_uri` in every authorize request and handles the callback route
- [[PKCE]] — works alongside Redirect URI: PKCE prevents code theft after delivery; Redirect URI prevents code delivery to the wrong place
- [[Microsoft-Entra-ID]] — validates the redirect_uri against the registered whitelist before sending the auth code
