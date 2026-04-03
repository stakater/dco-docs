# Configuring an OIDC Connector

This guide walks through setting up an OpenID Connect (OIDC) connector with the Dex Config Operator. OIDC is the most common connector type and works with providers such as Keycloak, Okta, Auth0, and any standards-compliant identity provider.

## Overview

An OIDC connector requires two Kubernetes resources:

1. A **Secret** containing the provider configuration as base64-encoded JSON.
2. A **Connector** custom resource that references the secret.

## Step 1: Prepare the Configuration Secret

The secret must contain a JSON object with your OIDC provider details. Below is the decoded JSON structure for reference:

```json
{
  "issuer": "https://keycloak.example.com/realms/myrealm",
  "clientID": "dex-client",
  "clientSecret": "your-secret",
  "redirectURI": "https://dex.example.com/callback",
  "scopes": ["openid", "profile", "email", "groups"]
}
```

| Field          | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| `issuer`       | The OIDC issuer URL. Dex uses this to discover provider endpoints.          |
| `clientID`     | The OAuth 2.0 client ID registered with your provider.                      |
| `clientSecret` | The OAuth 2.0 client secret associated with the client ID.                  |
| `redirectURI`  | The callback URL that your provider will redirect to after authentication.  |
| `scopes`       | The OIDC scopes to request. Must include `openid`; add others as needed.    |

Base64-encode the JSON and store it in a Kubernetes Secret under the `config` key.

## Step 2: Create the Secret and Connector

Apply the following manifests to your cluster:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: keycloak-connector-config
type: Opaque
data:
  config: eyJpc3N1ZXIiOiAiaHR0cHM6Ly9rZXljbG9hay5leGFtcGxlLmNvbS9yZWFsbXMvbXlyZWFsbSIsICJjbGllbnRJRCI6ICJkZXgtY2xpZW50IiwgImNsaWVudFNlY3JldCI6ICJ5b3VyLXNlY3JldCIsICJyZWRpcmVjdFVSSSI6ICJodHRwczovL2RleC5leGFtcGxlLmNvbS9jYWxsYmFjayIsICJzY29wZXMiOiBbIm9wZW5pZCIsICJwcm9maWxlIiwgImVtYWlsIiwgImdyb3VwcyJdfQ==
---
apiVersion: auth.stakater.com/v1alpha1
kind: Connector
metadata:
  name: keycloak
spec:
  type: oidc
  id: keycloak
  name: Corporate SSO
  configSecretRef:
    name: keycloak-connector-config
  enabled: true
```

## Step 3: Verify

Confirm the connector was created successfully:

```bash
kubectl get connectors
```

The connector should appear with its name and type. Dex will pick up the configuration and present the OIDC login option on its login page.

## Understanding `configSecretRef`

The `configSecretRef` field tells the operator which Secret holds the provider configuration.

| Sub-field | Required | Default    | Description                                              |
|-----------|----------|------------|----------------------------------------------------------|
| `name`    | Yes      | --         | The name of the Kubernetes Secret.                       |
| `key`     | No       | `"config"` | The key inside the Secret that contains the JSON config. |

If your Secret uses a key other than `config`, specify it explicitly:

```yaml
configSecretRef:
  name: keycloak-connector-config
  key: my-custom-key
```

When the `key` field is omitted, the operator defaults to reading from the `config` key in the referenced Secret.

## Tips

- Generate the base64 value with: `echo -n '{"issuer": "...", ...}' | base64 -w0`
- The `redirectURI` must match exactly what is configured in your identity provider.
- Add `"insecureSkipEmailVerified": true` to the JSON if your provider does not supply the `email_verified` claim.
- Add `"getUserInfo": true` if you need claims that are only available from the UserInfo endpoint.
