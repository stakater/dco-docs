# Configuring a SAML Connector

This guide explains how to set up a SAML 2.0 connector with the Dex Config Operator, enabling authentication against SAML-based identity providers such as ADFS, Shibboleth, or any SAML 2.0 compliant IdP.

## Overview

A SAML connector requires:

1. A **Secret** containing the SAML configuration as base64-encoded JSON.
1. A **Connector** custom resource that references the secret.

## Configuration Fields

| Field                 | Required | Description                                                                                     |
|-----------------------|----------|-------------------------------------------------------------------------------------------------|
| `ssoURL`              | Yes      | The IdP's Single Sign-On URL where Dex sends SAML authentication requests.                      |
| `ca`                  | Yes      | The PEM-encoded CA certificate (or certificate itself) used to verify the IdP's SAML responses. |
| `redirectURI`         | Yes      | The Dex callback URL that the IdP redirects to after authentication.                            |
| `entityIssuer`        | No       | The entity ID that Dex uses to identify itself to the IdP. Defaults to the redirect URI.        |
| `ssoIssuer`           | No       | The expected issuer value in SAML responses from the IdP.                                       |
| `usernameAttr`        | Yes      | The SAML attribute that maps to the user's display name.                                        |
| `emailAttr`           | Yes      | The SAML attribute that maps to the user's email address.                                       |
| `groupsAttr`          | No       | The SAML attribute that maps to group membership.                                               |
| `nameIDPolicyFormat`  | No       | The NameID policy format to request. Common values listed below.                                |
| `insecureSkipSignatureValidation` | No | When `true`, skips signature validation. Use only for debugging.                      |

### Common `nameIDPolicyFormat` Values

| Format                                                           | Description       |
|------------------------------------------------------------------|-------------------|
| `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`        | Email address      |
| `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`         | Unspecified        |
| `urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`          | Persistent ID      |
| `urn:oasis:names:tc:SAML:2.0:nameid-format:transient`           | Transient ID       |

## Decoded JSON Example

```json
{
  "ssoURL": "https://idp.example.com/saml/sso",
  "ca": "/etc/dex/saml-ca.pem",
  "redirectURI": "https://dex.example.com/callback",
  "entityIssuer": "https://dex.example.com",
  "usernameAttr": "displayName",
  "emailAttr": "email",
  "groupsAttr": "groups",
  "nameIDPolicyFormat": "urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"
}
```

## Full Example

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: saml-connector-config
type: Opaque
data:
  config: eyJzc29VUkwiOiAiaHR0cHM6Ly9pZHAuZXhhbXBsZS5jb20vc2FtbC9zc28iLCAiY2EiOiAiL2V0Yy9kZXgvc2FtbC1jYS5wZW0iLCAicmVkaXJlY3RVUkkiOiAiaHR0cHM6Ly9kZXguZXhhbXBsZS5jb20vY2FsbGJhY2siLCAiZW50aXR5SXNzdWVyIjogImh0dHBzOi8vZGV4LmV4YW1wbGUuY29tIiwgInVzZXJuYW1lQXR0ciI6ICJkaXNwbGF5TmFtZSIsICJlbWFpbEF0dHIiOiAiZW1haWwiLCAiZ3JvdXBzQXR0ciI6ICJncm91cHMiLCAibmFtZUlEUG9saWN5Rm9ybWF0IjogInVybjpvYXNpczpuYW1lczp0YzpTQU1MOjIuMDpuYW1laWQtZm9ybWF0OnBlcnNpc3RlbnQifQ==
---
apiVersion: auth.stakater.com/v1alpha1
kind: Connector
metadata:
  name: corporate-saml
spec:
  type: saml
  id: corporate-saml
  name: Corporate SAML
  configSecretRef:
    name: saml-connector-config
  enabled: true
```

## Verify

Confirm the connector was created:

```bash
kubectl get connectors
```

## IdP Configuration

When configuring your Identity Provider, you will need to supply:

- **ACS URL (Assertion Consumer Service):** This is the Dex callback URL, e.g., `https://dex.example.com/callback`.
- **Entity ID / Audience:** This should match the `entityIssuer` value in your configuration.
- **Attribute mappings:** Ensure the IdP sends the attributes that match your `usernameAttr`, `emailAttr`, and `groupsAttr` values.

## Tips

- The `ca` field can be either a file path to a PEM certificate mounted into the Dex pod, or the PEM-encoded certificate content itself (inline).
- If you are using ADFS, the `nameIDPolicyFormat` is typically `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`.
- Set `groupsDelim` in the JSON if your IdP returns groups as a single delimited string rather than multiple attribute values.
- Always test with `insecureSkipSignatureValidation: false` (the default) in production to ensure SAML response integrity.
