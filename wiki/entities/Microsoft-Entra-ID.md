---
title: "Microsoft Entra ID"
type: entity
created: 2026-04-10
updated: 2026-04-22
sources:
  - "[[Source---Entra-ID-OAuth-Reference]]"
  - "[[Source---Microsoft-Learn-Conditional-Access-Overview]]"
  - "[[Source---Microsoft-Learn-Permissions-and-Consent-Overview]]"
  - "[[Source---AKS-Workload-Identity-Federated-Token]]"
tags:
  - azure
  - identity-provider
  - authentication
---

# Microsoft Entra ID

Microsoft's cloud identity and access management service, formerly known as **Azure Active Directory (Azure AD)**. Serves as the **Authorization Server** in [[OAuth-2.0-Authorization-Code-Flow]] flows.

## Responsibilities

- Verifies user identity (authentication)
- Issues cryptographically signed [[JWT]] tokens as proof
- Maintains [[App-Registration]] records, permissions, and consent records
- Publishes public keys via [[JWKS]] so any server can verify tokens
- Hosts [[Service-Principal]] objects per tenant
- Evaluates [[Conditional-Access]] policies during sign-in and resource access decisions

## Key Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/oauth2/v2.0/authorize` | User login + consent (browser redirect) |
| `/oauth2/v2.0/token` | Token exchange (server-to-server) |
| `/discovery/v2.0/keys` | [[JWKS]] public keys (public, no auth needed) |

## Design Principle

Your application never handles user credentials. Authentication is fully delegated to Entra ID. Your app only receives and validates tokens.

## Related Control Layers

- [[Scope]] and consent determine what delegated access a client app can request.
- [[Consent]] records whether delegated or application permissions were approved.
- [[App-Roles]] define RBAC values that can appear in tokens.
- [[Conditional-Access]] evaluates sign-in context and can require extra controls or block access entirely.

## Workload Identity (Pod-to-Azure Auth)

Entra ID also serves as the token exchange endpoint for [[Azure-Workload-Identity]] in [[AKS]]. In this flow, a pod presents a Kubernetes-issued [[Projected-Service-Account-Token]] to Entra ID's token endpoint. Entra ID validates it against [[Federated-Credentials]] (which trust the cluster's OIDC issuer), then issues a short-lived Azure access token. No user is involved; this is always application-permission auth (see [[Delegated-vs-Application-Permissions]]).
