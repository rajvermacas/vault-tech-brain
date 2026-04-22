---
title: "Mutating Admission Webhook"
type: concept
created: 2026-04-22
updated: 2026-04-22
sources:
  - "[[Source---AKS-Workload-Identity-Federated-Token]]"
tags:
  - kubernetes
  - admission-control
  - workload-identity
---

# Mutating Admission Webhook

A **Mutating Admission Webhook** is a Kubernetes extension point that intercepts API object creation/update requests and modifies (mutates) them before they are persisted to etcd. It runs during the **admission phase** — after authentication and authorization, but before the object is stored.

## Role in Azure Workload Identity

In the context of [[Azure-Workload-Identity]], AKS installs a webhook (enabled via `--enable-workload-identity`) that watches for pods labeled `azure.workload.identity/use: "true"`. When such a pod is created, the webhook mutates the pod spec by injecting:

1. **Three environment variables:**
   - `AZURE_FEDERATED_TOKEN_FILE` — path to the [[Projected-Service-Account-Token]] file
   - `AZURE_CLIENT_ID` — from the `azure.workload.identity/client-id` annotation on the Kubernetes ServiceAccount
   - `AZURE_TENANT_ID` — from cluster-level configuration

2. **A `projected` volume** — defines where the kubelet will write the short-lived OIDC token.

3. **A `volumeMount`** — mounts the projected volume into the container at the token file path.

## Key Characteristics

- **Timing:** Runs at admission time — before the pod is scheduled or started. The env vars and volume are part of the pod spec from the moment it's accepted by the API server.
- **Stateless:** The webhook itself does not create the token file; it only defines the pod spec mutation. The actual file is written by the kubelet.
- **Source of truth for client ID:** It reads `azure.workload.identity/client-id` from the ServiceAccount annotation to populate `AZURE_CLIENT_ID` — this is the binding between K8s identity and Azure identity.

## General Kubernetes Context

Mutating webhooks are one half of the admission controller pattern (the other being **Validating** webhooks). They are widely used for:
- Injecting sidecars (e.g., Istio, Linkerd)
- Injecting environment variables and secrets (e.g., Vault Agent, Workload Identity)
- Enforcing defaults and policy

## Connections

- [[Azure-Workload-Identity]] — the feature this webhook implements
- [[AKS]] — the platform that installs and manages this webhook
- [[Projected-Service-Account-Token]] — the volume type the webhook instructs the kubelet to create
