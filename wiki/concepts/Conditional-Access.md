---
title: "Conditional Access"
type: concept
created: 2026-04-12
updated: 2026-04-12
sources:
  - "[[Source---Microsoft-Learn-Conditional-Access-Overview]]"
tags:
  - azure
  - identity
  - zero-trust
  - policy
  - security
---

# Conditional Access

A policy layer in [[Microsoft-Entra-ID]] that evaluates identity and device context **after first-factor authentication** and decides whether access should be blocked or granted with additional controls.

## Mental Model

Microsoft describes Conditional Access as the **Zero Trust policy engine** for Entra ID.

At the simplest level:

```text
IF principal attempts to access a resource
THEN evaluate policy signals
AND either block access or require extra controls
```

This is separate from [[Scope|OAuth scopes]], [[App-Roles]], and consent. Those describe **what an app is allowed to ask for** or **what role a principal has**. Conditional Access decides **under which conditions access is allowed right now**.

## When It Runs

Conditional Access is enforced **after first-factor authentication**. It is not the first line of defense for pre-auth attacks such as denial-of-service, though Microsoft notes it can use related risk signals in its decisions.

## Common Signals

- User, group, or agent identity
- IP location
- Device platform or compliance state
- Target cloud app / resource
- Real-time or calculated risk signals

## Common Decisions

- Block access
- Grant access only if MFA is completed
- Grant access only if an authentication strength requirement is met
- Grant access only from a compliant or hybrid-joined device
- Grant access only through an approved client app or app protection policy
- Grant access only after password change or terms-of-use acceptance

## Targeting Model

Conditional Access policies can include or exclude:

- All users
- Selected users and groups
- Guest or external users
- Directory roles
- Some workload identities, including **single-tenant service principals**

Microsoft documents two important scoping rules:

- **Exclusions override inclusions** within a policy.
- **Managed identities are not covered** by Conditional Access policy.

## Relationship to Service Principals

Conditional Access is relevant to [[Service-Principal|service principals]] because Microsoft allows targeting of **single-tenant service principals** as workload identities. That means Conditional Access is not only for interactive end-user sign-ins.

## Practical Notes

- Microsoft documents a group-membership evaluation limit: if a user or group belongs to more than **2048 groups**, access might be blocked during Conditional Access evaluation.
- Using Conditional Access requires **Microsoft Entra ID P1** licensing or an equivalent qualifying license.

## Connections

- [[Microsoft-Entra-ID]] — where Conditional Access policies are defined and enforced
- [[Service-Principal]] — workload-identity targeting includes some service principals
- [[Scope]] — permission model that Conditional Access does not replace
- [[App-Roles]] — app-level RBAC that Conditional Access does not replace
- [[Source---Microsoft-Learn-Conditional-Access-Overview]] — official-doc source
