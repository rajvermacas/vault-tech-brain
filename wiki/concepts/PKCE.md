---
title: "PKCE"
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

# PKCE (Proof Key for Code Exchange)

A security mechanism used by [[Public-vs-Confidential-Client|public clients]] (like React SPAs) to prove their identity to [[Microsoft-Entra-ID]] without using a `client_secret`. PKCE replaces the secret with a hash-based challenge/response proof.

## Why It Exists

[[App-Registration|Public clients]] (e.g. the crick-info-buzz React frontend) cannot safely store a `client_secret` — the app runs in Mrinal's browser, and anyone can open Chrome DevTools and read every variable. Storing a secret there is meaningless.

PKCE solves this: instead of *"prove you are who you say you are by showing a secret"*, it says *"prove you are who you say you are by showing the original string that produces this hash."*

## How It Works (crick-info-buzz Example)

**Step 1 — Frontend generates a random string (the `code_verifier`):**
```
code_verifier = "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk"
```
This stays in memory inside [[MSAL]]. Never sent to anyone yet.

**Step 2 — Frontend hashes it (the `code_challenge`):**
```
code_challenge = Base64url(SHA256(code_verifier))
              = "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cg"
```

**Step 3 — Frontend sends the hash (not the original) to Entra ID in the authorize request:**
```
GET /authorize
  ?code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cg
  &code_challenge_method=S256
  ...
```
Entra ID stores this hash and issues the auth code.

**Step 4 — Backend sends the original string during token exchange:**
```
POST /token
  ?code=0.AX4Tj9mK3p...
  &code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```
Entra ID hashes the received `code_verifier` and checks: does `SHA256(verifier) == stored_challenge`? If yes → issue the token. If no → reject.

## Security Property

An attacker who intercepts the auth code cannot use it. To exchange the code for a token, they would need the original `code_verifier` — which was only ever in [[MSAL]]'s memory and was never transmitted until the legitimate token exchange.

## Connections

- [[Public-vs-Confidential-Client]] — PKCE is the mechanism public clients use instead of client_secret
- [[App-Registration]] — "Allow public client flows" enables PKCE on the frontend registration
- [[OAuth-2.0-Authorization-Code-Flow]] — PKCE is embedded in Steps 1 and 6 of the flow
- [[MSAL]] — the library that generates and manages the code_verifier/code_challenge pair
- [[Microsoft-Entra-ID]] — validates the PKCE proof during token exchange
