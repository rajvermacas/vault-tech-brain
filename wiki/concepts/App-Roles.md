---
title: "App Roles"
type: concept
created: 2026-04-11
updated: 2026-04-11
sources:
  - "[[Source---Entra-ID-App-Roles-BFF-JWT-Signing]]"
tags:
  - azure
  - identity
  - authorization
  - rbac
---

# App Roles

A mechanism for **role-based access control (RBAC)** inside your own application, defined on an [[App-Registration]] and enforced by the backend. Distinct from [[Scope|OAuth scopes]].

## App Roles vs Scopes

| | App Roles | Scopes |
|---|---|---|
| JWT claim | `roles` | `scp` |
| Answers | Who the user IS | What the app can DO |
| Model | RBAC | Delegated permissions |
| Granted by | Admin assigns users/groups | User consents on consent screen |

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
5. User NOT in any assigned group → roles claim ABSENT from JWT (not empty array)
6. Backend receives JWT, checks roles claim
7. Missing or wrong role → 403 (thrown by backend, not Entra ID)
8. Correct role present → request proceeds
```

> [!warning]
> Entra ID **never throws 403**. It always issues the JWT. The `roles` claim is either present (with the assigned values) or absent entirely. The 403 decision belongs entirely to the backend.

## Absence vs Empty Array

When a user has no role assignments, the `roles` claim is **absent from the JWT** — not an empty array. Backend code must handle this:

```javascript
// Wrong — crashes if roles claim is absent
if (token.roles.includes('Reports.Viewer')) { ... }

// Right — safely handles absent claim
if (token.roles?.includes('Reports.Viewer')) { ... }
```

## App Roles for Applications (not just users)

When `Allowed member types` includes `Applications`, App Roles appear as **application permissions** in the API Permissions tab of another App Registration. This is how daemon apps get role-based access to an API without a user.

## Connections

- [[App-Registration]] — where App Roles are defined
- [[Service-Principal]] — where App Roles are assigned to users and groups
- [[Scope]] — the delegated-permission counterpart (`scp` claim vs `roles` claim)
- [[JWT]] — carries the `roles` claim
- [[Microsoft-Entra-ID]] — issues the JWT with roles populated from assignments
- [[Source---Entra-ID-App-Roles-BFF-JWT-Signing]] — source session
