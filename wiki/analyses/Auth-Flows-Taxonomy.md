---
title: "Auth Flows Taxonomy"
type: analysis
created: 2026-04-13
updated: 2026-04-13
sources:
  - "[[Source---Auth-Flows-Delegated-OID-Sub-Session]]"
  - "[[Source---Entra-ID-OAuth-Reference]]"
  - "[[Source---Microsoft-Learn-Permissions-and-Consent-Overview]]"
tags:
  - oauth
  - entra-id
  - authorization
  - analysis
---

# Auth Flows Taxonomy

A corrected taxonomy of Microsoft Entra ID authentication flows, organized by two independent dimensions: who authenticates, and how authorization is enforced in the token.

## The Two Dimensions

**Dimension 1 — Who authenticates:**
- A human user (Authorization Code Flow, Device Code, ROPC, On-Behalf-Of)
- A service acting as itself (Client Credentials)

**Dimension 2 — What claim carries authorization:**
- `scp` claim — delegated scope (only possible in user flows)
- `roles` claim — App Role assignment
- Neither — identity alone, all-or-nothing trust

## Corrected Taxonomy Table

| | User-authenticated (Auth Code / Device Code / etc.) | Service-to-service (Client Credentials) |
|---|---|---|
| **Fine-grained via `scp`** | ✅ Delegated scope; user/admin consent required | ❌ `scp` never appears in Client Credentials tokens |
| **Fine-grained via `roles`** | ✅ App Roles assigned to user/group on Enterprise App | ✅ App Roles assigned to calling SPN |
| **Coarse (identity only)** | ✅ Authenticate the user, trust them fully | ✅ Authenticate the SPN, trust it fully |

## Why `scp` Cannot Appear in Client Credentials

The `scp` claim represents a *delegated* permission — an action the client is permitted to perform *on behalf of a user*. When there is no user, there is nothing to delegate from. Client Credentials tokens carry `roles` instead, which represent *application permissions* granted directly to the calling service's SPN.

Confirmed by Microsoft official documentation:
- "scp — Only included for user tokens." (Access Token Claims Reference)
- "The scp claim is absent in application permission tokens." (Application and Delegated Permissions)

## Why `scp` and `roles` Are Not Mutually Exclusive in User Flows

A user-authenticated token can contain both:
- `scp` — the delegated scope the user consented to (e.g., `Scores.Read`)
- `roles` — the App Roles the user has been assigned (e.g., `Reports.Viewer`)

These answer different questions:
- `scp` → what is the client permitted to do on behalf of this user?
- `roles` → what role does this user have within the application?

The backend checks whichever is relevant to the endpoint being called.

## The BFF Pattern in This Taxonomy

The [[BFF-Pattern]] uses [[OAuth-2.0-Authorization-Code-Flow]] — a user flow — so tokens carry `scp` (delegated scope) and user identity claims (`oid`, `sub`). Entra ID is only involved at initial authentication and silent token renewal; all subsequent API calls validate locally.

## Connections

- [[OAuth-2.0-Authorization-Code-Flow]] — primary user-authenticated flow
- [[Scope]] — `scp` claim and delegated permission mechanism
- [[App-Roles]] — `roles` claim and application/user RBAC mechanism
- [[Delegated-vs-Application-Permissions]] — core distinction between the two columns
- [[BFF-Pattern]] — the user-authenticated flow with server-side token storage
- [[JWT]] — where `scp`, `roles`, `oid`, `sub` all live
- [[Service-Principal]] — holds app role assignments for Client Credentials
- [[Consent]] — required for delegated scopes; admin-only for application permissions
