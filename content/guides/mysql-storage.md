# Setting Up Dex with MySQL Storage

MySQL is a production-grade storage backend for Dex. Like the PostgreSQL backend, the Dex Config Operator reads database connection details from a Kubernetes Secret, keeping credentials separate from the `DexConfig` resource.

## Prerequisites

- A running Kubernetes cluster
- The Dex Config Operator (DCO) installed
- A MySQL instance accessible from the cluster
- `kubectl` access to the cluster

## Step 1: Create the Database Credentials Secret

Create a Kubernetes Secret containing the MySQL connection details. The operator expects the following keys:

| Key | Description |
|---|---|
| `MYSQL_DATABASE` | Name of the database Dex will use. |
| `MYSQL_USER` | Database user with read/write access. |
| `MYSQL_PASSWORD` | Password for the database user. |
| `MYSQL_PORT` | Port the MySQL instance listens on. |
| `MYSQL_SERVICE` | Hostname or in-cluster DNS name of the MySQL service. |

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-credentials
type: Opaque
stringData:
  MYSQL_DATABASE: dex
  MYSQL_USER: dexuser
  MYSQL_PASSWORD: password123
  MYSQL_PORT: "3306"
  MYSQL_SERVICE: mysql.default.svc.cluster.local
```

Apply the Secret:

```bash
kubectl apply -f mysql-credentials.yaml
```

!!! warning
    Replace the example password with a strong, unique value. Never commit unencrypted credentials to version control.

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
    type: mysql
    configSecretRef:
      name: mysql-credentials
```

Apply the configuration:

```bash
kubectl apply -f dex-mysql-config.yaml
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
    type: mysql
    configSecretRef:
      name: mysql-credentials
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-credentials
type: Opaque
stringData:
  MYSQL_DATABASE: dex
  MYSQL_USER: dexuser
  MYSQL_PASSWORD: password123
  MYSQL_PORT: "3306"
  MYSQL_SERVICE: mysql.default.svc.cluster.local
```

## Field Reference

| Field | Description |
|---|---|
| `spec.storage.type` | Set to `mysql` to use a MySQL database. |
| `spec.storage.configSecretRef.name` | Name of the Kubernetes Secret containing the connection details. |

## Verifying the Connection

After applying both resources, confirm Dex has started successfully:

```bash
kubectl get dexconfig dex-config
kubectl logs deployment/dex
```

Look for log entries indicating a successful database connection. If Dex fails to start, verify that:

1. The MySQL instance is reachable from within the cluster.
1. The credentials in the Secret are correct.
1. The target database exists and the user has the required permissions.
1. The MySQL user has been granted appropriate privileges on the target database.
