# Microsoft Learn — Conditional Access Overview (2026-04-12)

Captured: 2026-04-12 UTC

Primary Microsoft Learn pages consulted:

- https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview
- https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-users-groups

Key facts extracted:

- Microsoft describes Conditional Access as its **Zero Trust policy engine**.
- Conditional Access policies are effectively **if-then statements**: if a principal attempts to access a resource, then one or more controls can be enforced.
- Conditional Access is enforced **after first-factor authentication**. It is not the frontline control for events like DoS, though it can use related risk signals.
- Common policy signals include:
  - user, group, or agent assignment
  - IP location
  - device state/platform
  - target application/resource
  - real-time or calculated risk signals
- Common policy outcomes include:
  - block access
  - grant access with controls such as MFA, authentication strength, compliant device, hybrid joined device, approved client app, app protection policy, password change, or terms of use
- Microsoft states that Entra ID evaluates all applicable Conditional Access policies and ensures all requirements are met before granting access.
- Policies can target:
  - all users
  - selected users and groups
  - guest or external users
  - directory roles
  - workload identities (single-tenant service principals)
- Managed identities are not covered by Conditional Access policy.
- Exclusions override inclusions inside a Conditional Access policy.
- Microsoft notes a practical limit: if users or groups belong to more than 2048 groups, their access might be blocked for Conditional Access evaluation.
- Licensing note: Conditional Access requires Microsoft Entra ID P1 or equivalent qualifying licensing.
