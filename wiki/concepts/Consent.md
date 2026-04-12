---
title: "Consent"
type: concept
created: 2026-04-12
updated: 2026-04-12
sources:
  - "[[Source---Microsoft-Learn-Permissions-and-Consent-Overview]]"
tags:
  - azure
  - identity
  - oauth
  - authorization
---

# Consent

The step in [[Microsoft-Entra-ID]] where a user or administrator authorizes a client application to access a protected resource.

## Mental Model

Consent answers a different question than authentication:

- Authentication: "Who is this principal?"
- Consent: "What is this client app allowed to access?"

It also differs from [[Conditional-Access]] and [[App-Roles]]:

- [[Conditional-Access]] decides whether access is allowed under current sign-in conditions.
- [[App-Roles]] define role assignments and application permissions.
- Consent records whether the requested delegated permission or application permission has been approved.

## Two Main Consent Modes

- **User consent** — a user approves delegated permissions for access to their own data, when tenant policy allows it.
- **Admin consent** — an administrator approves permissions on behalf of the tenant, or approves permissions that users are not allowed to grant.

Microsoft documents that **application permissions** in app-only scenarios require admin consent.

## Static vs Dynamic Consent

- **Static consent** — permissions are listed on the [[App-Registration]] and granted as a predefined set.
- **Dynamic consent** — an app requests additional delegated permissions at runtime as needed.

Dynamic consent applies to delegated permissions, not app-only application permissions.

## When Prompts Appear

Consent prompts typically appear when:

- The required permission has never been granted before
- Prior consent was revoked
- The app explicitly asks again during sign-in
- The app requests new delegated permissions dynamically

If tenant policy blocks user consent or the requested permission requires admin approval, the user is redirected to seek administrator approval instead of self-consenting.

## Practical Notes

- Microsoft documents that SPAs using [[MSAL]] currently require explicit consent via the **Grant permissions** button in the portal.
- Preauthorization is a related but different feature: a resource app owner can preapprove a trusted client app so end users do not see the same consent prompt.

## Connections

- [[Scope]] — delegated permissions commonly granted through consent
- [[App-Roles]] — application permissions / app roles that require admin consent in app-only scenarios
- [[App-Registration]] — where requested permissions are declared
- [[Service-Principal]] — where granted permissions are reflected in tenant context
- [[Microsoft-Entra-ID]] — stores consent records and drives consent prompts
- [[Conditional-Access]] — separate policy layer from consent
- [[Source---Microsoft-Learn-Permissions-and-Consent-Overview]] — official-doc source
