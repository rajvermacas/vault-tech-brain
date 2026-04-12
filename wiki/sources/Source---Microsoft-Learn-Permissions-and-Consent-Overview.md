---
title: "Source - Microsoft Learn Permissions and Consent Overview"
type: source
created: 2026-04-12
updated: 2026-04-12
sources:
  - "raw/microsoft-learn-permissions-consent-overview-2026-04-12.md"
tags:
  - azure
  - identity
  - consent
  - microsoft-docs
  - oauth
---

# Source - Microsoft Learn Permissions and Consent Overview

## Summary

Official Microsoft Learn grounding for [[Consent]] in the Microsoft identity platform: what consent means, who can grant it, when prompts appear, how delegated and app-only permissions differ, and when admin approval is required.

## Main Claims

- [[Consent]] is the authorization step where a user or admin allows an app to access a protected resource.
- Delegated access uses [[Scope|scopes]]; app-only access uses [[App-Roles|application permissions / app roles]].
- Users can sometimes grant delegated permissions for their own data, but **application permissions require administrator consent**.
- Consent can be **static** or **dynamic**, depending on whether permissions are preconfigured or requested incrementally at runtime.
- Consent prompts can reappear when permissions change, prior grants are revoked, or the app uses dynamic consent.
- If tenant policy blocks user consent or the requested permission is high privilege, non-admin users are redirected to seek admin approval.
- Microsoft documents an SPA-specific nuance: SPAs using [[MSAL]] currently require explicit consent via the **Grant permissions** button.
- Preauthorization is a separate optimization where a resource app owner suppresses user prompts for trusted client apps.

## Key Takeaways

- Consent is not the same as [[Conditional-Access]] or [[App-Roles]]; it is the permission-grant step that determines what the client app may request.
- The admin-vs-user distinction depends on permission type and tenant policy, not just on whether the app is interactive.
- Consent records are part of the requested-to-granted transition already described across [[Scope]] and [[Service-Principal]].

## Connections

- [[Consent]] — concept distilled from the official docs
- [[Scope]] — delegated permissions that often rely on user or admin consent
- [[App-Roles]] — application permissions that require admin consent in app-only scenarios
- [[MSAL]] — client library that triggers consent prompts in interactive flows
- [[Microsoft-Entra-ID]] — stores and enforces consent records
- [[Conditional-Access]] — separate policy layer from consent
