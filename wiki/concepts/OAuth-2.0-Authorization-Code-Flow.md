---
title: "OAuth 2.0 Authorization Code Flow"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---Entra-ID-OAuth-Reference]]"
tags:
  - authentication
  - oauth
  - security
---

# OAuth 2.0 Authorization Code Flow

The standard OAuth 2.0 grant type for web applications with a backend server. Used by [[Microsoft-Entra-ID]] to authenticate users and issue tokens.

## The 4 Runtime Actors

1. **Browser** — the user's browser/client
2. **Frontend** — the SPA/React app
3. **Backend** — the API server (holds the `client_secret`)
4. **Entra ID** — the authorization server

> [!important]
> The [[App-Registration]] is NOT a 5th actor. It is a config record inside Entra ID.

## 10-Step Traffic Path

| Step | From → To | What Happens |
|------|-----------|--------------|
| 1 | Browser → Entra ID | Redirect to `/authorize` with `client_id`, scopes, PKCE challenge |
| 2 | Entra ID → Browser | Microsoft login UI served |
| 3 | Browser → Entra ID | User submits credentials (app never sees them) |
| 4 | Entra ID → Browser | Redirect back with short-lived auth code + state |
| 5 | Frontend → Backend | Ships auth code to backend |
| 6 | Backend → Entra ID | **Token exchange** — `client_secret` sent server-to-server |
| 7 | Entra ID → Backend | Returns access_token, id_token, refresh_token |
| 8 | Backend → Browser | Sets session cookie (raw JWT never in browser) |
| 9 | Browser → Backend | Subsequent requests carry session cookie |
| 10 | Backend (local) | Validates [[JWT]] using cached [[JWKS]] public keys — zero network hops |

## Security Design

- **Step 6 is the security heart** — `client_secret` never touches the browser.
- **PKCE** (`code_challenge` / `code_verifier`) prevents authorization code interception.
- **State parameter** prevents CSRF attacks (frontend must verify it matches).
- **Refresh tokens** silently renew access tokens (up to 90 days) without re-login.

## Token Types

| Token | Purpose | Audience | TTL |
|-------|---------|----------|-----|
| `access_token` | Call APIs (Bearer header) | Your API or MS Graph | ~1 hour |
| `id_token` | Know who the user is (identity) | Your app's client_id | ~1 hour |
| `refresh_token` | Silent token renewal | Token endpoint only | Up to 90 days |

> [!warning]
> Never use `id_token` to authorize API calls. Use `access_token` for that.
