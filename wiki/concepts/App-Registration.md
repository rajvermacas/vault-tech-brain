---
title: "App Registration"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---Entra-ID-OAuth-Reference]]"
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
