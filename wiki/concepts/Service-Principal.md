---
title: "Service Principal"
type: concept
created: 2026-04-10
updated: 2026-04-22
sources:
  - "[[Source---Entra-ID-OAuth-Reference]]"
  - "[[Source---Entra-ID-App-Roles-BFF-JWT-Signing]]"
  - "[[Source---Microsoft-Learn-Conditional-Access-Overview]]"
  - "[[Source---AKS-Workload-Identity-Federated-Token]]"
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
- [[Conditional-Access|Conditional Access]] policies applied
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

## Home SPN vs Guest SPN

When an App Registration is used across multiple tenants:

- **Home SPN** — created automatically in the home tenant when the App Registration is created. No consent needed.
- **Guest SPN** — created in each external tenant when a user from that tenant consents (or admin pre-consents). Same `appId` as home SPN, but different Object ID and independent granted permissions.

Each tenant's admin independently controls what permissions they grant their guest SPN. A tenant's [[Conditional-Access|Conditional Access]] policies apply to the guest SPN in that tenant.

## App Role Assignments

The SPN (Enterprise Application) is also where [[App-Roles]] are assigned to users and groups:

```
Enterprise Applications → [Your App] → Users and Groups → Add Assignment
  → Select user or security group
  → Select App Role (defined on the App Registration)
```

This is the assignment layer — separate from the definition layer (App Registration).

## Permissions Model

See also: [[App-Registration]] (requested permissions).

Permissions follow a **wishlist → stamp** model:
- **Requested** → declared on the App Registration
- **Granted** → written to the SPN at consent time
- A token only carries granted permissions, never requested ones

## Workload Identity Context

In [[Azure-Workload-Identity]], an [[AKS]] pod authenticates to Azure as a managed identity (a special-purpose SPN). The pod never holds a client secret; instead it uses [[Federated-Credentials]] to exchange a Kubernetes-issued [[Projected-Service-Account-Token]] for an Azure access token tied to the managed identity's SPN. This is always application-permission auth (see [[Delegated-vs-Application-Permissions]]) — no user is present in the flow.
