# Microsoft Learn — Entra ID App Roles Fact Check (2026-04-12)

Captured: 2026-04-12 UTC

This raw note records the official Microsoft documentation consulted to fact-check the wiki's Entra ID App Role content.

Sources checked:

- https://learn.microsoft.com/en-us/entra/identity-platform/howto-add-app-roles-in-apps
- https://learn.microsoft.com/en-us/entra/identity-platform/reference-microsoft-graph-app-manifest
- https://learn.microsoft.com/en-us/entra/identity-platform/scenario-protected-web-api-verification-scope-app-roles
- https://learn.microsoft.com/en-us/entra/identity-platform/scenario-protected-web-api-app-registration?view=aspnetcore-1.0
- https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/assign-user-or-group-access-portal

Facts confirmed from the docs:

- App roles are defined on the application object / app registration and are exposed in the `appRoles` manifest collection.
- The portal UI for app-role creation centers on Display name, Allowed member types, Value, Description, and enabled state. The manifest also includes fields such as `id`, `isEnabled`, and `origin`.
- Assignment of users and groups happens on the Enterprise Application (service principal), not on the app-role definition itself.
- App roles whose `allowedMemberTypes` includes `Application` are exposed to client apps as application permissions.
- Missing app-role claims are typically handled by the protected API or framework at request time, but token issuance can also be blocked earlier when assignment is required on the enterprise application.
- If `Assignment required?` is enabled, Microsoft documents that unassigned client applications can be prevented from obtaining tokens for that API, returning `AADSTS501051`.
- Microsoft notes two important edge cases:
  - Disabled app roles continue to pass in tokens until assignments are removed.
  - If a service principal is added to a group that has an app-role assignment, the `roles` claim is not emitted for that service principal.
