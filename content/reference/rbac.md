# RBAC Permissions

This page lists the Kubernetes RBAC permissions required by the Dex Config Operator and the pre-built ClusterRoles shipped for end users.

## Operator Service Account Permissions

The operator's own service account needs the following cluster-level permissions to reconcile CRDs, manage the generated config Secret, and restart Dex.

### Core API Resources

| API Group | Resource | Verbs |
|---|---|---|
| `""` (core) | `secrets` | get, list, watch, create, update, patch |
| `apps` | `deployments` | get, list, watch, update, patch, delete |

### Custom Resource Definitions

| API Group | Resource | Verbs |
|---|---|---|
| `auth.stakater.com` | `dexconfigs` | get, list, watch, create, update, patch, delete |
| `auth.stakater.com` | `dexconfigs/status` | get, update, patch |
| `auth.stakater.com` | `dexconfigs/finalizers` | update |
| `auth.stakater.com` | `connectors` | get, list, watch, create, update, patch, delete |
| `auth.stakater.com` | `connectors/status` | get, update, patch |
| `auth.stakater.com` | `connectors/finalizers` | update |
| `auth.stakater.com` | `clients` | get, list, watch, create, update, patch, delete |
| `auth.stakater.com` | `clients/status` | get, update, patch |
| `auth.stakater.com` | `clients/finalizers` | update |
| `auth.stakater.com` | `localusers` | get, list, watch, create, update, patch, delete |
| `auth.stakater.com` | `localusers/status` | get, update, patch |
| `auth.stakater.com` | `localusers/finalizers` | update |

### Leader Election

When `--leader-elect` is enabled, the operator also requires:

| API Group | Resource | Verbs |
|---|---|---|
| `coordination.k8s.io` | `leases` | get, list, watch, create, update, patch, delete |
| `""` (core) | `events` | create, patch |

## Pre-built ClusterRoles

The Helm chart ships three ClusterRoles for granting access to DCO custom resources. Bind them to users or groups with a ClusterRoleBinding or namespace-scoped RoleBinding as needed.

### Admin

**ClusterRole:** `dex-config-operator-admin-role`

Full CRUD access to all DCO custom resources. Intended for platform administrators who manage the complete Dex configuration lifecycle.

| Resource | Verbs |
|---|---|
| `dexconfigs` | get, list, watch, create, update, patch, delete |
| `connectors` | get, list, watch, create, update, patch, delete |
| `clients` | get, list, watch, create, update, patch, delete |
| `localusers` | get, list, watch, create, update, patch, delete |

### Editor

**ClusterRole:** `dex-config-operator-editor-role`

Create, update, and delete access without the ability to manage RBAC bindings. Suitable for teams that own specific connectors or clients.

| Resource | Verbs |
|---|---|
| `dexconfigs` | get, list, watch, create, update, patch, delete |
| `connectors` | get, list, watch, create, update, patch, delete |
| `clients` | get, list, watch, create, update, patch, delete |
| `localusers` | get, list, watch, create, update, patch, delete |

### Viewer

**ClusterRole:** `dex-config-operator-viewer-role`

Read-only access. Useful for auditing or monitoring dashboards.

| Resource | Verbs |
|---|---|
| `dexconfigs` | get, list, watch |
| `connectors` | get, list, watch |
| `clients` | get, list, watch |
| `localusers` | get, list, watch |

## Example RoleBinding

Bind the Viewer role to a group for a specific namespace:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dco-viewers
  namespace: auth
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: dex-config-operator-viewer-role
subjects:
  - kind: Group
    name: sre-team
    apiGroup: rbac.authorization.k8s.io
```
