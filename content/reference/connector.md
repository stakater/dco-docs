# Connector CRD Reference

## Overview

| Property | Value |
|---|---|
| **API Group** | `auth.stakater.com` |
| **API Version** | `v1alpha1` |
| **Kind** | `Connector` |
| **Scope** | `Namespaced` |

---

## Spec Fields

| Field | Type | Required | Default | Validation | Description |
|---|---|---|---|---|---|
| `type` | `string` | Yes | - | `Enum`: `oidc`, `google`, `saml`, `ldap`, `github`, `gitlab`, `bitbucket`, `microsoft` | The type of identity provider connector. |
| `id` | `string` | Yes | - | Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` | A unique identifier for this connector. Must be a DNS-compatible lowercase string. |
| `name` | `string` | Yes | - | - | A human-readable display name shown on the Dex login page. |
| `configSecretRef` | `object` | Yes | - | - | Reference to a Secret containing the connector configuration. See [Config Secret Reference](#config-secret-reference). |
| `enabled` | `bool` | No | `true` | - | When set to `false`, the connector is not registered with Dex. |

### Config Secret Reference

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `configSecretRef.name` | `string` | Yes | - | Name of the Secret containing the connector configuration. |
| `configSecretRef.namespace` | `string` | Yes | - | Namespace of the Secret. |
| `configSecretRef.key` | `string` | No | `config` | Key within the Secret data that holds the connector configuration JSON/YAML. |

---

## Status Fields

| Field | Type | Description |
|---|---|---|
| `conditions` | `[]Condition` | Standard Kubernetes conditions representing the state of the resource. |
| `observedGeneration` | `int64` | The last `.metadata.generation` observed by the controller. |
| `phase` | `string` | The current phase of the Connector (e.g., `Active`, `Failed`). |
| `message` | `string` | A human-readable message describing the current state or error. |
| `lastUpdated` | `string` | Timestamp of the last status update. |

---

## Connector Type Configuration Reference

Each connector type requires specific fields in the Secret referenced by `configSecretRef`. The table below lists the required configuration fields per connector type.

### OIDC

| Field | Required | Description |
|---|---|---|
| `issuer` | Yes | The OIDC provider issuer URL. |
| `clientID` | Yes | OAuth2 client ID. |
| `clientSecret` | Yes | OAuth2 client secret. |
| `redirectURI` | Yes | The redirect URI registered with the OIDC provider. |

### Google

| Field | Required | Description |
|---|---|---|
| `clientID` | Yes | Google OAuth2 client ID. |
| `clientSecret` | Yes | Google OAuth2 client secret. |
| `redirectURI` | Yes | The redirect URI registered with Google. |
| `serviceAccountFilePath` | No | Path to a Google service account JSON key file (required for group sync). |
| `adminEmail` | No | Google Workspace admin email (required for group sync). |

### SAML

| Field | Required | Description |
|---|---|---|
| `ssoURL` | Yes | The SAML Single Sign-On URL of the IdP. |
| `ca` | Yes | Path or content of the CA certificate to validate the IdP's SAML response. |
| `redirectURI` | Yes | The redirect URI (Assertion Consumer Service URL). |
| `entityIssuer` | No | The Issuer value expected in the SAML assertion. |
| `nameIDPolicyFormat` | No | The NameID format requested (e.g., `persistent`, `emailAddress`). |

### LDAP

| Field | Required | Description |
|---|---|---|
| `host` | Yes | LDAP server host and port (e.g., `ldap.example.com:636`). |
| `bindDN` | Yes | Distinguished name for the bind user. |
| `bindPW` | Yes | Password for the bind user. |
| `userSearch` | Yes | User search configuration (baseDN, filter, username, `idAttr`, `emailAttr`, `nameAttr`). |
| `groupSearch` | No | Group search configuration (baseDN, filter, `userMatchers`, `nameAttr`). |
| `rootCA` | No | Path or content of the CA certificate for LDAPS connections. |
| `insecureNoSSL` | No | If `true`, connects to the LDAP server without TLS. |
| `insecureSkipVerify` | No | If `true`, skips TLS certificate verification. |

### GitHub

| Field | Required | Description |
|---|---|---|
| `clientID` | Yes | GitHub OAuth App client ID. |
| `clientSecret` | Yes | GitHub OAuth App client secret. |
| `redirectURI` | Yes | The redirect URI registered with GitHub. |
| `orgs` | No | List of GitHub organizations to restrict access to. |
| `hostName` | No | Hostname for GitHub Enterprise (omit for GitHub.com). |

### GitLab

| Field | Required | Description |
|---|---|---|
| `clientID` | Yes | GitLab OAuth Application ID. |
| `clientSecret` | Yes | GitLab OAuth Application secret. |
| `redirectURI` | Yes | The redirect URI registered with GitLab. |
| `baseURL` | No | Base URL for self-hosted GitLab instances (omit for GitLab.com). |
| `groups` | No | List of GitLab groups to restrict access to. |

### Bitbucket

| Field | Required | Description |
|---|---|---|
| `clientID` | Yes | Bitbucket OAuth consumer key. |
| `clientSecret` | Yes | Bitbucket OAuth consumer secret. |
| `redirectURI` | Yes | The redirect URI registered with Bitbucket. |
| `teams` | No | List of Bitbucket teams to restrict access to. |

### Microsoft

| Field | Required | Description |
|---|---|---|
| `clientID` | Yes | Microsoft (Azure AD) Application client ID. |
| `clientSecret` | Yes | Microsoft (Azure AD) Application client secret. |
| `redirectURI` | Yes | The redirect URI registered with Azure AD. |
| `tenant` | Yes | Azure AD tenant ID or name (e.g., `common`, `organizations`, or a specific tenant ID). |
| `groups` | No | List of Azure AD group IDs to restrict access to. |
| `onlySecurityGroups` | No | If `true`, only fetches security groups from Azure AD. |
