---
title: "AKS"
type: entity
created: 2026-04-22
updated: 2026-04-22
sources:
  - "[[Source---AKS-Workload-Identity-Federated-Token]]"
tags:
  - kubernetes
  - azure
  - platform
---

# AKS

**Azure Kubernetes Service** — Microsoft's managed Kubernetes offering. AKS handles the control plane (API server, etcd, scheduler) as a managed service; customers manage node pools (worker VMs).

## Relevant to This Wiki

AKS is the runtime platform for workloads that need to authenticate to Azure services. The primary mechanism for this is [[Azure-Workload-Identity]], which uses a cluster-level OIDC issuer to issue short-lived [[JWT]] tokens for pod service accounts. These tokens are exchanged with [[Microsoft-Entra-ID]] for Azure access tokens via [[Federated-Credentials]].

## Key AKS Flags for Workload Identity

| Flag | Purpose |
|---|---|
| `--enable-oidc-issuer` | Enables the cluster OIDC issuer URL used by Federated Credentials |
| `--enable-workload-identity` | Installs the [[Mutating-Admission-Webhook]] that injects identity env vars into pods |

## Connections

- [[Azure-Workload-Identity]] — AKS feature for pod-to-Azure authentication without secrets
- [[Mutating-Admission-Webhook]] — webhook installed by AKS to inject workload identity env vars
- [[Projected-Service-Account-Token]] — K8s mechanism AKS uses to deliver short-lived tokens to pods
- [[Federated-Credentials]] — Azure AD trust record that validates tokens issued by the AKS OIDC issuer
- [[Microsoft-Entra-ID]] — identity provider that AKS workloads authenticate against
- [[Service-Principal]] — the Azure identity pods ultimately assume
