---
title: "Source - Microsoft Learn Entra ID App Roles Fact Check"
type: source
created: 2026-04-12
updated: 2026-04-12
sources:
  - "raw/microsoft-learn-entra-id-app-roles-fact-check-2026-04-12.md"
tags:
  - azure
  - identity
  - app-roles
  - microsoft-docs
  - fact-check
---

# Source - Microsoft Learn Entra ID App Roles Fact Check

## Summary

Cross-check of the wiki's [[App-Roles]] material against current Microsoft Learn documentation for Entra ID app-role definition, assignment, token validation, and assignment-required behavior.

## Main Claims

- App roles are defined on the [[App-Registration]] / application object and represented in the manifest's `appRoles` collection.
- User and group assignment happens on the [[Service-Principal]] (Enterprise Application), not on the app-role definition itself.
- `Allowed member types` is a category constraint (`User`, `Application`, or both), not a reference to a specific group.
- When app roles allow `Application`, they surface to client applications as application permissions.
- Missing role claims are often enforced by the protected API or framework, but token issuance can also be denied earlier when the enterprise application's **Assignment required?** setting is enabled.
- Microsoft documents two edge cases worth preserving in the wiki: disabled app roles can still be emitted for existing assignments, and service principals do not inherit app-role claims through group membership.

## Key Takeaways

- The wiki's definition-vs-assignment model was correct.
- The earlier absolute claim that Entra ID "always issues the JWT" was too strong and needed correction.
- Role enforcement should be described as a mix of Entra issuance behavior and API/framework authorization behavior, depending on tenant configuration.

## Connections

- [[App-Roles]] — corrected with assignment-required and claim-emission nuances
- [[App-Registration]] — source of the `appRoles` definition
- [[Service-Principal]] — assignment layer for users and groups
- [[Microsoft-Entra-ID]] — token issuer and policy enforcement point
- [[Source---Entra-ID-App-Roles-BFF-JWT-Signing]] — original source page that required the fact-check correction
