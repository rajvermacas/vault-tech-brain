# Microsoft Learn — Permissions and Consent Overview (2026-04-12)

Captured: 2026-04-12 UTC

Primary Microsoft Learn pages consulted:

- https://learn.microsoft.com/en-us/entra/identity-platform/permissions-consent-overview
- https://learn.microsoft.com/en-us/entra/identity-platform/application-consent-experience

Key facts extracted:

- Consent is the process by which a user or admin authorizes an application to access a protected resource.
- Microsoft separates two access models:
  - delegated access uses scopes and acts on behalf of a signed-in user
  - app-only access uses application permissions / app roles and acts without a signed-in user
- For delegated permissions:
  - users can consent for their own data when tenant policy allows it
  - admins can consent for all users
- For application permissions:
  - only an administrator can consent
- Microsoft distinguishes:
  - static consent: permissions configured on the app registration
  - dynamic consent: additional delegated permissions requested at runtime
- Consent prompts can appear when:
  - no previous consent exists
  - prior consent was revoked
  - the app explicitly prompts again
  - the app uses dynamic consent
- If the tenant requires admin approval, non-admin users are blocked and directed to ask an administrator for access.
- Admins can grant consent through the consent prompt or through the API Permissions page in the Entra admin center.
- Microsoft notes an SPA-specific nuance: explicit consent using the **Grant permissions** button is currently required for SPAs that use MSAL.js.
- Preauthorization lets a resource app owner suppress user consent prompts for a trusted client app and permission set.
