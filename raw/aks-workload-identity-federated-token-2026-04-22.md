# AKS Workload Identity — AZURE_FEDERATED_TOKEN_FILE

**Source type:** Cowork conversation  
**Date:** 2026-04-22  
**Topic:** What AZURE_FEDERATED_TOKEN_FILE is in AKS, and who creates it and when

---

## Q1: What is AZURE_FEDERATED_TOKEN_FILE in AKS?

`AZURE_FEDERATED_TOKEN_FILE` is an environment variable automatically injected into pods by AKS when you use **Workload Identity**. It points to a file path (typically `/var/run/secrets/azure/tokens/azure-identity-token`) containing a Kubernetes-issued **OIDC token** for the pod's service account.

### The problem it solves

Pods need to authenticate to Azure services (Key Vault, Storage, etc.) without storing long-lived credentials. Workload Identity replaces the older Pod Identity (aad-pod-identity) approach.

### How it works

1. AKS has an OIDC issuer URL. Kubernetes issues short-lived JWT tokens for service accounts and projects them into pods via that file.
2. The Azure SDK reads `AZURE_FEDERATED_TOKEN_FILE` (along with `AZURE_CLIENT_ID` and `AZURE_TENANT_ID`, also auto-injected) to perform a **federated credential exchange** with Azure AD.
3. Azure AD trusts the token because a federated credential was pre-configured on the managed identity/app registration, tying it to the specific Kubernetes service account + namespace + cluster OIDC issuer.
4. Azure AD issues a short-lived Azure access token in return, granting access to Azure resources.

### The three env vars injected together

| Variable | Value |
|---|---|
| `AZURE_FEDERATED_TOKEN_FILE` | Path to the projected service account token |
| `AZURE_CLIENT_ID` | Client ID of the managed identity |
| `AZURE_TENANT_ID` | Your Azure AD tenant |

### Prerequisites to get these injected

- Enable OIDC issuer on the AKS cluster
- Enable Workload Identity on the cluster (`--enable-workload-identity`)
- Annotate the Kubernetes service account with `azure.workload.identity/client-id`
- Label the pod with `azure.workload.identity/use: "true"`
- Create a federated credential on the managed identity pointing to the OIDC issuer + SA + namespace

The Azure SDKs (Python, Go, .NET, Java) all natively support this flow via `DefaultAzureCredential` or `WorkloadIdentityCredential` — they automatically read these env vars and handle the token exchange without any code changes.

---

## Q2: Who creates AZURE_FEDERATED_TOKEN_FILE and when?

Two separate components are responsible, at two different times:

### 1. The Mutating Admission Webhook (at pod creation)

When you label a pod with `azure.workload.identity/use: "true"`, the **Azure Workload Identity webhook** (a mutating admission webhook running in the cluster) intercepts the pod creation request before the pod is scheduled. It mutates the pod spec by injecting:

- The three env vars (`AZURE_FEDERATED_TOKEN_FILE`, `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`)
- A `volumeMount` pointing to where the token file will live (e.g. `/var/run/secrets/azure/tokens/`)
- A `volume` of type `projected` that references the service account token

This all happens **during the API server admission phase**, before the pod ever runs. The webhook reads the `azure.workload.identity/client-id` annotation from the Kubernetes service account to know which client ID to inject.

### 2. The Kubelet (at pod startup, continuously)

The kubelet on the node is responsible for actually **projecting the token file onto disk**. It uses the `projected volume` definition (injected by the webhook above) to request a short-lived OIDC token from the Kubernetes API server for that service account, then writes it to the file path inside the container.

Crucially, the kubelet **refreshes this token automatically** before it expires (tokens are typically valid for 1 hour, refreshed at ~80% of their lifetime). So the file content is kept fresh without any pod restart.

### Timeline summary

```
kubectl apply (pod manifest)
       │
       ▼
API Server → Admission Webhook mutates pod spec
       │        (injects env vars + projected volume)
       ▼
Pod scheduled to node
       │
       ▼
Kubelet starts pod → requests SA token from API server
       │              writes token to file on disk
       ▼
Container starts → AZURE_FEDERATED_TOKEN_FILE already set,
                   file already exists and populated
       │
       ▼
Kubelet keeps refreshing the file every ~48 min
```

So the **env var** is created by the webhook (static, set once), but the **file it points to** is created and maintained by the kubelet (dynamic, refreshed continuously).

---

## Key Terms

- **Workload Identity** — AKS feature enabling pods to authenticate to Azure using Kubernetes service account tokens, without secrets
- **Mutating Admission Webhook** — Kubernetes extension point that intercepts and modifies API objects at creation time
- **Projected Volume** — Kubernetes volume type that projects service account tokens, secrets, and configmaps into a pod
- **Federated Credential** — Azure AD configuration that trusts tokens issued by an external OIDC provider (here: the AKS cluster)
- **OIDC Issuer** — The AKS cluster's endpoint that issues signed JWT tokens for service accounts, trusted by Azure AD
- **Kubelet** — Node-level Kubernetes agent responsible for running pods and managing volumes
- **aad-pod-identity** — Older, now deprecated approach to pod identity in AKS; replaced by Workload Identity
