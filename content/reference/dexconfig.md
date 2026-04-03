# DexConfig CRD Reference

## Overview

| Property | Value |
|---|---|
| **API Group** | `auth.stakater.com` |
| **API Version** | `v1alpha1` |
| **Kind** | `DexConfig` |
| **Scope** | Namespaced (one per cluster) |

### Print Columns

| Name | JSON Path | Description |
|---|---|---|
| Issuer | `.spec.issuer` | The issuer URL configured for Dex |
| Phase | `.status.phase` | Current phase of the DexConfig resource |
| Age | `.metadata.creationTimestamp` | Time since creation |

---

## Spec Fields

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `issuer` | `string` | Yes | - | Pattern: `^https?://.*` | The issuer URL that Dex will use. Must be a valid HTTP or HTTPS URL. This is the public-facing URL of the Dex instance. |
| `storage` | `object` | Yes | - | - | Configures the storage backend for Dex. See [Storage](#storage). |
| `web` | `object` | No | - | - | Configures the HTTP/HTTPS web server. See [Web](#web). |
| `grpc` | `object` | No | - | - | Configures the gRPC server. See [gRPC](#grpc). |
| `telemetry` | `object` | No | - | - | Configures the telemetry/metrics endpoint. See [Telemetry](#telemetry). |
| `oauth2` | `object` | No | - | - | Configures OAuth2 behavior. See [OAuth2](#oauth2). |
| `expiry` | `object` | No | - | - | Configures token and key expiry durations. See [Expiry](#expiry). |
| `logger` | `object` | No | - | - | Configures logging behavior. See [Logger](#logger). |
| `frontend` | `object` | No | - | - | Configures the web frontend appearance. See [Frontend](#frontend). |
| `enablePasswordDB` | `bool` | No | `false` | - | Enables the local password database. Must be set to `true` when using `LocalUser` resources. |

### Storage

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `storage.type` | `string` | Yes | - | - | The storage backend type (e.g., `postgres`, `mysql`, `sqlite3`, `memory`). |
| `storage.config` | `object` | No | - | - | Inline storage configuration. |
| `storage.configSecretRef` | `object` | No | - | - | Reference to a Secret containing the storage configuration. See [Database Secret Key Reference](#database-secret-key-reference). |

### Web

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `web.http` | `string` | No | - | - | The address to listen on for HTTP traffic (e.g., `0.0.0.0:5556`). |
| `web.https` | `string` | No | - | - | The address to listen on for HTTPS traffic (e.g., `0.0.0.0:5554`). |
| `web.tlsCert` | `string` | No | - | - | Path to the TLS certificate file. |
| `web.tlsKey` | `string` | No | - | - | Path to the TLS private key file. |
| `web.allowedOrigins` | `[]string` | No | - | - | List of allowed CORS origins for web requests. |

### gRPC

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `grpc.addr` | `string` | No | - | - | The address to listen on for gRPC traffic (e.g., `0.0.0.0:5557`). |
| `grpc.tlsCert` | `string` | No | - | - | Path to the TLS certificate file for gRPC. |
| `grpc.tlsKey` | `string` | No | - | - | Path to the TLS private key file for gRPC. |
| `grpc.tlsClientCA` | `string` | No | - | - | Path to the TLS client CA file for mutual TLS authentication. |
| `grpc.reflection` | `bool` | No | `false` | - | Enables gRPC server reflection. |

### Telemetry

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `telemetry.http` | `string` | No | - | - | The address to listen on for telemetry/metrics traffic (e.g., `0.0.0.0:5558`). |

### OAuth2

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `oauth2.skipApprovalScreen` | `bool` | No | `false` | - | If `true`, skips the approval screen after user authentication. |
| `oauth2.alwaysShowLoginScreen` | `bool` | No | `false` | - | If `true`, always shows the login screen even when only one connector is configured. |
| `oauth2.passwordConnector` | `string` | No | - | - | The ID of the connector to use for password grants. |
| `oauth2.responseTypes` | `[]string` | No | - | - | List of allowed OAuth2 response types (e.g., `code`, `token`, `id_token`). |

### Expiry

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `expiry.signingKeys` | `string` | No | `6h` | - | Duration after which signing keys are rotated. |
| `expiry.idTokens` | `string` | No | `24h` | - | Duration for which ID tokens are valid. |
| `expiry.authRequests` | `string` | No | - | - | Duration for which authorization requests are valid. |
| `expiry.deviceRequests` | `string` | No | - | - | Duration for which device authorization requests are valid. |
| `expiry.refreshTokens` | `object` | No | - | - | Configures refresh token expiry behavior. See [Refresh Tokens](#refresh-tokens). |

#### Refresh Tokens

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `expiry.refreshTokens.validIfNotUsedFor` | `string` | No | - | - | Duration after which an unused refresh token expires. |
| `expiry.refreshTokens.absoluteLifetime` | `string` | No | - | - | Absolute lifetime of a refresh token regardless of usage. |
| `expiry.refreshTokens.reuseInterval` | `string` | No | - | - | Interval during which a refresh token can be reused without rotation. |
| `expiry.refreshTokens.disableRotation` | `bool` | No | `false` | - | If `true`, disables refresh token rotation. |

### Logger

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `logger.level` | `string` | No | `info` | Enum: `debug`, `info`, `error` | The log level for Dex. |
| `logger.format` | `string` | No | `text` | Enum: `text`, `json` | The log output format. |

### Frontend

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `frontend.issuer` | `string` | No | - | - | The name displayed on the login page. |
| `frontend.logoURL` | `string` | No | - | - | URL of the logo displayed on the login page. |
| `frontend.theme` | `string` | No | - | - | The frontend theme to use. |
| `frontend.dir` | `string` | No | - | - | Path to a directory containing custom web assets. |
| `frontend.extra` | `map[string]string` | No | - | - | Extra key-value pairs passed to the frontend templates. |

---

## Status Fields

| Field | Type | Description |
|---|---|---|
| `conditions` | `[]Condition` | Standard Kubernetes conditions representing the state of the resource. |
| `observedGeneration` | `int64` | The last `.metadata.generation` observed by the controller. |
| `phase` | `string` | The current phase of the DexConfig (e.g., `Active`, `Failed`). |
| `message` | `string` | A human-readable message describing the current state or error. |
| `lastUpdated` | `string` | Timestamp of the last status update. |
| `appliedConfiguration` | `object` | A snapshot of the last successfully applied configuration. |

---

## Database Secret Key Reference

When using `storage.configSecretRef`, the referenced Secret must contain the appropriate keys depending on the database backend.

### PostgreSQL

| Secret Key | Description |
|---|---|
| `POSTGRESQL_HOST` | Database server hostname. |
| `POSTGRESQL_PORT` | Database server port. |
| `POSTGRESQL_DATABASE` | Name of the database. |
| `POSTGRESQL_USER` | Database user. |
| `POSTGRESQL_PASSWORD` | Database password. |
| `POSTGRESQL_SSL_MODE` | SSL mode for the connection (e.g., `disable`, `require`, `verify-ca`, `verify-full`). |
| `POSTGRESQL_SSL_CA` | Path or content of the CA certificate for SSL connections. |

### MySQL

| Secret Key | Description |
|---|---|
| `MYSQL_HOST` | Database server hostname. |
| `MYSQL_PORT` | Database server port. |
| `MYSQL_DATABASE` | Name of the database. |
| `MYSQL_USER` | Database user. |
| `MYSQL_PASSWORD` | Database password. |
| `MYSQL_SSL_CA` | Path or content of the CA certificate for SSL connections. |
| `MYSQL_SSL_CERT` | Path or content of the client certificate for SSL connections. |
| `MYSQL_SSL_KEY` | Path or content of the client private key for SSL connections. |
