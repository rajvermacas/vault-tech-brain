---
title: "OAuth 2.0 Authorization Code Flow"
type: concept
created: 2026-04-10
updated: 2026-04-11
sources:
  - "[[Source---Entra-ID-OAuth-Reference]]"
  - "[[Source---Entra-ID-App-Roles-BFF-JWT-Signing]]"
tags:
  - authentication
  - oauth
  - security
---

# OAuth 2.0 Authorization Code Flow

The standard OAuth 2.0 grant type used by [[Microsoft-Entra-ID]] to authenticate users and issue tokens. Steps 1–4 are identical across all patterns. Steps 5–10 diverge depending on whether the app uses Pattern 1 (pure SPA) or Pattern 2 ([[BFF-Pattern]]).

## The 4 Runtime Actors

1. **Browser** — the user's browser/client
2. **Frontend** — the SPA/React app
3. **Backend** — the API server
4. **Entra ID** — the authorization server

> [!important]
> The [[App-Registration]] is NOT a 5th actor. It is a config record inside Entra ID.

## Two Patterns

### Pattern 1 — Pure SPA (MSAL exchanges token in browser)

Microsoft's recommended approach for SPAs. No `client_secret` involved in the token exchange. [[MSAL]] handles the full exchange in the browser.

| Step | From → To | What Happens |
|------|-----------|--------------|
| 1 | Browser → Entra ID | Redirect to `/authorize` with `client_id`, scopes, PKCE challenge |
| 2 | Entra ID → Browser | Microsoft login UI served |
| 3 | Browser → Entra ID | User submits credentials (app never sees them) |
| 4 | Entra ID → Browser | Redirect back with short-lived auth code + state |
| 5 | MSAL → Entra ID | **Token exchange** — `code_verifier` only, no `client_secret` |
| 6 | Entra ID → MSAL | Returns access_token, id_token, refresh_token (stored in browser memory) |
| 7 | Browser → Backend | API calls with `Authorization: Bearer <access_token>` |
| 8 | Backend (local) | Validates [[JWT]] using cached [[JWKS]] public keys — zero network hops |

### Pattern 2 — BFF / Confidential Client (backend exchanges token)

More secure for sensitive apps. Backend acts as the confidential OAuth client. Access token never reaches the browser. See [[BFF-Pattern]] for the full cookie traffic path.

| Step | From → To | What Happens |
|------|-----------|--------------|
| 1 | Browser → Entra ID | Redirect to `/authorize` with `client_id`, scopes, PKCE challenge |
| 2 | Entra ID → Browser | Microsoft login UI served |
| 3 | Browser → Entra ID | User submits credentials (app never sees them) |
| 4 | Entra ID → Browser | Redirect back with short-lived auth code + state |
| 5 | Frontend → Backend | Ships auth code + `code_verifier` to backend |
| 6 | Backend → Entra ID | **Token exchange** — `client_secret` + `code_verifier` sent server-to-server |
| 7 | Entra ID → Backend | Returns access_token, id_token, refresh_token |
| 8 | Backend → Session Store | Tokens stored server-side |
| 9 | Backend → Browser | Sets HttpOnly session cookie (token never in browser) |
| 10 | Browser → Backend | Subsequent requests carry session cookie |
| 11 | Backend (local) | Validates [[JWT]] using cached [[JWKS]] public keys — zero network hops |

> [!important]
> Microsoft docs state: "Public clients, which include native applications and single page apps, must not use secrets or certificates when redeeming an authorization code." Pattern 1 is the official SPA flow. Pattern 2 requires the backend to be a confidential client.

## Security Design

- **PKCE** (`code_challenge` / `code_verifier`) prevents authorization code interception in both patterns.
- **State parameter** prevents CSRF attacks (frontend must verify it matches).
- **`client_secret`** — Pattern 1: not used in token exchange. Pattern 2: used server-to-server only, never touches browser.
- **Refresh tokens** silently renew access tokens (up to 90 days) without re-login.
- **[[XSS]] risk** — Pattern 1 tokens in memory are vulnerable to fetch monkey-patching. Pattern 2 eliminates this via HttpOnly cookie.

## Token Types

| Token | Purpose | Audience | TTL |
|-------|---------|----------|-----|
| `access_token` | Call APIs (Bearer header) | Your API or MS Graph | ~1 hour |
| `id_token` | Know who the user is (identity) | Your app's client_id | ~1 hour |
| `refresh_token` | Silent token renewal | Token endpoint only | Up to 90 days |

> [!warning]
> Never use `id_token` to authorize API calls. Use `access_token` for that.
