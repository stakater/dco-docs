# Multi-Environment Connector Configuration

When managing multiple environments (such as development and production), you can store all connector credentials in a single Kubernetes Secret with separate keys for each environment. This simplifies secret management and makes it straightforward to promote configurations between environments.

## Approach

Instead of creating one Secret per environment, create a single Secret containing keys for each environment's credentials. Each `Connector` resource then references the appropriate key from the shared Secret.

## Step 1: Create the Shared Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: oidc-connector-credentials
type: Opaque
stringData:
  dev-client-id: "dex-dev-client"
  dev-client-secret: "dev-secret-value"
  prod-client-id: "dex-prod-client"
  prod-client-secret: "prod-secret-value"
```

Apply the Secret:

```bash
kubectl apply -f oidc-connector-credentials.yaml
```

!!! warning
    Replace the example values with real credentials. Never commit `plaintext` secrets to version control.

## Step 2: Create Environment-Specific Connectors

### Development Connector

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: Connector
metadata:
  name: oidc-dev
spec:
  id: oidc-dev
  name: "Corporate SSO (Dev)"
  type: oidc
  config:
    issuer: https://idp-dev.example.com
    redirectURI: https://dex-dev.example.com/callback
    clientSecretRef:
      name: oidc-connector-credentials
      key: dev-client-secret
    clientIDRef:
      name: oidc-connector-credentials
      key: dev-client-id
    scopes:
      - openid
      - profile
      - email
    insecureSkipEmailVerified: true
  enabled: true
```

### Production Connector

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: Connector
metadata:
  name: oidc-prod
spec:
  id: oidc-prod
  name: "Corporate SSO (Prod)"
  type: oidc
  config:
    issuer: https://idp.example.com
    redirectURI: https://dex.example.com/callback
    clientSecretRef:
      name: oidc-connector-credentials
      key: prod-client-secret
    clientIDRef:
      name: oidc-connector-credentials
      key: prod-client-id
    scopes:
      - openid
      - profile
      - email
    insecureSkipEmailVerified: false
  enabled: true
```

## Complete Manifest

The following manifest combines the shared Secret with both connector configurations:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: oidc-connector-credentials
type: Opaque
stringData:
  dev-client-id: "dex-dev-client"
  dev-client-secret: "dev-secret-value"
  prod-client-id: "dex-prod-client"
  prod-client-secret: "prod-secret-value"
---
apiVersion: auth.stakater.com/v1alpha1
kind: Connector
metadata:
  name: oidc-dev
spec:
  id: oidc-dev
  name: "Corporate SSO (Dev)"
  type: oidc
  config:
    issuer: https://idp-dev.example.com
    redirectURI: https://dex-dev.example.com/callback
    clientSecretRef:
      name: oidc-connector-credentials
      key: dev-client-secret
    clientIDRef:
      name: oidc-connector-credentials
      key: dev-client-id
    scopes:
      - openid
      - profile
      - email
    insecureSkipEmailVerified: true
  enabled: true
---
apiVersion: auth.stakater.com/v1alpha1
kind: Connector
metadata:
  name: oidc-prod
spec:
  id: oidc-prod
  name: "Corporate SSO (Prod)"
  type: oidc
  config:
    issuer: https://idp.example.com
    redirectURI: https://dex.example.com/callback
    clientSecretRef:
      name: oidc-connector-credentials
      key: prod-client-secret
    clientIDRef:
      name: oidc-connector-credentials
      key: prod-client-id
    scopes:
      - openid
      - profile
      - email
    insecureSkipEmailVerified: false
  enabled: true
```

## Benefits

| Benefit | Description |
|---|---|
| Fewer Secrets to manage | One Secret holds credentials for all environments. |
| Consistent naming | Both connectors reference the same Secret, reducing configuration drift. |
| Easier rotation | Update a single Secret when rotating credentials across environments. |
| Clear separation | Each connector explicitly selects its own key, preventing cross-environment mistakes. |

## Considerations

- Ensure RBAC policies allow the operator's service account to read the shared Secret.
- If environments run in separate namespaces, the Secret must exist in the same namespace as each `Connector` resource, or you must use one Secret per namespace.
- Use separate Secrets if your security policy requires strict isolation between environment credentials.
