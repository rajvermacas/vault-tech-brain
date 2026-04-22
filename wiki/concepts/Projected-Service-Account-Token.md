---
title: "Projected Service Account Token"
type: concept
created: 2026-04-22
updated: 2026-04-22
sources:
  - "[[Source---AKS-Workload-Identity-Federated-Token]]"
tags:
  - kubernetes
  - token
  - workload-identity
---

# Projected Service Account Token

A **Projected Service Account Token** is a short-lived [[JWT]] that Kubernetes issues for a pod's service account and projects (writes) to a file inside the container. It is distinct from the older long-lived static service account tokens (which were stored as Secrets and are now deprecated in favor of this mechanism).

## How It Gets Into a Pod

In the [[Azure-Workload-Identity]] flow:

1. The [[Mutating-Admission-Webhook]] adds a `projected` volume definition to the pod spec, referencing the service account and specifying a desired audience (e.g., `api://AzureADTokenExchange`).
2. The kubelet, when starting the pod, calls the Kubernetes `TokenRequest` API to obtain a signed JWT for the service account with that audience and a defined expiry.
3. The kubelet writes the token to the file path specified in the volume mount (typically `/var/run/secrets/azure/tokens/azure-identity-token`).
4. The env var `AZURE_FEDERATED_TOKEN_FILE` (injected by the webhook) points to this path.
5. The kubelet **automatically refreshes** the token when it reaches ~80% of its lifetime — no pod restart needed. Tokens are typically valid for 1 hour, so refresh happens every ~48 minutes.

## Token Properties

| Property | Value |
|---|---|
| Format | [[JWT]] (signed RS256 by the cluster's OIDC issuer) |
| Issuer (`iss`) | AKS cluster OIDC issuer URL |
| Subject (`sub`) | `system:serviceaccount:<namespace>:<sa-name>` |
| Audience (`aud`) | `api://AzureADTokenExchange` (for Azure Workload Identity) |
| Expiry | ~1 hour |
| Refresh | ~48 minutes (kubelet auto-refreshes) |

## Distinction from Legacy Service Account Tokens

| | Projected Token | Legacy Secret-based Token |
|---|---|---|
| Lifetime | Short (~1 hour) | Infinite (until deleted) |
| Audience | Bound to specific audience | Generic |
| Storage | File in pod via projected volume | Kubernetes Secret |
| Refresh | Auto by kubelet | Manual rotation |
| Kubernetes version | 1.20+ | Pre-1.24 default |

## Connections

- [[Azure-Workload-Identity]] — the AKS feature that uses projected tokens as identity proof
- [[Mutating-Admission-Webhook]] — injects the projected volume definition into the pod spec
- [[Federated-Credentials]] — the Azure AD configuration that trusts and validates these tokens
- [[AKS]] — the cluster platform whose kubelet writes and refreshes the token
- [[JWT]] — the format of the projected token
- [[JWKS]] — the cluster OIDC issuer's public key endpoint used to verify the token signature
