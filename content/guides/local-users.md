# Managing Local Users

The Dex Config Operator supports local user accounts through the `LocalUser` custom resource. Local users authenticate with a username and password directly against Dex, without requiring an external identity provider. When any `LocalUser` resource exists in the cluster, the operator automatically enables the `passwordDB` connector in Dex.

## Prerequisites

- The Dex Config Operator installed
- A tool to generate `bcrypt` password hashes

## Generating `bcrypt` Password Hashes

`LocalUser` credentials require bcrypt-hashed passwords. Use one of the following methods to generate a hash.

Using `htpasswd`:

```bash
htpasswd -bnBC 10 "" your-password | tr -d ':\n'
```

Using Python:

```bash
python3 -c 'import bcrypt; print(bcrypt.hashpw(b"your-password", bcrypt.gensalt()).decode())'
```

!!! warning
    Never store raw passwords in your manifests or version control. Always use the `bcrypt` hash.

## Credential Secret Formats

The `LocalUser` resource references a Kubernetes Secret for credentials. The operator supports two secret formats: structured and flat.

### Structured Format

The structured format stores all credential fields as a single JSON value under one key:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: admin-user-credentials
type: Opaque
stringData:
  credentials: |
    {
      "username": "admin",
      "email": "admin@example.com",
      "hash": "$2y$10$eiDkS3GlH5GvOvsnMKrfGOEXrGEmHJMsLCaMPKGBFTraGHOFpGOi6",
      "groups": ["admins", "developers"]
    }
```

The corresponding `LocalUser` resource:

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: LocalUser
metadata:
  name: admin-user
spec:
  secretRef:
    name: admin-user-credentials
    key: credentials
  enabled: true
```

### Flat Keys Format

The flat format stores each credential field as a separate key in the Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dev-user-credentials
type: Opaque
stringData:
  username: developer
  email: developer@example.com
  hash: "$2y$10$eiDkS3GlH5GvOvsnMKrfGOEXrGEmHJMsLCaMPKGBFTraGHOFpGOi6"
  groups: "developers,qa"
```

The corresponding `LocalUser` resource:

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: LocalUser
metadata:
  name: dev-user
spec:
  secretRef:
    name: dev-user-credentials
  enabled: true
```

!!! note
    When using the flat keys format, omit the `key` field in `secretRef`. The operator detects the format automatically based on whether `key` is specified.

## Complete Example

The following manifest creates a local admin user using the structured format:

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: LocalUser
metadata:
  name: admin-user
spec:
  secretRef:
    name: admin-user-credentials
    key: credentials
  enabled: true
---
apiVersion: v1
kind: Secret
metadata:
  name: admin-user-credentials
type: Opaque
stringData:
  credentials: |
    {
      "username": "admin",
      "email": "admin@example.com",
      "hash": "$2y$10$eiDkS3GlH5GvOvsnMKrfGOEXrGEmHJMsLCaMPKGBFTraGHOFpGOi6",
      "groups": ["admins", "developers"]
    }
```

Apply the resources:

```bash
kubectl apply -f admin-user.yaml
```

## Automatic passwordDB Connector

When the operator detects one or more `LocalUser` resources in the cluster, it automatically adds the `passwordDB` connector to the Dex configuration. You do not need to configure this connector manually. If all `LocalUser` resources are deleted or disabled, the `passwordDB` connector is removed.

## Field Reference

| Field | Description |
|---|---|
| `spec.secretRef.name` | Name of the Kubernetes Secret containing the user credentials. |
| `spec.secretRef.key` | Key within the Secret that holds a JSON credential object (structured format only). |
| `spec.enabled` | Set to `true` to activate the user, `false` to disable the account. |

## Credential Fields

| Field | Description |
|---|---|
| `username` | The login username. |
| `email` | The user's email address, used as a unique identifier in Dex. |
| `hash` | `bcrypt` hash of the user's password. |
| `groups` | Groups the user belongs to. JSON array (structured) or comma-separated string (flat). |
