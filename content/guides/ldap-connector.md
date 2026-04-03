# Configuring an LDAP Connector

This guide explains how to set up an LDAP connector with the Dex Config Operator, enabling authentication against LDAP directories such as OpenLDAP or Active Directory.

## Overview

An LDAP connector requires:

1. A **Secret** containing the LDAP configuration as base64-encoded JSON.
2. A **Connector** custom resource that references the secret.

## Configuration Fields

### Connection

| Field                | Required | Description                                                                              |
|----------------------|----------|------------------------------------------------------------------------------------------|
| `host`               | Yes      | LDAP server host and port (e.g., `ldap.example.com:636`).                                |
| `bindDN`             | Yes      | The Distinguished Name used to bind (authenticate) to the LDAP server.                   |
| `bindPW`             | Yes      | The password for the bind DN.                                                            |
| `insecureNoSSL`      | No       | When `true`, connects over plain LDAP instead of LDAPS. Default: `false`.                |
| `insecureSkipVerify` | No       | When `true`, skips TLS certificate verification. Default: `false`.                       |
| `startTLS`           | No       | When `true`, uses StartTLS to upgrade a plain connection to TLS. Default: `false`.       |
| `rootCA`             | No       | Path to a PEM-encoded root CA certificate for verifying the LDAP server's TLS cert.      |

### User Search

The `userSearch` object configures how Dex finds user entries in the directory.

| Field        | Required | Description                                                                            |
|--------------|----------|----------------------------------------------------------------------------------------|
| `baseDN`     | Yes      | The base DN under which to search for users (e.g., `ou=users,dc=example,dc=com`).      |
| `filter`     | No       | An LDAP filter to apply when searching for users (e.g., `(objectClass=person)`).        |
| `username`   | Yes      | The LDAP attribute that users type as their login username (e.g., `uid` or `mail`).     |
| `idAttr`     | Yes      | The LDAP attribute used as the unique user identifier (e.g., `uid`).                    |
| `emailAttr`  | Yes      | The LDAP attribute containing the user's email address (e.g., `mail`).                  |
| `nameAttr`   | Yes      | The LDAP attribute containing the user's display name (e.g., `cn`).                     |

### Group Search

The `groupSearch` object configures how Dex resolves group membership.

| Field          | Required | Description                                                                          |
|----------------|----------|--------------------------------------------------------------------------------------|
| `baseDN`       | Yes      | The base DN under which to search for groups (e.g., `ou=groups,dc=example,dc=com`).  |
| `filter`       | No       | An LDAP filter for group entries (e.g., `(objectClass=groupOfNames)`).               |
| `nameAttr`     | No       | The LDAP attribute for the group name (e.g., `cn`). Default: `cn`.                   |
| `userMatchers` | Yes      | A list of mappings between user attributes and group member attributes.              |

Each entry in `userMatchers` has:

| Field       | Description                                                                  |
|-------------|------------------------------------------------------------------------------|
| `userAttr`  | The user attribute to match (e.g., `DN`).                                    |
| `groupAttr` | The group attribute that holds member references (e.g., `member`).           |

## Decoded JSON Example

```json
{
  "host": "ldap.example.com:636",
  "bindDN": "cn=serviceaccount,dc=example,dc=com",
  "bindPW": "service-account-password",
  "insecureNoSSL": false,
  "insecureSkipVerify": false,
  "userSearch": {
    "baseDN": "ou=users,dc=example,dc=com",
    "filter": "(objectClass=person)",
    "username": "uid",
    "idAttr": "uid",
    "emailAttr": "mail",
    "nameAttr": "cn"
  },
  "groupSearch": {
    "baseDN": "ou=groups,dc=example,dc=com",
    "filter": "(objectClass=groupOfNames)",
    "nameAttr": "cn",
    "userMatchers": [
      {
        "userAttr": "DN",
        "groupAttr": "member"
      }
    ]
  }
}
```

## Full Example

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ldap-connector-config
type: Opaque
data:
  config: eyJob3N0IjogImxkYXAuZXhhbXBsZS5jb206NjM2IiwgImJpbmRETiI6ICJjbj1zZXJ2aWNlYWNjb3VudCxkYz1leGFtcGxlLGRjPWNvbSIsICJiaW5kUFciOiAic2VydmljZS1hY2NvdW50LXBhc3N3b3JkIiwgImluc2VjdXJlTm9TU0wiOiBmYWxzZSwgImluc2VjdXJlU2tpcFZlcmlmeSI6IGZhbHNlLCAidXNlclNlYXJjaCI6IHsiYmFzZUROIjogIm91PXVzZXJzLGRjPWV4YW1wbGUsZGM9Y29tIiwgImZpbHRlciI6ICIob2JqZWN0Q2xhc3M9cGVyc29uKSIsICJ1c2VybmFtZSI6ICJ1aWQiLCAiaWRBdHRyIjogInVpZCIsICJlbWFpbEF0dHIiOiAibWFpbCIsICJuYW1lQXR0ciI6ICJjbiJ9LCAiZ3JvdXBTZWFyY2giOiB7ImJhc2VETiI6ICJvdT1ncm91cHMsZGM9ZXhhbXBsZSxkYz1jb20iLCAiZmlsdGVyIjogIihvYmplY3RDbGFzcz1ncm91cE9mTmFtZXMpIiwgIm5hbWVBdHRyIjogImNuIiwgInVzZXJNYXRjaGVycyI6IFt7InVzZXJBdHRyIjogIkROIiwgImdyb3VwQXR0ciI6ICJtZW1iZXIifV19fQ==
---
apiVersion: auth.stakater.com/v1alpha1
kind: Connector
metadata:
  name: corporate-ldap
spec:
  type: ldap
  id: corporate-ldap
  name: Corporate LDAP
  configSecretRef:
    name: ldap-connector-config
  enabled: true
```

## Verify

Confirm the connector was created:

```bash
kubectl get connectors
```

## Active Directory Example

When connecting to Active Directory, the configuration differs slightly. Here is a decoded JSON example:

```json
{
  "host": "ad.example.com:636",
  "bindDN": "CN=svc-dex,OU=ServiceAccounts,DC=example,DC=com",
  "bindPW": "service-account-password",
  "insecureNoSSL": false,
  "insecureSkipVerify": false,
  "userSearch": {
    "baseDN": "OU=Employees,DC=example,DC=com",
    "filter": "(objectClass=user)",
    "username": "sAMAccountName",
    "idAttr": "sAMAccountName",
    "emailAttr": "mail",
    "nameAttr": "displayName"
  },
  "groupSearch": {
    "baseDN": "OU=Groups,DC=example,DC=com",
    "filter": "(objectClass=group)",
    "nameAttr": "cn",
    "userMatchers": [
      {
        "userAttr": "DN",
        "groupAttr": "member"
      }
    ]
  }
}
```

## Tips

- Always use LDAPS (port 636) or StartTLS in production. Set `insecureNoSSL` to `true` only during development.
- The `bindDN` account should be a dedicated service account with read-only access to user and group entries.
- Use `filter` to narrow search scope and improve query performance on large directories.
- For Active Directory, use `sAMAccountName` as the `username` attribute; for OpenLDAP, `uid` is typical.
- The `userMatchers` list supports multiple entries if group membership is tracked across different attributes.
- Generate the base64 value with: `echo -n '{"host": "...", ...}' | base64 -w0`
