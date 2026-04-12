---
title: "Source - Microsoft Learn Conditional Access Overview"
type: source
created: 2026-04-12
updated: 2026-04-12
sources:
  - "raw/microsoft-learn-conditional-access-overview-2026-04-12.md"
tags:
  - azure
  - identity
  - conditional-access
  - microsoft-docs
  - zero-trust
---

# Source - Microsoft Learn Conditional Access Overview

## Summary

Official Microsoft Learn grounding for [[Conditional-Access]] as an Entra policy engine: how policies are structured, when they run, which signals they evaluate, what controls they can enforce, and which identities can be targeted.

## Main Claims

- [[Conditional-Access]] is Microsoft's **Zero Trust policy engine** for identity-driven access decisions.
- Policies are effectively **if-then rules**: if a principal accesses a resource, then one or more controls can be enforced before access is granted.
- Conditional Access evaluates after **first-factor authentication**, not before sign-in starts.
- Policy signals can include users/groups, location, device state, target application, and risk signals.
- Policy outcomes include either **blocking access** or **granting access with extra controls** such as MFA or compliant-device requirements.
- Microsoft states that Entra evaluates all applicable Conditional Access policies and ensures all requirements are satisfied before granting access.
- Policies can target both human principals and some workload identities, but **managed identities are out of scope**.

## Key Takeaways

- Conditional Access is not the same thing as OAuth consent or app-role assignment; it is a separate policy layer that can gate access after primary authentication.
- Service principals matter here because Microsoft allows Conditional Access targeting for **single-tenant service principals** as workload identities.
- Exclusions are safety-critical because Microsoft documents that exclusions override inclusions within a policy.

## Connections

- [[Conditional-Access]] — concept distilled from the official docs
- [[Microsoft-Entra-ID]] — hosts and evaluates Conditional Access policies
- [[Service-Principal]] — some workload identities can be targeted by Conditional Access
- [[App-Roles]] — separate authorization layer from Conditional Access
- [[Scope]] — delegated permission model distinct from Conditional Access
