---
title: "Public vs Confidential Client"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---Entra-ID-Audience-Scopes-Deep-Dive]]"
tags:
  - authentication
  - oauth
  - security
  - azure
---

# Public vs Confidential Client

A fundamental split in [[OAuth-2.0-Authorization-Code-Flow]] that determines how an application proves its identity to [[Microsoft-Entra-ID]]. The split is based on one question: **can this app safely store a secret?**

## Confidential Client

An application that runs in a controlled environment inaccessible to end users. It can safely store a `client_secret`.

**Example:** The crick-info-buzz Node.js backend running on a server. Mrinal (the user) never has access to the server filesystem. The `client_secret = xK9~mP2qR8vT...` lives in an environment variable on that server.

**How it authenticates to Entra ID:** Sends `client_secret` in the POST body during the token exchange (Step 6 of [[OAuth-2.0-Authorization-Code-Flow]]). Entra ID verifies the secret matches what is registered in the [[App-Registration]].

**Azure configuration:** "Allow public client flows" = No.

## Public Client

An application that runs in an environment the end user controls. It **cannot** safely store a secret.

**Example:** The crick-info-buzz React frontend running in Mrinal's browser. Anyone can open Chrome DevTools → Sources and read every variable, including any `client_secret` that might be stored. A secret here provides zero security.

**How it authenticates to Entra ID:** Uses [[PKCE]] (Proof Key for Code Exchange) — a hash-based challenge/response that proves identity without a stored secret.

**Azure configuration:** "Allow public client flows" = Yes.

## Side-by-Side Comparison

| Property | Confidential Client | Public Client |
|---|---|---|
| Example | Node.js backend | React SPA |
| Runs in | Server (controlled) | Browser (user-controlled) |
| Can store secrets | Yes | No |
| Authenticates via | `client_secret` | [[PKCE]] |
| Has Redirect URI | No | Yes |
| "Allow public client flows" | No | Yes |
| [[App-Registration]] type | Web | Single-page application (SPA) |

## Why This Drives Two App Registrations

The public/confidential split is the core reason Microsoft recommends two separate [[App-Registration|App Registrations]] for a frontend + backend app. Mixing a public client and a confidential client in one registration creates a situation where secret management and no-secret management are entangled — which is a security and operational problem.

## Connections

- [[App-Registration]] — each type gets its own registration with different platform and secret settings
- [[PKCE]] — the authentication mechanism for public clients
- [[OAuth-2.0-Authorization-Code-Flow]] — the flow where this distinction matters most (Step 6: token exchange)
- [[MSAL]] — the library used by public clients (React SPAs) to execute token requests
- [[Microsoft-Entra-ID]] — enforces the distinction: checks secret for confidential clients, validates PKCE for public clients
