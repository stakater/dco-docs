# Client CRD Reference

## Overview

| Property | Value |
|---|---|
| **API Group** | `auth.stakater.com` |
| **API Version** | `v1alpha1` |
| **Kind** | `Client` |
| **Scope** | `Namespaced` |

---

## Spec Fields

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `id` | `string` | Yes | - | Pattern: `^[a-zA-Z0-9._-]+$` | A unique identifier for the OAuth2 client. Must contain only alphanumeric characters, dots, underscores, and hyphens. |
| `name` | `string` | Yes | - | - | A human-readable display name for the client. |
| `redirectURIs` | `[]string` | Yes | - | MinItems: 1 | List of allowed redirect URIs for the client. At least one URI must be provided. |
| `public` | `bool` | No | `false` | - | If `true`, the client is a public client (e.g., a single-page application) and does not require a secret. |
| `secretRef` | `object` | Conditional | - | Required if `public` is `false` | Reference to a Secret containing the client secret. See [Secret Reference](#secret-reference). |
| `trustedPeers` | `[]string` | No | - | - | List of client IDs that are trusted peers. Trusted peers can exchange tokens with this client. |
| `logoURL` | `string` | No | - | - | URL of the client logo displayed on the approval screen. |
| `enabled` | `bool` | No | `true` | - | When set to `false`, the client is not registered with Dex. |

### Secret Reference

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `secretRef.name` | `string` | Yes | - | Name of the Secret containing the client secret. |
| `secretRef.namespace` | `string` | Yes | - | Namespace of the Secret. |
| `secretRef.key` | `string` | No | `secret` | Key within the Secret data that holds the client secret value. |

---

## Status Fields

| Field | Type | Description |
|---|---|---|
| `conditions` | `[]Condition` | Standard Kubernetes conditions representing the state of the resource. |
| `observedGeneration` | `int64` | The last `.metadata.generation` observed by the controller. |
| `phase` | `string` | The current phase of the Client (e.g., `Active`, `Failed`). |
| `message` | `string` | A human-readable message describing the current state or error. |
| `lastUpdated` | `string` | Timestamp of the last status update. |
