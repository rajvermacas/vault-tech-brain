---
title: "App Roles"
type: concept
created: 2026-04-11
updated: 2026-04-12
sources:
  - "[[Source---Entra-ID-App-Roles-BFF-JWT-Signing]]"
  - "[[Source---Microsoft-Learn-Entra-ID-App-Roles-Fact-Check]]"
tags:
  - azure
  - identity
  - authorization
  - rbac
---

# App Roles

A mechanism for **role-based access control (RBAC)** inside your own application, defined on an [[App-Registration]]. App-role assignments affect token contents, and enforcement can happen either during token issuance or in the protected app/API depending on enterprise-app settings. Distinct from [[Scope|OAuth scopes]].

## App Roles vs Scopes

| | App Roles | Scopes |
|---|---|---|
| JWT claim | `roles` | `scp` |
| Answers | What role the calling principal has | What the app can DO |
| Model | RBAC | Delegated permissions |
| Granted by | Admin assigns users, groups, or applications | User or admin consent |

## Definition Fields (on the App Registration)

Confirmed via Microsoft official docs. There is **no `group_assignment` property** on the App Role definition itself.

| Field | Description | Example |
|---|---|---|
| Display name | Human-readable label | "Report Viewer" |
| Allowed member types | Type constraint only — Users/Groups, Applications, or Both | Users/Groups |
| Value | String that appears in `roles` JWT claim | `Reports.Viewer` |
| Description | Shown during admin assignment in portal | "Can view reports" |
| State | Enabled / Disabled toggle | Enabled |

`Allowed member types` is a **type constraint**, not a reference to any specific group. It declares what *category* of things can be assigned — not which specific groups.

In the manifest, app roles also carry generated metadata such as `id`, `isEnabled`, and `origin`.

## Two-Layer Model: Definition vs Assignment

```
Layer 1 — Definition (App Registration)
  App Roles tab → create role
  Value: "Reports.Viewer"
  → Says: "this role EXISTS"
  → No reference to any specific user or group here

Layer 2 — Assignment (Enterprise Application / Service Principal)
  Enterprise Applications → [Your App] → Users and Groups
  → Add Assignment
  → Select: Security Group X
  → Select role: Reports.Viewer
  → Says: "Security Group X HAS this role"
```

The [[App-Registration]] is the definition layer. The [[Service-Principal]] (Enterprise Application) is the assignment layer. These are **separate places** in the Azure portal.

## Runtime Flow

```
1. User logs in via OAuth flow
2. Entra ID checks assignments on Enterprise Application (Service Principal)
3. User is in Security Group X → Group X is assigned Reports.Viewer
4. JWT issued with: roles: ["Reports.Viewer"]
5. User NOT in any assigned group → token may be issued without a `roles` claim
6. If the Enterprise Application has `Assignment required? = Yes`, Entra ID can deny token issuance to unassigned principals instead
7. If a token reaches the backend, the backend/framework checks `roles`
8. Missing or wrong role → app/framework returns an authorization failure (commonly 401 or 403)
9. Correct role present → request proceeds
```

> [!warning]
> Fact-check correction (2026-04-12): the earlier wording here overstated token issuance behavior. Microsoft documents that Entra ID can deny token issuance when the Enterprise Application is configured with **Assignment required? = Yes**. Once a token is issued, missing-role handling is then up to the protected app or framework.

## Absence vs Empty Array

When no app-role assignment is present, the `roles` claim is typically **omitted** rather than emitted as an empty array. Backend code should treat the claim as optional:

```javascript
// Wrong — crashes if roles claim is absent
if (token.roles.includes('Reports.Viewer')) { ... }

// Right — safely handles absent claim
if (token.roles?.includes('Reports.Viewer')) { ... }
```

## App Roles for Applications (not just users)

When `Allowed member types` includes `Applications`, App Roles appear as **application permissions** in the API Permissions tab of another App Registration. This is how daemon apps get role-based access to an API without a user.

## Important Edge Cases

- If a role is disabled, Microsoft notes it can **continue to appear in tokens for already assigned users, groups, or applications** until those assignments are removed.
- If a **service principal** is placed in a group that has an app-role assignment, Microsoft notes the `roles` claim is **not emitted** for that service principal via group inheritance.

## Connections

- [[App-Registration]] — where App Roles are defined
- [[Service-Principal]] — where App Roles are assigned to users and groups
- [[Scope]] — the delegated-permission counterpart (`scp` claim vs `roles` claim)
- [[JWT]] — carries the `roles` claim
- [[Microsoft-Entra-ID]] — issues the JWT with roles populated from assignments
- [[Source---Microsoft-Learn-Entra-ID-App-Roles-Fact-Check]] — official-doc fact check for issuance and assignment edge cases
- [[Source---Entra-ID-App-Roles-BFF-JWT-Signing]] — source session
- [[Auth-Flows-Taxonomy]] — shows where roles claim appears vs scp in the full auth flow matrix
