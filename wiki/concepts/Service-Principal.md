---
title: "Service Principal"
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

# Service Principal (SPN)

The **runtime instance** of an [[App-Registration]] inside a specific [[Microsoft-Entra-ID]] tenant. If App Registration is the class definition, the Service Principal is the instantiated object.

## What It Holds

- `appId` — references the parent App Registration
- Object ID — SPN's own unique ID within this tenant
- **Granted permissions** — what was actually consented to (the stamp, not the wishlist)
- User and group assignments
- Sign-in logs for this tenant's users
- Conditional Access policies applied
- Admin consent records

## Relationship to App Registration

```
App Registration  =  class definition  (global, one only)
Service Principal =  instance          (one per tenant)

1 App Registration → MANY SPNs    ✔  (one per tenant)
1 SPN → MANY App Registrations    ✗  NEVER
```

## Creation Triggers

An SPN is created in an external tenant via whichever happens first:
1. **First user login** — consent screen appears, user clicks Accept
2. **Admin pre-consent** via consent URL
3. **Manual creation** via PowerShell, MS Graph, or Azure CLI

## Required in All Cases

Both single-tenant and multi-tenant apps require an SPN. Without one:
```
401 Unauthorized
"The client application {appId} is missing a service principal in tenant {tenantId}"
```

## Credentials Clarification

The `client_id` and `client_secret` live on the [[App-Registration]], not the SPN. Engineers colloquially say "SPN credentials" but technically the SPN is WHO acts at runtime, while the App Registration is WHERE credentials are stored.

## Permissions Model

See also: [[App-Registration]] (requested permissions).

Permissions follow a **wishlist → stamp** model:
- **Requested** → declared on the App Registration
- **Granted** → written to the SPN at consent time
- A token only carries granted permissions, never requested ones
