# Entra ID — Audience, Scopes, App Registrations Deep Dive

Source: Conversation with LLM Wiki (2026-04-10)
App used as concrete example throughout: **crick-info-buzz** (cricket scores app)

---

## Q: How does Entra ID create the value of `aud` (audience)?

The `aud` claim in a JWT is not computed at runtime — it is directly derived from configuration in the App Registration.

### Chain:
1. You register your backend in Azure portal → **App Registration → Expose an API** → set Application ID URI to `api://crick-info-buzz-backend`
2. The React frontend (via MSAL) requests a token with `scope=api://crick-info-buzz-backend/Scores.Read`
3. Entra ID splits the scope on `/`: left side (`api://crick-info-buzz-backend`) becomes `aud`, right side (`Scores.Read`) becomes `scp`
4. The minted access token contains `"aud": "api://crick-info-buzz-backend"`

---

## Q: What is `api://`? Is it a real network protocol?

No. `api://` is NOT a network protocol. Nothing connects to it. No browser opens it. No HTTP request goes there.

It is purely a made-up string used as a name tag (a URI-shaped unique identifier). Microsoft borrowed the URI format because URIs are an already-understood system for globally unique names. They invented the `api://` prefix to signal "this is an app identifier, not a real web address."

`api://crick-info-buzz-backend` = the official registered name of the Node.js backend inside Entra ID's registry. That's it.

---

## Q: What does "canonical identity of your API resource" mean?

It means: the one official name that represents your backend in Entra ID's registry. Like a driver's license number — your name might be "Mrinal" casually, but your official government ID is a unique number. Similarly your backend has an official registered identity: `api://crick-info-buzz-backend`. Entra ID uses that name when stamping tokens.

---

## Q: What is a scope?

A scope is a named permission — a declaration of what specific action the token holder is allowed to perform.

Examples:
- `User.Read` → allowed to read this user's profile
- `Mail.Send` → allowed to send email on behalf of this user
- `api://crick-info-buzz-backend/Scores.Read` → allowed to call the Scores.Read feature of the crick-info-buzz backend
- `api://crick-info-buzz-backend/Scores.Write` → allowed to submit/update scores

---

## Q: Who are the entities involved in asking for a token?

The **MSAL library** (Microsoft Authentication Library — a JavaScript library installed in the React frontend) is the one sending the token request to Entra ID. When Mrinal clicks Login, MSAL constructs the request automatically. The developer configured MSAL once with the scope, and it includes it in every token request from then on.

---

## Q: One App Registration or two for frontend + backend?

**Two separate App Registrations.** Microsoft officially recommends this (confirmed via Microsoft Q&A, answered by Alfredo Revilla, Microsoft IAM expert).

Reasons:
- SPA (React) runs in browser → cannot keep secrets (public client)
- Backend (Node.js) runs on server → can keep secrets (confidential client)
- Mixing them creates security and management problems

Source: https://learn.microsoft.com/en-us/answers/questions/1421814/one-vs-two-app-registrations-for-an-app-with-front

---

## Side-by-side: Two App Registrations for crick-info-buzz

### App Registration 1: crick-info-buzz-frontend
- Name: `crick-info-buzz-frontend`
- Application (client) ID: `a7f3c921-4d82-4b1e-9c3a-f28d01e5b6c4` (assigned by Microsoft)
- Supported account types: This org only (single-tenant)
- Platform: Single-page application (SPA)
- Redirect URIs: `https://crick-info-buzz.com/callback`
- Client secret: NONE (public client, runs in browser)
- Expose an API → Application ID URI: NOT SET (frontend is not an API)
- Expose an API → Scopes: NONE DEFINED
- API Permissions (what it requests): `api://crick-info-buzz-backend/Scores.Read`
- Allow public client flows: YES (no secret, uses PKCE)

### App Registration 2: crick-info-buzz-backend
- Name: `crick-info-buzz-backend`
- Application (client) ID: `f9d82b14-3c71-4e90-a827-b61e59f3d204` (assigned by Microsoft)
- Supported account types: This org only (single-tenant)
- Platform: Web
- Redirect URIs: NONE (backend never redirects users)
- Client secret: `xK9~mP2qR8vT...` (server-side only, never touches browser)
- Expose an API → Application ID URI: `api://crick-info-buzz-backend`
- Expose an API → Scopes: `Scores.Read`, `Scores.Write`
- API Permissions (what it requests): NONE (it is the destination, not the caller)
- Allow public client flows: NO (confidential client, has secret)

---

## Q: What does "Allow public client flows" mean?

Two types of clients:
- **Confidential client**: App that can keep a secret (Node.js backend on a server). Uses `client_secret` to prove its identity to Entra ID.
- **Public client**: App that cannot keep a secret (React in the browser — anyone can open DevTools and read variables). Cannot use `client_secret`.

"Allow public client flows = Yes" tells Entra ID: "Do not require a `client_secret` from this app. It will prove identity using PKCE instead."

PKCE (Proof Key for Code Exchange): The React app generates a random string, hashes it, sends the hash upfront, then reveals the original string later. Only the real app that generated the original string can produce the correct hash — proves identity without a secret.

---

## Q: How are scopes checked? Who does the checking?

Two completely separate moments:

### Moment 1 — Entra ID checks scope at token issuance time
When MSAL requests `scope=api://crick-info-buzz-backend/Scores.Read`:
- Entra ID checks: does `Scores.Read` exist as a defined scope on `api://crick-info-buzz-backend`? If no → request rejected, token never issued.
- Entra ID checks: has the user consented to this scope? First time → shows consent screen to Mrinal. After consent → recorded, never shown again.
- If all checks pass → token minted with `"scp": "Scores.Read"`

### Moment 2 — Node.js backend checks scope on every API request
Every request to the backend arrives with the token in the Authorization header:
```
GET /matches/today
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
```

The backend middleware decodes the token and checks:
```javascript
if (token.scp !== "Scores.Read") {
    return res.status(403).json({ error: "Insufficient scope" });
}
```

Key insight: Entra ID checks at issuance whether the scope exists and was consented to. The backend checks at request time whether the token carries the right scope for that specific endpoint. Entra ID cannot know in advance which endpoint will be called — that is the backend's responsibility.

Example: Mrinal has Scores.Read. He tries `POST /matches` which requires `Scores.Write`. Backend rejects with 403. Entra ID already did its job correctly — the backend enforces the endpoint-level check.

---

## Q: How do Redirect URIs work in the big picture?

Exact sequence with real URLs:

**Step 1** — Mrinal clicks Login. MSAL redirects browser to:
```
https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/authorize
  ?client_id=a7f3c921-4d82-4b1e-9c3a-f28d01e5b6c4
  &response_type=code
  &redirect_uri=https://crick-info-buzz.com/callback
  &scope=api://crick-info-buzz-backend/Scores.Read
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cg
  &state=abc123
```

**Step 2** — Entra ID serves Microsoft login page. Mrinal types credentials. Entra ID verifies them.

**Step 3** — Entra ID needs to send the auth code back. It checks the `redirect_uri` in the request (`https://crick-info-buzz.com/callback`) against the registered Redirect URIs in the App Registration. It matches → Entra ID redirects browser to:
```
https://crick-info-buzz.com/callback
  ?code=0.AX4Tj9mK3p...
  &state=abc123
```

**Step 4** — React `/callback` route receives this. MSAL reads the `code`, verifies `state` matches (CSRF protection), sends code to Node.js backend.

**Step 5** — Node.js backend does token exchange with Entra ID server-to-server using `client_secret`. Entra ID returns access token.

### Why Redirect URI is a security whitelist:
If an attacker builds a fake app and puts `redirect_uri=https://evil-site.com/steal` in the authorize request, Entra ID compares it against the registered list in the App Registration. It does not match → request refused. The auth code never reaches the attacker.
