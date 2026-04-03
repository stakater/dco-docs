# Key Features

## Security-first design

All sensitive data -- client secrets, connector credentials, database passwords -- is stored in Kubernetes Secrets. CRD manifests contain only Secret references, so credentials never appear in plain text inside your resource definitions or Git repositories.

## Event-driven configuration

DCO watches for changes to both CRDs and the Kubernetes Secrets they reference. When any watched resource is created, updated, or deleted, DCO automatically regenerates the Dex configuration. There is no polling interval and no manual trigger required.

## Automatic Dex restart

After regenerating the configuration, DCO performs a rolling restart of the Dex Deployment. This ensures the running Dex instance always reflects the latest desired state without operator intervention.

## Secret watching

DCO monitors every Secret referenced by a DexConnector, DexClient, or storage configuration. If a Secret is rotated or modified externally (for example, by a secrets manager), DCO detects the change and regenerates the Dex configuration accordingly.

## Comprehensive validation

Each CRD reports detailed status conditions that surface configuration errors early. When a resource is invalid -- for example, a referenced Secret does not exist or a required field is missing -- the status condition provides a clear, actionable message.

!!! note
    Status conditions follow the standard Kubernetes condition pattern (`type`, `status`, `reason`, `message`) and can be queried with `kubectl get` or consumed by monitoring tools.

## Provider abstraction layer

DCO introduces an abstraction over OIDC provider configuration. While Dex is the primary supported backend today, this design enables future support for additional OIDC providers without changes to the CRD API surface.

## Multiple storage backends

DCO supports the full set of Dex storage backends:

| Backend | Use case |
|---|---|
| **Kubernetes** | Default; uses Kubernetes CRDs as the storage layer. No external database required. |
| **PostgreSQL** | Production workloads requiring a relational database. |
| **MySQL** | Alternative relational database option. |
| **SQLite3** | Lightweight single-node or development setups. |
| **Memory** | Ephemeral; suitable for testing only. |

Storage backend credentials are referenced through Secrets, consistent with the security-first approach.

## Flexible CRD watching scope

DCO can be configured to watch CRDs in a single namespace or across all namespaces in the cluster. Namespace-scoped mode is useful for multi-tenant environments where each team manages its own OIDC resources.

## Enable and disable resources

Individual DexConnector or DexClient resources can be disabled without deleting them. Setting the `enabled` field to `false` removes the resource from the generated Dex configuration while preserving the CRD for later re-enablement.

!!! tip
    Disabling a resource is useful for temporarily removing an identity provider connector during maintenance without losing its configuration.
