# Setting Up Dex with PostgreSQL Storage

PostgreSQL is a robust storage backend for Dex, suitable for production deployments that require durability and scalability. The Dex Config Operator reads database connection details from a Kubernetes Secret, keeping credentials out of the `DexConfig` resource itself.

## Prerequisites

- A running Kubernetes cluster
- The Dex Config Operator (DCO) installed
- A PostgreSQL instance accessible from the cluster
- `kubectl` access to the cluster

## Step 1: Create the Database Credentials Secret

Create a Kubernetes Secret containing the PostgreSQL connection details. The operator expects the following keys:

| Key | Description |
|---|---|
| `POSTGRESQL_DATABASE` | Name of the database Dex will use. |
| `POSTGRESQL_USER` | Database user with read/write access. |
| `POSTGRESQL_PASSWORD` | Password for the database user. |
| `POSTGRESQL_PORT` | Port the PostgreSQL instance listens on. |
| `POSTGRESQL_SERVICE` | Hostname or in-cluster DNS name of the PostgreSQL service. |
| `POSTGRESQL_SSL` | SSL mode for the connection (e.g., `disable`, `require`, `verify-full`). |

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-credentials
type: Opaque
stringData:
  POSTGRESQL_DATABASE: dex
  POSTGRESQL_USER: dexuser
  POSTGRESQL_PASSWORD: password123
  POSTGRESQL_PORT: "5432"
  POSTGRESQL_SERVICE: postgres.default.svc.cluster.local
  POSTGRESQL_SSL: disable
```

Apply the Secret:

```bash
kubectl apply -f postgres-credentials.yaml
```

!!! warning
    Replace the example password with a strong, unique value. Never commit plaintext credentials to version control.

## Step 2: Create the DexConfig Resource

Create a `DexConfig` resource that references the Secret:

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: DexConfig
metadata:
  name: dex-config
spec:
  issuer: https://dex.example.com
  storage:
    type: postgres
    configSecretRef:
      name: postgres-credentials
```

Apply the configuration:

```bash
kubectl apply -f dex-postgres-config.yaml
```

## Complete Manifest

The following manifest combines both resources for convenience:

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: DexConfig
metadata:
  name: dex-config
spec:
  issuer: https://dex.example.com
  storage:
    type: postgres
    configSecretRef:
      name: postgres-credentials
---
apiVersion: v1
kind: Secret
metadata:
  name: postgres-credentials
type: Opaque
stringData:
  POSTGRESQL_DATABASE: dex
  POSTGRESQL_USER: dexuser
  POSTGRESQL_PASSWORD: password123
  POSTGRESQL_PORT: "5432"
  POSTGRESQL_SERVICE: postgres.default.svc.cluster.local
  POSTGRESQL_SSL: disable
```

## Field Reference

| Field | Description |
|---|---|
| `spec.storage.type` | Set to `postgres` to use a PostgreSQL database. |
| `spec.storage.configSecretRef.name` | Name of the Kubernetes Secret containing the connection details. |

## Verifying the Connection

After applying both resources, confirm Dex has started successfully:

```bash
kubectl get dexconfig dex-config
kubectl logs deployment/dex
```

Look for log entries indicating a successful database connection. If Dex fails to start, verify that:

1. The PostgreSQL instance is reachable from within the cluster.
1. The credentials in the Secret are correct.
1. The target database exists and the user has the required permissions.
1. The SSL mode matches the PostgreSQL server configuration.
