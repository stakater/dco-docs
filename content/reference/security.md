# Security Model

The Dex Config Operator is designed around the principle that sensitive data must never appear in custom resources. This page explains the layered security controls that protect credentials at rest and in transit.

## Secrets-First Design

All sensitive data -- client secrets, connector credentials, user passwords -- is stored exclusively in Kubernetes Secrets. CRDs contain only structural references that point to those Secrets. This means:

- **CRDs are safe to store in Git.** They contain no embedded credentials.
- **Kubernetes RBAC on Secrets controls who can read sensitive data.** Access to a CRD does not grant access to the underlying secret values.
- **Audit logs remain clean.** Watching CRD changes does not expose credentials.

## Secret Reference Patterns

DCO uses two reference structures depending on the resource type.

### SecretReference

Used by **Connectors** and **LocalUsers**. Points to an entire Secret by name, namespace, and key.

```yaml
secretRef:
  name: github-connector-credentials
  namespace: auth
  key: config.yaml
```

The operator reads the referenced key from the Secret and merges its content into the generated Dex configuration. The CRD itself never contains the credential values.

### SecretKeyReference

Used by **Clients**. Points to a single key within a Secret.

```yaml
secretRef:
  name: my-app-client
  namespace: auth
  key: client-secret
```

The operator extracts the value of the specified key and injects it as the client secret in the generated configuration.

### Key Difference

| Pattern | Used By | What It References |
|---|---|---|
| `SecretReference` | Connectors, LocalUsers | A structured block of configuration stored under a single key |
| `SecretKeyReference` | Clients | A single scalar value (e.g., a client secret string) |

## Selective Secret Watching

The operator does not watch every Secret in the cluster. Only Secrets that carry the following label trigger reconciliation when they change:

```yaml
metadata:
  labels:
    dex-config-operator.stakater.com/watch: "true"
```

This label-based filter:

- **Reduces API server load** by ignoring unrelated Secret events.
- **Prevents accidental reconciliation** from changes to Secrets the operator does not manage.
- **Provides an explicit opt-in mechanism** so that teams control which Secrets participate in the Dex configuration pipeline.

If a Secret referenced by a CRD does not have this label, the operator resolves it during reconciliation but does not set up a watch for future changes. Adding the label enables automatic re-reconciliation whenever the Secret is updated.

## Pod Security

The operator pod is configured with a restricted security context that follows Kubernetes pod security best practices.

| Control | Setting |
|---|---|
| `runAsNonRoot` | `true` |
| `seccompProfile.type` | `RuntimeDefault` |
| `capabilities.drop` | `ALL` |
| `allowPrivilegeEscalation` | `false` |
| `readOnlyRootFilesystem` | `true` |

These settings ensure that even if the operator container is compromised, the blast radius is minimized. The pod cannot escalate privileges, write to the filesystem, or use any Linux capabilities.

## Cross-Namespace References

CRDs in one namespace can reference Secrets in another namespace. This enables a central secret management pattern where a platform team maintains credentials in a dedicated namespace while application teams define CRDs in their own namespaces.

```text
Namespace: team-a                    Namespace: central-secrets
+-----------------+                  +---------------------------+
| Connector       |  --- secretRef ---> | Secret                 |
|  (OIDC GitHub)  |                  |  github-connector-creds  |
+-----------------+                  +---------------------------+
```

To use cross-namespace references, the operator's service account must have permission to read Secrets in the target namespace. The default RBAC configuration grants cluster-wide Secret read access to support this pattern.

!!! warning
    Cross-namespace Secret access means the operator can read any labeled Secret in the cluster. Restrict the operator's service account permissions if your security policy requires namespace isolation.
