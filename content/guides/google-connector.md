# Configuring a Google OAuth Connector

This guide explains how to set up a Google OAuth connector with the Dex Config Operator so that users can authenticate using their Google accounts.

## Prerequisites

1. A Google Cloud project with the **OAuth consent screen** configured.
2. An OAuth 2.0 Client ID created under **APIs & Services > Credentials** in the Google Cloud Console.
3. The authorized redirect URI set to your Dex callback URL (e.g., `https://dex.example.com/callback`).

## Configuration Secret

The secret contains a JSON object with the required Google OAuth fields:

```json
{
  "clientID": "your-google-client-id",
  "clientSecret": "your-secret",
  "redirectURI": "https://dex.example.com/callback"
}
```

| Field          | Required | Description                                                                |
|----------------|----------|----------------------------------------------------------------------------|
| `clientID`     | Yes      | The OAuth 2.0 client ID from the Google Cloud Console.                     |
| `clientSecret` | Yes      | The OAuth 2.0 client secret from the Google Cloud Console.                 |
| `redirectURI`  | Yes      | The callback URL configured in Google and matching your Dex deployment.    |

## Full Example

Apply the following manifests to create both the Secret and the Connector:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: google-config
type: Opaque
data:
  config: eyJjbGllbnRJRCI6ICJ5b3VyLWdvb2dsZS1jbGllbnQtaWQiLCAiY2xpZW50U2VjcmV0IjogInlvdXItc2VjcmV0IiwgInJlZGlyZWN0VVJJIjogImh0dHBzOi8vZGV4LmV4YW1wbGUuY29tL2NhbGxiYWNrIn0=
---
apiVersion: auth.stakater.com/v1alpha1
kind: Connector
metadata:
  name: google
spec:
  type: google
  id: google
  name: Google
  configSecretRef:
    name: google-config
  enabled: true
```

## Verify

Check that the connector was created:

```bash
kubectl get connectors
```

Once active, a "Log in with Google" option will appear on the Dex login page.

## Optional Fields

The Google connector JSON also supports these optional fields:

| Field              | Description                                                                                  |
|--------------------|----------------------------------------------------------------------------------------------|
| `hostedDomains`    | A list of allowed Google Workspace domains (e.g., `["example.com"]`). Restricts sign-in.     |
| `serviceAccountFilePath` | Path to a Google service account JSON key file, required for fetching group membership. |
| `adminEmail`       | A Workspace admin email, required when using service account-based group fetching.           |
| `fetchTransitiveGroupMembership` | When `true`, resolves nested group memberships.                                |

### Example with Domain Restriction

Decoded JSON:

```json
{
  "clientID": "your-google-client-id",
  "clientSecret": "your-secret",
  "redirectURI": "https://dex.example.com/callback",
  "hostedDomains": ["example.com"]
}
```

Base64-encode this JSON and place it in the Secret's `config` key as shown above.

## Tips

- The `redirectURI` must exactly match one of the authorized redirect URIs in your Google Cloud OAuth client configuration.
- To restrict login to specific domains, always use the `hostedDomains` field rather than relying on post-authentication checks.
- Group fetching requires a Google Workspace service account with domain-wide delegation and the `admin.directory.group.readonly` scope.
