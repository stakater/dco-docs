# Configuring OAuth2 Clients

The Dex Config Operator manages OAuth2 clients through the `Client` custom resource. Clients fall into two categories — public and confidential — depending on whether the application can securely store a secret.

## Public Clients

Public clients are applications that cannot keep a client secret confidential. This includes single-page applications (SPAs), mobile apps, and desktop applications. These clients rely on PKCE (Proof Key for Code Exchange) for security instead of a shared secret.

Key characteristics:

- `public: true`
- No `secretRef` required
- Typically use `http://localhost` redirect URIs during development

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: Client
metadata:
  name: spa-app
spec:
  id: spa-app
  name: "Single Page Application"
  redirectURIs:
    - "http://localhost:3000/callback"
  public: true
  enabled: true
```

Apply the resource:

```bash
kubectl apply -f spa-app-client.yaml
```

## Confidential Clients

Confidential clients are server-side applications that can securely store a client secret. The secret is stored in a Kubernetes Secret and referenced by the `Client` resource.

Key characteristics:

- `public: false`
- `secretRef` is required, pointing to a Kubernetes Secret
- Suitable for backend services, APIs, and server-rendered web applications

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: Client
metadata:
  name: backend-app
spec:
  id: backend-app
  name: "Backend Application"
  redirectURIs:
    - "https://app.example.com/callback"
  public: false
  secretRef:
    name: backend-app-secret
    key: secret
  trustedPeers: ["spa-app"]
  logoURL: "https://app.example.com/logo.png"
  enabled: true
---
apiVersion: v1
kind: Secret
metadata:
  name: backend-app-secret
type: Opaque
stringData:
  secret: "your-client-secret"
```

Apply the resources:

```bash
kubectl apply -f backend-app-client.yaml
```

!!! warning
    Replace the example secret value with a strong, randomly generated string. Never commit secrets in plain text to version control.

## Trusted Peers

The `trustedPeers` field allows a confidential client to share tokens with other clients. This is useful when a backend application issues tokens that a frontend SPA also needs to consume.

In the example above, `backend-app` trusts `spa-app`, meaning tokens issued to `backend-app` can be used by `spa-app` through cross-client token exchange.

```yaml
spec:
  trustedPeers: ["spa-app", "cli-tool"]
```

Trusted peers are referenced by their `id`, not their Kubernetes resource name (though these are often the same).

## Field Reference

| Field | Description |
|---|---|
| `spec.id` | Unique identifier for the client. Must be stable across updates. |
| `spec.name` | Human-readable display name shown on the consent screen. |
| `spec.redirectURIs` | List of allowed redirect URIs after authentication. |
| `spec.public` | Set to `true` for public clients, `false` for confidential clients. |
| `spec.secretRef.name` | Name of the Kubernetes Secret containing the client secret. |
| `spec.secretRef.key` | Key within the Secret that holds the client secret value. |
| `spec.trustedPeers` | List of client IDs that are allowed to exchange tokens with this client. |
| `spec.logoURL` | URL to a logo image displayed on the consent screen. |
| `spec.enabled` | Set to `true` to activate the client, `false` to disable it. |

## Disabling a Client

To temporarily disable a client without deleting it, set `enabled: false`:

```yaml
spec:
  enabled: false
```

The client will remain in the cluster but Dex will reject authentication requests for it.
