# Setting Up Dex with Kubernetes Storage

Kubernetes storage is the simplest storage backend for Dex. It stores all data as Kubernetes Custom Resources within the cluster, requiring no external database. This makes it ideal for development environments, small deployments, or scenarios where operational simplicity is a priority.

## Prerequisites

- A running Kubernetes cluster
- The Dex Config Operator (DCO) installed
- `kubectl` access to the cluster

## Configuration

Create a `DexConfig` resource that uses Kubernetes as the storage backend:

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: DexConfig
metadata:
  name: dex-config
spec:
  issuer: https://dex.example.com
  storage:
    type: kubernetes
    config:
      inCluster: true
  web:
    http: 0.0.0.0:5556
  oauth2:
    skipApprovalScreen: true
  expiry:
    signingKeys: "6h"
    idTokens: "24h"
  logger:
    level: info
    format: text
```

## Field Reference

| Field | Description |
|---|---|
| `spec.issuer` | The externally accessible URL of the Dex instance. This must match the URL clients use to reach Dex. |
| `spec.storage.type` | Set to `kubernetes` to use in-cluster CRD-based storage. |
| `spec.storage.config.inCluster` | Must be `true` when Dex is running inside the Kubernetes cluster. |
| `spec.web.http` | The address and port Dex listens on for HTTP requests. |
| `spec.oauth2.skipApprovalScreen` | When `true`, users are not prompted to approve application access after authentication. |
| `spec.expiry.signingKeys` | The duration before signing keys are rotated (e.g., `"6h"`). |
| `spec.expiry.idTokens` | The lifetime of issued ID tokens (e.g., `"24h"`). |
| `spec.logger.level` | Log verbosity. Accepted values: `debug`, `info`, `warn`, `error`. |
| `spec.logger.format` | Log output format. Accepted values: `text`, `json`. |

## Applying the Configuration

Save the manifest above to a file and apply it:

```bash
kubectl apply -f dex-kubernetes-storage.yaml
```

Verify the resource was created:

```bash
kubectl get dexconfig dex-config
```

## When to Use Kubernetes Storage

- **Development and testing** -- no external dependencies required.
- **Small-scale deployments** -- suitable when the number of tokens and authentication sessions is low.
- **Single-cluster setups** -- data is stored within the same cluster that runs Dex.

!!! warning
    Kubernetes storage is not recommended for large-scale production deployments. For high-availability or multi-cluster environments, consider using [PostgreSQL](postgresql-storage.md) or [MySQL](mysql-storage.md) as the storage backend.
