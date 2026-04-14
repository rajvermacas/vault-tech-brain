---
title: "Token Expiry and Refresh"
type: concept
created: 2026-04-14
updated: 2026-04-14
sources:
  - "[[Source---Auth-Flows-Delegated-OID-Sub-Session]]"
  - "[[Source---Entra-ID-OAuth-Reference]]"
tags:
  - oauth
  - tokens
  - entra-id
  - security
---

# Token Expiry and Refresh

How Entra ID tokens expire and how clients silently renew them without interrupting the user.

## Token Lifetimes (Entra ID Defaults)

| Token | Default Lifetime | Configurable? |
|---|---|---|
| **Access token** | ~1 hour (3600s) | Yes (Token Lifetime Policy) |
| **ID token** | ~1 hour | Yes |
| **Refresh token** | Up to 90 days (sliding window) | Yes |
| **Refresh token (inactive)** | 24 hours with no use | Inherent behavior |

Access tokens are short-lived by design — if one leaks, it expires quickly. Refresh tokens are long-lived but single-use (rotated on each exchange).

## Refresh Token Exchange

When an access token nears expiry, the client exchanges the refresh token for a fresh access token **without prompting the user**:

```
Client → Entra ID:  POST /token
                    grant_type=refresh_token
                    refresh_token=<RT>
                    client_id=...
                    scope=...

Entra ID → Client:  new access_token
                    new refresh_token  ← old RT is invalidated
                    new id_token (optional)
```

**Refresh token rotation**: Entra ID issues a new refresh token on every exchange and immediately invalidates the old one. If an attacker steals an old RT and tries to use it after rotation, Entra ID detects the replay and revokes the entire token family (all RTs for that session).

## In the BFF Pattern

In the [[BFF-Pattern]], the backend holds all tokens. The user's browser never sees them. Token renewal is invisible:

1. Access token expires (~1 hour).
2. Backend silently calls Entra ID with the stored refresh token.
3. Entra ID returns new access token + new refresh token.
4. Backend stores the new pair; user session continues uninterrupted.

[[Microsoft-Entra-ID]] is **off the hot path** for all requests between refreshes — only re-enters for token renewal (~1/hour) and CAE signals.

## CAE (Continuous Access Evaluation)

CAE allows Entra ID to **proactively revoke** access tokens before they expire when a security event occurs:

| Trigger | CAE action |
|---|---|
| User account disabled | Immediate revocation signal to resource server |
| Password reset | Revocation signal |
| Conditional Access policy changes | Re-evaluation required |
| Sign-in risk elevated | Revocation signal |

Without CAE, a compromised account could remain active for up to 1 hour (full access token lifetime). CAE reduces this to near-real-time. CAE-capable resource servers (e.g., Microsoft Graph) check revocation claims in tokens; non-CAE resources still rely on expiry.

## When Refresh Tokens Become Invalid

A refresh token is invalidated when:
- It has been used and rotated (the old RT is consumed).
- The user explicitly signs out.
- The user's password is changed.
- An admin revokes all refresh tokens (PowerShell: `Revoke-AzureADUserAllRefreshToken`).
- The refresh token has been inactive for 24 hours (sliding-window reset).
- The 90-day maximum lifetime is exceeded.
- A CAE event triggers full session revocation.

When a refresh token is invalid, the user must re-authenticate (full interactive sign-in).

## Sliding Window vs Absolute Lifetime

Entra ID refresh tokens use a **sliding window** model by default:
- Every time the RT is exchanged, the 90-day clock resets.
- An active user never needs to re-authenticate.
- An inactive user (no exchanges for 90 days) must re-authenticate.

This can be adjusted via a **Token Lifetime Policy** assigned to the App Registration or Service Principal.

## See Also

- [[OAuth-2.0-Authorization-Code-Flow]] — the flow that issues the initial token set
- [[JWT]] — the format of access and ID tokens
- [[BFF-Pattern]] — backend holds tokens; handles silent refresh transparently
- [[Conditional-Access]] — policies that interact with CAE to trigger revocation
- [[Microsoft-Entra-ID]] — the authorization server managing token lifetimes
- [[Delegated-vs-Application-Permissions]] — refresh tokens only exist in delegated flows (user present)
