---
title: "App Registration"
type: concept
created: 2026-04-10
updated: 2026-04-11
sources:
  - "[[Source---Entra-ID-OAuth-Reference]]"
  - "[[Source---Entra-ID-Audience-Scopes-Deep-Dive]]"
  - "[[Source---Entra-ID-App-Roles-BFF-JWT-Signing]]"
tags:
  - azure
  - identity
  - authentication
---

# App Registration

The **blueprint / definition** of an application in [[Microsoft-Entra-ID]]. Created once in the Azure portal, lives in the developer's home tenant.

## What It Holds

- `client_id` (appId) — globally unique GUID issued by Microsoft
- `client_secret` / certificates — the app's credentials
- Redirect URIs
- **Requested API permissions** — the wishlist (not yet granted)
- Exposed API scopes

## Relationship to Service Principal

App Registration is the class; [[Service-Principal]] is the instance.

- App Registration has a **one-to-one** relationship with the software
- App Registration has a **one-to-many** relationship with Service Principals (one per tenant)
- The `appId` links them together — globally unique across all Microsoft tenants

## Creation

```
Azure Portal → Microsoft Entra ID → App Registrations → New Registration
  Name, Supported accounts (single/multi), Redirect URI
  → client_id appears on overview page
  → Then: Certificates & Secrets → New Client Secret
  → COPY VALUE IMMEDIATELY (shown only once)
```

## Full Property Sections

The App Registration has 8 configuration sections:

1. **Identity (Overview)** — Display name, client_id, tenant ID, supported account types
2. **Authentication** — Platform type, Redirect URIs, implicit grant toggles, logout URL
3. **Certificates & Secrets** — Client secrets, certificates, federated identity credentials
4. **API Permissions** — Delegated and application permissions (wishlist — not yet granted)
5. **Expose an API** — Application ID URI, scopes, authorized client applications (backend registrations only)
6. **App Roles** — RBAC role definitions. See [[App-Roles]] for full mechanics.
7. **Token Configuration** — Optional claims, group claims
8. **Manifest** — Raw JSON including advanced properties like `accessTokenAcceptedVersion`

See [[Source---Entra-ID-App-Roles-BFF-JWT-Signing]] for the complete property-by-property breakdown.

## Cardinality

| Direction | Cardinality |
|---|---|
| Tenant → App Registrations | 1 → MANY |
| App Registration → Tenant (home) | MANY → 1 |
| App Registration → Service Principals | 1 → MANY (one per tenant) |

One tenant owns many App Registrations. Each App Registration belongs to exactly one home tenant. These are two independent directional statements — not a 1-to-1 relationship.

## Mental Models

| Model | App Registration | Service Principal |
|-------|-----------------|-------------------|
| OOP | Class definition | Instance |
| Architecture | Blueprint | Live identity |
| Credentials | WHERE they live | WHO uses them |
| Permissions | Wishlist (requested) | Stamp (granted) |

## Client Secret Expiry

> [!warning]
> When the `client_secret` expires, the token exchange silently returns `401` and the entire login flow breaks. Set calendar reminders and rotate proactively.

## Two-Registration Pattern (Frontend + Backend)

For any app with a separate frontend and backend, Microsoft recommends **two separate App Registrations** — not one. The reason is that a frontend (React SPA) and a backend (Node.js API) have fundamentally different security properties:

| | Frontend Registration | Backend Registration |
|---|---|---|
| Example | `crick-info-buzz-frontend` | `crick-info-buzz-backend` |
| Client type | [[Public-vs-Confidential-Client|Public client]] | [[Public-vs-Confidential-Client|Confidential client]] |
| Has `client_secret` | No | Yes |
| Platform | Single-page application (SPA) | Web |
| Redirect URIs | `https://crick-info-buzz.com/callback` | None |
| Expose an API URI | Not set | `api://crick-info-buzz-backend` |
| Exposed scopes | None | `Scores.Read`, `Scores.Write` |
| API Permissions | `api://crick-info-buzz-backend/Scores.Read` | None |
| Allow public client flows | Yes (uses [[PKCE]]) | No |

The frontend registration's `client_id` goes into [[MSAL]] config — this is "who is asking." The backend registration's Application ID URI becomes `aud` in the [[JWT]] — this is "who the token is for."

The two registrations are linked: the frontend's **API Permissions** section explicitly lists the backend's scopes, which is the formal Azure declaration that the frontend is allowed to request tokens for the backend.
