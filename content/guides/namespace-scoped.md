# Namespace-Scoped vs Cluster-Wide Operation

By default, the Dex Config Operator watches all namespaces for `Client`, `Connector`, and `LocalUser` resources. You can restrict the operator to a single namespace when multi-tenancy or RBAC boundaries require it.

## Default Behavior: Cluster-Wide

When no `--watch-namespace` flag is set, the operator reconciles resources across every namespace in the cluster. This is the simplest configuration and works well for single-team or single-tenant clusters.

```text
┌─────────────────────────────────────────────┐
│  Cluster                                    │
│                                             │
│  namespace: team-a     namespace: team-b    │
│  ┌────────────┐        ┌────────────┐       │
│  │ Client     │        │ Client     │       │
│  │ Connector  │        │ Connector  │       │
│  │ LocalUser  │        │ LocalUser  │       │
│  └────────────┘        └────────────┘       │
│           ▲                  ▲              │
│           └──────┬───────────┘              │
│                  │                          │
│          ┌───────┴────────┐                 │
│          │  DCO Operator  │                 │
│          │  (all ns)      │                 │
│          └────────────────┘                 │
└─────────────────────────────────────────────┘
```

## Single-Namespace Mode

To restrict the operator to a single namespace, set the `--watch-namespace` flag on the operator deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dex-config-operator
spec:
  template:
    spec:
      containers:
        - name: manager
          args:
            - "--watch-namespace=team-a"
```

With this flag set, the operator only reconciles `Client`, `Connector`, and `LocalUser` resources in the `team-a` namespace. Resources in other namespaces are ignored.

## DexConfig Is Always Global

The `DexConfig` resource is cluster-scoped and is always reconciled regardless of the `--watch-namespace` setting. There is exactly one `DexConfig` per cluster, and it defines the global Dex configuration (issuer, storage, expiry, frontend).

| Resource | Affected by `--watch-namespace` |
|---|---|
| `DexConfig` | No -- always cluster-scoped. |
| `Client` | Yes -- only resources in the watched namespace are reconciled. |
| `Connector` | Yes -- only resources in the watched namespace are reconciled. |
| `LocalUser` | Yes -- only resources in the watched namespace are reconciled. |

## When to Use Single-Namespace Mode

- **Multi-tenant clusters** where teams should only manage their own identity resources.
- **Least-privilege RBAC** policies that restrict the operator's service account to a specific namespace.
- **Multiple Dex instances** where each operator instance manages a separate namespace.

## Running Multiple Operator Instances

You can run multiple instances of the operator, each watching a different namespace, to support independent Dex configurations per team:

```yaml
# Operator for team-a
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dex-config-operator-team-a
  namespace: team-a
spec:
  template:
    spec:
      containers:
        - name: manager
          args:
            - "--watch-namespace=team-a"
            - "--dex-namespace=team-a"
            - "--dex-deployment-name=dex-team-a"
---
# Operator for team-b
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dex-config-operator-team-b
  namespace: team-b
spec:
  template:
    spec:
      containers:
        - name: manager
          args:
            - "--watch-namespace=team-b"
            - "--dex-namespace=team-b"
            - "--dex-deployment-name=dex-team-b"
```

## RBAC Considerations

When using single-namespace mode, you can tighten the operator's RBAC to only allow access to the target namespace. This reduces the blast radius of any misconfiguration:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dex-config-operator
  namespace: team-a
rules:
  - apiGroups: ["auth.stakater.com"]
    resources: ["clients", "connectors", "localusers"]
    verbs: ["get", "list", "watch", "update", "patch"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get", "list", "watch"]
```

!!! note
    The `DexConfig` resource remains cluster-scoped, so the operator still needs a `ClusterRole` for that resource even when running in single-namespace mode.
