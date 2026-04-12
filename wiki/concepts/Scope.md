---
title: "Scope"
type: concept
created: 2026-04-10
updated: 2026-04-12
sources:
  - "[[Source---Entra-ID-Audience-Scopes-Deep-Dive]]"
  - "[[Source---Microsoft-Learn-Permissions-and-Consent-Overview]]"
tags:
  - authentication
  - oauth
  - authorization
  - security
---

# Scope

A **named permission** in [[OAuth-2.0-Authorization-Code-Flow]] — a declaration of what specific action the token holder is allowed to perform. It is not enough to say "this person is authenticated." Scope answers the follow-up: "authenticated to do *what exactly?*"

## Concrete Examples (crick-info-buzz)

| Scope string | Meaning |
|---|---|
| `api://crick-info-buzz-backend/Scores.Read` | Allowed to read cricket scores from the backend |
| `api://crick-info-buzz-backend/Scores.Write` | Allowed to submit or update scores (admin) |
| `User.Read` | Allowed to read the signed-in user's profile (Microsoft Graph) |
| `Mail.Send` | Allowed to send email on behalf of the user (Microsoft Graph) |

## Structure of a Custom Scope String

```
api://crick-info-buzz-backend  /  Scores.Read
↑                                  ↑
Application ID URI of the API      The permission name you defined
(becomes aud in the JWT)           (becomes scp in the JWT)
```

[[MSAL]] sends this as a single string in the `scope` parameter of the authorize request. [[Microsoft-Entra-ID]] splits on `/` to derive `aud` and `scp` separately.

## Who Defines Scopes

Scopes are defined by the **API owner** — in the backend's [[App-Registration]] under "Expose an API → Add a scope." The names (`Scores.Read`, `Scores.Write`) are invented by the developer. Entra ID stores them. The frontend then requests them by their full path.

## Two-Stage Scope Enforcement

Scope is checked at **two completely separate moments**:

### Stage 1 — Entra ID at token issuance

When MSAL requests `scope=api://crick-info-buzz-backend/Scores.Read`, Entra ID checks:

1. Does `Scores.Read` exist as a defined scope on `api://crick-info-buzz-backend`? No → reject, token never issued.
2. Has the user or an admin granted [[Consent]] for this scope? No → show consent screen to Mrinal: *"crick-info-buzz wants to read your cricket scores data. Allow?"* After consent is granted, Entra records that approval for future requests.
3. Both pass → token minted with `"scp": "Scores.Read"`.

### Stage 2 — Backend middleware on every request

Every request to `GET /matches/today` carries the token in the Authorization header. The Node.js backend checks:

```javascript
if (token.scp !== "Scores.Read") {
    return res.status(403).json({ error: "Insufficient scope" });
}
```

Entra ID cannot know in advance which endpoint will be called — endpoint-level authorization is entirely the backend's responsibility.

## Why Both Stages Are Necessary

Entra ID's stage ensures the scope exists and the required [[Consent]] is in place. The backend's stage ensures the token carries the right scope for *this specific endpoint*. Example:

- Mrinal has `Scores.Read` in his token.
- He calls `POST /matches` which requires `Scores.Write`.
- Entra ID did its job correctly (the token is valid and signed).
- The backend rejects with `403` because `scp = "Scores.Read" ≠ "Scores.Write"`.

## Connections

- [[App-Registration]] — scopes are defined in the backend's App Registration under "Expose an API"
- [[JWT]] — `scp` claim in the token payload carries the granted scope
- [[MSAL]] — the library that includes scope in the authorize request
- [[OAuth-2.0-Authorization-Code-Flow]] — scope is requested in Step 1, carried in the token from Step 7 onward
- [[Consent]] — approval layer that determines whether the requested scope can be granted
- [[Microsoft-Entra-ID]] — checks scope existence and consent at issuance time
