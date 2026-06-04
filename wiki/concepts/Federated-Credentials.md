---
title: "Federated Credentials"
type: concept
created: 2026-04-22
updated: 2026-04-22
sources:
  - "[[Source---AKS-Workload-Identity-Federated-Token]]"
tags:
  - azure
  - identity
  - oidc
  - workload-identity
---

# Federated Credentials

**Federated Credentials** (also called *workload identity federation* in Azure) is an [[Microsoft-Entra-ID]] configuration that allows an external OIDC identity provider to be trusted for authenticating Azure identities — without requiring a client secret or certificate.

## Core Idea

Instead of a pod holding a secret, Azure AD is pre-configured to trust tokens issued by a specific external OIDC provider (e.g., an [[AKS]] cluster). When a pod presents such a token, Azure AD validates it against the trusted OIDC issuer's public keys and issues an Azure access token in exchange.

## Configuration

A Federated Credential record is attached to a managed identity or [[App-Registration]] and specifies three things:

| Field | Description |
|---|---|
| **Issuer URL** | The AKS cluster's OIDC issuer URL (e.g., `https://oidc.prod-aks.azure.com/<cluster-id>/`) |
| **Subject** | The Kubernetes identity: `system:serviceaccount:<namespace>:<service-account-name>` |
| **Audience** | Typically `api://AzureADTokenExchange` |

Azure AD will only issue tokens when all three fields match the presented [[JWT]].

## Token Exchange Flow

```
Pod (kubelet-written token at AZURE_FEDERATED_TOKEN_FILE)
       │
       ▼
Azure SDK sends K8s JWT to Entra ID token endpoint
       │   POST /token
       │   grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
       │   client_assertion=<K8s JWT>
       ▼
Entra ID validates:
  1. JWT signature against AKS OIDC issuer's JWKS
  2. iss matches registered Issuer URL
  3. sub matches registered Subject
  4. aud matches registered Audience
       │
       ▼
Entra ID issues short-lived Azure access token for the managed identity
```

## Relation to Other Auth Concepts

- Federated Credentials implement the **OIDC federation** pattern — analogous to how Entra ID itself acts as an OIDC provider for other apps.
- It is distinct from [[Delegated-vs-Application-Permissions]]: pod-to-Azure auth is always application-level (no user present); the token carries managed identity claims, not user claims.
- The [[JWKS]] endpoint of the AKS OIDC issuer is the cryptographic trust anchor — Entra ID fetches it to verify the K8s JWT signature.

## Connections

- [[Azure-Workload-Identity]] — the AKS feature that uses Federated Credentials as its trust anchor
- [[AKS]] — the cluster whose OIDC issuer is registered as the trusted external provider
- [[Microsoft-Entra-ID]] — the identity provider that holds the Federated Credential record and performs token exchange
- [[Projected-Service-Account-Token]] — the K8s-issued token presented to Entra ID
- [[App-Registration]] — where Federated Credentials are configured (along with managed identities)
- [[Service-Principal]] — the Azure identity ultimately granted an access token
- [[JWT]] — format of both the K8s-issued assertion and the returned Azure token
- [[JWKS]] — AKS OIDC issuer's public key endpoint, used by Entra ID to verify the K8s JWT
- [[Delegated-vs-Application-Permissions]] — pod-to-Azure auth is always application-permission (no user present)
