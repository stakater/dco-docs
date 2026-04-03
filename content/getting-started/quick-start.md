# Quick Start

This guide takes you from a fresh operator deployment to a fully rendered Dex configuration. By the end you will have a working DexConfig, a Google OAuth Connector, and a Client -- all managed declaratively.

## Prerequisites

- A running Kubernetes cluster with DCO installed (see [Installation](installation.md))
- `kubectl` configured to communicate with the cluster

## 1. Install CRDs

If you have not already installed the Custom Resource Definitions:

```bash
make install
```

## 2. Deploy the operator

Deploy using any method described in the [Installation](installation.md) guide. For example, with raw manifests:

```bash
kubectl apply -f dist/install.yaml
```

## 3. Create a DexConfig resource

The DexConfig resource defines the core Dex server settings -- issuer URL, storage backend, web listener, and OAuth2 behaviour.

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

## 4. Create a Connector

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
  type: oidc
  id: google
  name: Google
  config:
    issuer: https://accounts.google.com
    redirectURI: https://dex.example.com/callback
  secretRef:
    name: google-connector-secret
    namespace: default
```

Apply it:

```bash
kubectl apply -f google-connector.yaml
```

## 5. Create a Client

Clients represent applications that authenticate through Dex. Save the following as `example-client.yaml`:

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: Client
metadata:
  name: example-app
spec:
  id: example-app
  name: Example Application
  secretRef:
    name: example-app-secret
    namespace: default
  redirectURIs:
    - https://app.example.com/callback
```

Create the corresponding client secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-app-secret
type: Opaque
stringData:
  clientSecret: "example-app-secret-value"
```

Apply both:

```bash
kubectl apply -f example-app-secret.yaml
kubectl apply -f example-client.yaml
```

## 6. Verify the generated configuration

DCO assembles all resources into a single Dex configuration Secret. Inspect it:

```bash
kubectl get secret dex -n dex -o jsonpath='{.data.config\.yaml}' | base64 -d
```

The output should include your issuer, storage settings, the Google connector, and the example client.

## 7. Check resource statuses

Confirm that each custom resource has been reconciled successfully:

```bash
kubectl get dexconfig
kubectl get connectors
kubectl get clients
```

All resources should show a status indicating they have been processed by the operator. If any resource reports an error, inspect the operator logs:

```bash
kubectl logs -l control-plane=controller-manager -n dex-config-operator-system
```

## Next steps

- Add more connectors for other identity providers (GitHub, LDAP, SAML)
- Configure additional clients for your applications
- Explore the [Reference](../reference/) section for full CRD specifications
