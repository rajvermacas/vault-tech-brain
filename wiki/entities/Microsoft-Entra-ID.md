---
title: "Microsoft Entra ID"
type: entity
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---Entra-ID-OAuth-Reference]]"
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

## Key Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/oauth2/v2.0/authorize` | User login + consent (browser redirect) |
| `/oauth2/v2.0/token` | Token exchange (server-to-server) |
| `/discovery/v2.0/keys` | [[JWKS]] public keys (public, no auth needed) |

## Design Principle

Your application never handles user credentials. Authentication is fully delegated to Entra ID. Your app only receives and validates tokens.
