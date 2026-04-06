# Core Concepts

## Configuration Flow

The Dex Config Operator follows a declarative workflow. Users express their desired state through CRDs, and the operator converges the running Dex instance to match.

```mermaid
sequenceDiagram
    participant User
    participant K8s as Kubernetes API
    participant Op as Operator
    participant Sec as Config Secret
    participant Dex as Dex Deployment

    User->>K8s: Create/Update CRD
    K8s->>Op: Change detected
    Op->>Op: Validate resource
    Op->>K8s: Collect all CRDs + referenced Secrets
    Op->>Sec: Write generated config.yaml
    Sec-->>Dex: Rolling restart triggered
```

1. The user creates or updates a CRD (DexConfig, Connector, Client, or LocalUser).
1. The operator detects the change and validates the resource.
1. The operator collects every CRD across the cluster and reads any referenced Secrets.
1. A complete `config.yaml` is generated and written into a Kubernetes Secret.
1. The Dex Deployment is restarted to pick up the new configuration.

## Secret References

CRDs that need sensitive data do not embed it inline. Instead, they reference a Kubernetes Secret:

```yaml
configSecretRef:
  name: my-oidc-secret
  namespace: dex        # optional, defaults to the CRD's namespace
  key: config           # optional, defaults vary by resource type
```

When the operator resolves a Secret reference, it labels the Secret so that future changes are detected automatically:

```yaml
labels:
  dex-config-operator.stakater.com/watch: "true"
```

If the referenced Secret is later updated (for example, during a credential rotation), the operator detects the change and regenerates the configuration. This ensures the Dex configuration always reflects the latest secret values without manual intervention.

## Status Conditions

Every CRD managed by the operator reports its health through a set of standard status conditions. Each condition has a `type`, a `status` (`True`, `False`, or `Unknown`), and a human-readable `message`.

| Condition | Meaning |
|---|---|
| **Ready** | The resource is healthy and contributing to the active Dex configuration. |
| **SecretResolved** | All Secret references in the resource have been resolved successfully. A `False` value indicates a missing or inaccessible Secret. |
| **ConfigurationValid** | The resource passed validation and produces a valid Dex configuration block. |
| **Synced** | The generated configuration has been written to the config Secret and Dex has been restarted. |

A resource is considered healthy when all four conditions are `True`.

## Phases

Each CRD carries a `phase` field in its status that provides a quick summary of the resource lifecycle:

| Phase | Description |
|---|---|
| **Pending** | The resource has been created but has not yet been fully processed. This is the initial state. |
| **Ready** | The resource is valid, all Secrets are resolved, and the configuration has been synced to Dex. |
| **Failed** | An error occurred. Inspect the status conditions for details. The operator will retry after 1 minute. |

```text
Pending ──► Ready
   │
   └──► Failed
```

## Enable and Disable

Every CRD includes an `enabled` field that defaults to `true`. Setting it to `false` excludes the resource from the generated Dex configuration without deleting it from the cluster.

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: Connector
metadata:
  name: github
spec:
  enabled: false
  # ... remaining spec
```

This is useful when you need to temporarily remove a connector or client from Dex (for example, during maintenance or debugging) but want to preserve the full resource definition for later re-enablement.

When a resource is disabled:

- It is excluded from the next configuration generation cycle.
- Its status phase remains **Ready** (the resource itself is valid; it is simply not active).
- Re-enabling it triggers an immediate update that adds it back to the Dex configuration.

## Singleton DexConfig

The **DexConfig** CRD is a singleton -- only one instance is permitted per cluster. It defines the global Dex settings such as the issuer URL, storage backend, and web server configuration.

If a second DexConfig resource is created, the operator rejects it and sets the resource phase to **Failed** with a descriptive error message.

The singleton constraint exists because Dex itself operates with a single configuration file. Allowing multiple DexConfig resources would create ambiguity about which settings should take effect.
