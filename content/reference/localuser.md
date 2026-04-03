# LocalUser CRD Reference

## Overview

| Property | Value |
|---|---|
| **API Group** | `auth.stakater.com` |
| **API Version** | `v1alpha1` |
| **Kind** | `LocalUser` |
| **Scope** | Namespaced |

---

## Spec Fields

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `userID` | `string` | Yes | - | - | A unique identifier for the local user. This value is used as the user's subject claim in tokens. |
| `credentialSecretRef` | `object` | Yes | - | - | Reference to a Secret containing the user's credentials. See [Credential Secret Reference](#credential-secret-reference). |
| `enabled` | `bool` | No | `true` | - | When set to `false`, the user is not registered with Dex and cannot authenticate. |

### Credential Secret Reference

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `credentialSecretRef.name` | `string` | Yes | - | Name of the Secret containing the user credentials. |
| `credentialSecretRef.namespace` | `string` | Yes | - | Namespace of the Secret. |
| `credentialSecretRef.key` | `string` | No | - | Key within the Secret data. If omitted, the Secret is expected to contain individual keys as described below. |

---

## Credential Secret Fields

The Secret referenced by `credentialSecretRef` must contain the following keys in its `data` (or `stringData`) section.

| Key | Required | Description |
|---|---|---|
| `username` | Yes | The username for the local user. Used as the display name. |
| `email` | Yes | The email address for the local user. Used as the login identifier. |
| `password` | Yes | The user's password, hashed using bcrypt. Generate with: `htpasswd -nbBC 10 "" 'password' \| cut -d: -f2` |
| `groups` | No | Groups the user belongs to. Accepts either a JSON array (e.g., `["admin","developers"]`) or a comma-separated string (e.g., `admin,developers`). |

!!! note
    The `enablePasswordDB` field in the `DexConfig` spec must be set to `true` for local user authentication to function.

---

## Status Fields

| Field | Type | Description |
|---|---|---|
| `conditions` | `[]Condition` | Standard Kubernetes conditions representing the state of the resource. |
| `observedGeneration` | `int64` | The last `.metadata.generation` observed by the controller. |
| `phase` | `string` | The current phase of the LocalUser (e.g., `Active`, `Failed`). |
| `message` | `string` | A human-readable message describing the current state or error. |
| `lastUpdated` | `string` | Timestamp of the last status update. |
