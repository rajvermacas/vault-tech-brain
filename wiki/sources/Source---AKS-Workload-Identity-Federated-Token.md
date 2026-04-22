---
title: "Source - AKS Workload Identity Federated Token"
type: source
created: 2026-04-22
updated: 2026-04-22
sources:
  - "raw/aks-workload-identity-federated-token-2026-04-22.md"
tags:
  - kubernetes
  - azure
  - identity
  - workload-identity
---

# Source - AKS Workload Identity Federated Token

**Origin:** Cowork conversation, 2026-04-22  
**Topic:** What `AZURE_FEDERATED_TOKEN_FILE` is in AKS, and the two-component lifecycle of its creation and refresh.

---

## Main Claims

1. `AZURE_FEDERATED_TOKEN_FILE` is an environment variable automatically injected into AKS pods by the [[Azure-Workload-Identity]] mutating admission webhook when a pod is labeled `azure.workload.identity/use: "true"`.
2. The env var points to `/var/run/secrets/azure/tokens/azure-identity-token` — a [[Projected-Service-Account-Token]] written to disk by the kubelet.
3. Two separate actors own two separate responsibilities: the [[Mutating-Admission-Webhook]] injects the env var and volume definition (static, once at admission time); the kubelet writes and refreshes the actual token file (dynamic, every ~48 minutes).
4. The token file holds a Kubernetes-issued [[JWT]] signed by the cluster's OIDC issuer. The Azure SDK exchanges this token with [[Microsoft-Entra-ID]] for a short-lived Azure access token via [[Federated-Credentials]].
5. Three env vars are always injected together: `AZURE_FEDERATED_TOKEN_FILE`, `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`.
6. `DefaultAzureCredential` and `WorkloadIdentityCredential` in all Azure SDKs natively read these env vars with no code changes needed.

## Key Evidence

- The webhook reads `azure.workload.identity/client-id` from the Kubernetes ServiceAccount to determine `AZURE_CLIENT_ID` to inject.
- Token lifetime is ~1 hour; kubelet refreshes at ~80% of expiry (every ~48 min) — pods never need to restart for fresh tokens.
- Azure AD trusts the Kubernetes-issued token because a [[Federated-Credentials]] record was pre-configured linking the cluster's OIDC issuer URL + namespace + service account name to a managed identity.
- Prerequisite cluster flags: `--enable-oidc-issuer`, `--enable-workload-identity`.

## Methodology

Q&A conversation exploring the mechanism from first principles. No primary Microsoft docs cited; based on Claude's training knowledge. Claims should be verified against Microsoft Learn for production use.

## Limitations

- Specific file paths and token refresh intervals may vary across AKS versions.
- aad-pod-identity (deprecated predecessor) is mentioned but not deeply covered.
- No coverage of the `TokenRequest` API mechanics or how the OIDC issuer signs tokens.

## Relevance to Existing Wiki Topics

- Extends the [[Microsoft-Entra-ID]] identity surface into Kubernetes workloads.
- [[Federated-Credentials]] is a new concept closely related to [[Service-Principal]] and [[App-Registration]] — it's the Azure AD mechanism that trusts external OIDC issuers.
- [[JWT]] is already a first-class concept; this source adds a Kubernetes-issued JWT as a new token type in the ecosystem.
- The [[Mutating-Admission-Webhook]] pattern is new infrastructure knowledge not previously covered.

---

## Key Takeaways

- `AZURE_FEDERATED_TOKEN_FILE` is the glue between Kubernetes pod identity and Azure AD — it holds a K8s-issued JWT the SDK uses to prove pod identity to Azure.
- The env var (static) and the file (dynamic) have different owners: webhook vs. kubelet respectively.
- No secrets are stored anywhere — the entire chain is built on short-lived, auto-refreshed tokens.
- `DefaultAzureCredential` makes the SDK side transparent; the heavy lifting is all in cluster setup.
- Federated Credentials on the managed identity is the trust anchor — it binds {OIDC issuer, namespace, service account name} to an Azure identity.

---

## Connections

- [[Azure-Workload-Identity]] — the AKS feature this env var is part of
- [[Mutating-Admission-Webhook]] — creates the env var and projected volume definition at admission time
- [[Projected-Service-Account-Token]] — the actual token file the env var points to, managed by kubelet
- [[Federated-Credentials]] — Azure AD trust record that validates the K8s-issued token
- [[AKS]] — the cluster platform
- [[Microsoft-Entra-ID]] — the identity provider that exchanges the K8s token for an Azure access token
- [[JWT]] — format of both the K8s-issued token and the returned Azure access token
- [[Service-Principal]] — the Azure identity ultimately authenticated via this chain
