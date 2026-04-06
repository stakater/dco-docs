# Quick Start

This guide takes you from a fresh operator deployment to a fully rendered Dex configuration. By the end you will have a working DexConfig, a Google OAuth Connector, and a Client — all managed declaratively.

## Prerequisites

- A running Kubernetes cluster with DCO installed (see [Installation](installation.md))
- `kubectl` configured to communicate with the cluster

## 1. Create a DexConfig resource

The DexConfig resource defines the core Dex server settings — issuer URL, storage backend, web listener, and OAuth2 behaviour.

Create a file named `dex-config.yaml`:

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: DexConfig
metadata:
  name: dex-config
spec:
  issuer: https://dex.example.com
  storage:
    type: kubernetes
    config:
      inCluster: true
  web:
    http: 0.0.0.0:5556
  oauth2:
    skipApprovalScreen: true
```

Apply it:

```bash
kubectl apply -f dex-config.yaml
```

## 2. Create a Connector

Connectors link Dex to upstream identity providers. This example configures Google OAuth.

First, create a Secret containing the Google OAuth client credentials:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: google-connector-secret
type: Opaque
stringData:
  clientID: "your-google-client-id"
  clientSecret: "your-google-client-secret"
```

Save this as `google-connector-secret.yaml` and apply it:

```bash
kubectl apply -f google-connector-secret.yaml
```

Next, create the Connector resource. Save the following as `google-connector.yaml`:

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: Connector
metadata:
  name: google-connector
spec:
  type: google
  id: google
  name: Google
  configSecretRef:
    name: google-connector-secret
  enabled: true
```

Apply it:

```bash
kubectl apply -f google-connector.yaml
```

## 3. Create a Client

Clients represent applications that authenticate through Dex. Save the following as `example-client.yaml`:

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: Client
metadata:
  name: example-app
spec:
  id: example-app
  name: Example Application
  redirectURIs:
    - https://app.example.com/callback
  public: false
  secretRef:
    name: example-app-secret
    key: secret
  enabled: true
```

Create the corresponding client secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-app-secret
type: Opaque
stringData:
  secret: "example-app-secret-value"
```

Apply both:

```bash
kubectl apply -f example-app-secret.yaml
kubectl apply -f example-client.yaml
```

## 4. Verify the generated configuration

DCO assembles all resources into a single Dex configuration Secret. Inspect it:

```bash
kubectl get secret dex -n dex -o jsonpath='{.data.config\.yaml}' | base64 -d
```

The output should include your issuer, storage settings, the Google connector, and the example client.

## 5. Check resource statuses

Confirm that each custom resource is healthy:

```bash
kubectl get dexconfig
kubectl get connectors
kubectl get clients
```

All resources should show a `Ready` phase. If any resource reports an error, inspect the operator logs:

```bash
kubectl logs -l app.kubernetes.io/name=dex-config-operator
```

## Next steps

- Add more connectors for other identity providers (GitHub, LDAP, SAML)
- Configure additional clients for your applications
- Explore the [Reference](../reference/) section for full CRD specifications
