---
title: "Azure Workload Identity"
type: concept
created: 2026-04-22
updated: 2026-04-22
sources:
  - "[[Source---AKS-Workload-Identity-Federated-Token]]"
tags:
  - kubernetes
  - azure
  - identity
  - workload-identity
---

# Azure Workload Identity

**Azure Workload Identity** is the AKS mechanism that allows pods to authenticate to Azure services without storing long-lived credentials (secrets, certificates, or connection strings). It replaces the older, now-deprecated **aad-pod-identity** approach.

## Core Idea

Instead of embedding a client secret in a pod, the cluster issues a short-lived [[Projected-Service-Account-Token]] (a [[JWT]]) for the pod's Kubernetes service account. The Azure SDK exchanges this token with [[Microsoft-Entra-ID]] for an Azure access token via the [[Federated-Credentials]] mechanism. No secret ever lives in the pod.

## How It Works (End-to-End)

1. **Cluster setup:** AKS is enabled with `--enable-oidc-issuer` and `--enable-workload-identity`. The cluster gets a public OIDC issuer URL.
2. **Identity binding:** A [[Federated-Credentials]] record on a managed identity (or App Registration) links `{OIDC issuer URL, Kubernetes namespace, Kubernetes service account name}` to an Azure identity.
3. **Service account annotation:** The Kubernetes ServiceAccount is annotated with `azure.workload.identity/client-id: <managed-identity-client-id>`.
4. **Pod labeling:** The pod is labeled `azure.workload.identity/use: "true"`.
5. **Webhook injection:** At pod creation, the [[Mutating-Admission-Webhook]] injects three env vars and a projected volume into the pod spec.
6. **Token projection:** The kubelet writes the OIDC token to the file path referenced by `AZURE_FEDERATED_TOKEN_FILE` and refreshes it automatically.
7. **SDK token exchange:** At runtime, `DefaultAzureCredential` or `WorkloadIdentityCredential` reads the three env vars, sends the K8s token to Entra ID, and receives an Azure access token.

## The Three Injected Env Vars

| Variable | Purpose |
|---|---|
| `AZURE_FEDERATED_TOKEN_FILE` | Path to the projected K8s service account token file |
| `AZURE_CLIENT_ID` | Client ID of the managed identity to authenticate as |
| `AZURE_TENANT_ID` | Azure AD tenant to authenticate against |

## Advantages Over aad-pod-identity

- No DaemonSet required; webhook is lighter weight.
- Works with upstream Kubernetes mechanisms (projected volumes, OIDC).
- Supported by all Azure SDKs via `DefaultAzureCredential` with zero code changes.
- Token refresh is handled by the kubelet — no pod restart needed.

## Connections

- [[AKS]] — the cluster platform this feature runs on
- [[Mutating-Admission-Webhook]] — injects env vars and projected volume at pod creation
- [[Projected-Service-Account-Token]] — the K8s-issued token that serves as the identity proof
- [[Federated-Credentials]] — Azure AD trust configuration that validates the K8s token
- [[Microsoft-Entra-ID]] — exchanges the K8s token for an Azure access token
- [[Service-Principal]] — the Azure identity the pod authenticates as
- [[JWT]] — format of the projected service account token
