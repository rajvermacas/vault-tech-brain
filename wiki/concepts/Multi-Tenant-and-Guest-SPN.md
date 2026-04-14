---
title: "Multi-Tenant and Guest SPN"
type: concept
created: 2026-04-14
updated: 2026-04-14
sources:
  - "[[Source---Entra-ID-OAuth-Reference]]"
  - "[[Source---Entra-ID-Audience-Scopes-Deep-Dive]]"
tags:
  - entra-id
  - oauth
  - multi-tenant
  - service-principal
---

# Multi-Tenant and Guest SPN

How [[Microsoft-Entra-ID]] handles applications and users that span multiple tenants. Central mechanism: a [[Service-Principal]] (SPN) is created in every tenant where the app operates — each independently governed.

## Single-Tenant vs Multi-Tenant Apps

Configured in the [[App-Registration]] under **Supported account types**:

| Setting | Who can sign in | SPN behavior |
|---|---|---|
| **Single-tenant** | Accounts in this directory only | SPN in home tenant only |
| **Multi-tenant (AAD)** | Any Azure AD tenant | Guest SPN created in each external tenant on first consent |
| **Multi-tenant + MSA** | AAD + Microsoft personal accounts | Same as above, plus MSA support |

## Home vs Guest SPN

```
Developer's Home Tenant                 External Tenant
-----------------------                 ---------------
App Registration (appId: abc)
         │
         └── Home SPN (Object ID: 111)   Guest SPN (Object ID: 999, appId: abc)
              ← auto-created on           ← created when a user consents
                App Registration            or admin pre-consents
```

- **Home SPN**: created automatically when the App Registration is created. No consent needed.
- **Guest SPN**: created in each external tenant when a user or admin consents. Same `appId` as the home SPN, but a **different Object ID** and **independent granted permissions**.

## Cross-Tenant Consent Flow

For a user in an external tenant to sign in to a multi-tenant app:

1. User from external tenant visits the app.
2. Entra ID detects no guest SPN exists in their tenant yet.
3. User (or admin) is presented with a **consent prompt** listing the permissions the app requests.
4. On consent:
   - A guest SPN is created in the external tenant.
   - Delegated permissions are recorded against that SPN.
5. Subsequent sign-ins from that tenant skip the consent prompt (SPN already exists).

**Admin pre-consent**: Tenant admins can consent on behalf of all users in their tenant, creating the guest SPN without individual users needing to consent. Required for application permissions (no user is present to consent).

## Independent Governance Per Tenant

Each tenant's admin independently controls:

| Control | Per-tenant |
|---|---|
| **Granted permissions** | Admin can grant or revoke permissions on the guest SPN |
| **Assignment required** | Whether users must be explicitly assigned to use the app |
| **[[Conditional-Access]]** | CA policies apply based on the tenant the user signs in to |
| **App Roles** | App Role assignments are per-tenant on the guest SPN |

A permission granted in the home tenant does **not** flow to external tenants. Each guest SPN starts with no permissions until consent is given.

## Conditional Access Across Tenant Boundaries

When a user from Tenant B signs in to an app registered in Tenant A:

- **Tenant B's CA policies** evaluate the sign-in (home tenant of the user).
- **Tenant A can also configure Cross-Tenant Access Settings** to enforce additional requirements on inbound users.
- Both sets of policies must be satisfied for the sign-in to succeed.

This means a user who passes their home tenant's MFA requirement may still be blocked if the resource tenant has stricter policies (e.g., requires compliant device, specific location).

## Token Audience in Multi-Tenant Apps

The `iss` (issuer) claim in the JWT changes per tenant:

```
Single-tenant: iss = https://login.microsoftonline.com/{home-tenant-id}/v2.0
Multi-tenant:  iss = https://login.microsoftonline.com/{signing-tenant-id}/v2.0
```

Multi-tenant backends must validate `iss` against a **dynamic list** (all known tenant IDs) or use the wildcard issuer `https://login.microsoftonline.com/{tenantid}/v2.0` and validate `tid` claim separately. Accepting any issuer is a security vulnerability.

## Common Pitfalls

- **Assuming home-tenant permissions apply everywhere**: they don't — each guest SPN needs its own consent.
- **Not validating `iss` in multi-tenant backends**: allows tokens from any tenant, potentially enabling cross-tenant token substitution.
- **App Roles not assigned in external tenant**: the `roles` claim will be empty even if the role is defined in the home App Registration.
- **Forgetting Cross-Tenant Access Settings**: Tenant A can block external users regardless of Tenant B's CA policies.

## See Also

- [[Service-Principal]] — the per-tenant runtime instance; home vs guest SPN
- [[App-Registration]] — the home-tenant blueprint; multi-tenant setting lives here
- [[Consent]] — how guest SPNs are created and permissions are granted
- [[Conditional-Access]] — per-tenant CA policies that apply across tenants
- [[JWT]] — `iss` and `tid` claims for multi-tenant validation
- [[Microsoft-Entra-ID]] — the authorization server managing tenant boundaries
- [[Delegated-vs-Application-Permissions]] — consent rules differ for delegated vs application permissions
