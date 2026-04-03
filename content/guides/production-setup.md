# Complete Production Setup

This guide walks through a full end-to-end production deployment of Dex using the Dex Config Operator. The example configures PostgreSQL storage, a Keycloak OIDC connector for enterprise SSO, a confidential OAuth2 client, and a local admin user for break-glass access.

## Prerequisites

- A running Kubernetes cluster
- The Dex Config Operator installed
- A PostgreSQL instance accessible from the cluster
- A Keycloak realm configured with a client for Dex
- `kubectl` access to the cluster

## Architecture Overview

```text
┌─────────────┐     ┌──────────────┐     ┌────────────┐
│  End Users   │────▶│  Dex (OIDC)  │────▶│  Keycloak  │
└─────────────┘     └──────┬───────┘     └────────────┘
                           │
                    ┌──────┴───────┐
                    │  PostgreSQL  │
                    └──────────────┘
```

- **Keycloak** provides enterprise identity federation (SAML, LDAP, social logins).
- **Dex** acts as a lightweight OIDC proxy, unifying authentication for your applications.
- **PostgreSQL** stores Dex state (tokens, keys, auth requests).
- **Local admin user** provides emergency access if the upstream IdP is unavailable.

## Complete Manifest

The following manifest contains all resources needed for this setup. Apply it as a single file or split it into separate files as your workflow requires.

```yaml
# ============================================================
# 1. PostgreSQL Storage Credentials
# ============================================================
apiVersion: v1
kind: Secret
metadata:
  name: postgres-credentials
  namespace: dex
type: Opaque
stringData:
  POSTGRESQL_DATABASE: dex
  POSTGRESQL_USER: dexuser
  POSTGRESQL_PASSWORD: "CHANGE-ME-strong-random-password"
  POSTGRESQL_PORT: "5432"
  POSTGRESQL_SERVICE: postgres.database.svc.cluster.local
  POSTGRESQL_SSL: require
---
# ============================================================
# 2. Keycloak OIDC Connector Credentials
# ============================================================
apiVersion: v1
kind: Secret
metadata:
  name: keycloak-oidc-credentials
  namespace: dex
type: Opaque
stringData:
  client-id: "dex-sso"
  client-secret: "CHANGE-ME-keycloak-client-secret"
---
# ============================================================
# 3. Confidential Client Secret
# ============================================================
apiVersion: v1
kind: Secret
metadata:
  name: app-client-secret
  namespace: dex
type: Opaque
stringData:
  secret: "CHANGE-ME-random-client-secret"
---
# ============================================================
# 4. Local Admin User Credentials
# ============================================================
apiVersion: v1
kind: Secret
metadata:
  name: admin-user-credentials
  namespace: dex
type: Opaque
stringData:
  credentials: |
    {
      "username": "admin",
      "email": "admin@example.com",
      "hash": "$2y$10$CHANGE_ME_WITH_REAL_BCRYPT_HASH",
      "groups": ["admins", "platform-team"]
    }
---
# ============================================================
# 5. DexConfig -- Global Configuration
# ============================================================
apiVersion: auth.stakater.com/v1alpha1
kind: DexConfig
metadata:
  name: dex-config
spec:
  issuer: https://dex.example.com
  storage:
    type: postgres
    configSecretRef:
      name: postgres-credentials
  expiry:
    signingKeys: "6h"
    idTokens: "8h"
    authRequests: "15m"
    deviceRequests: "5m"
    refreshTokens:
      validIfNotUsedFor: "168h"
      absoluteLifetime: "720h"
      reuseInterval: "3s"
      disableRotation: false
  frontend:
    logoURL: "https://cdn.example.com/logo.svg"
    theme: "auto"
---
# ============================================================
# 6. Keycloak OIDC Connector
# ============================================================
apiVersion: auth.stakater.com/v1alpha1
kind: Connector
metadata:
  name: keycloak
  namespace: dex
spec:
  id: keycloak
  name: "Corporate SSO"
  type: oidc
  config:
    issuer: https://keycloak.example.com/realms/corporate
    redirectURI: https://dex.example.com/callback
    clientIDRef:
      name: keycloak-oidc-credentials
      key: client-id
    clientSecretRef:
      name: keycloak-oidc-credentials
      key: client-secret
    scopes:
      - openid
      - profile
      - email
      - groups
    getUserInfo: true
    insecureSkipEmailVerified: false
    claimMapping:
      groups: groups
  enabled: true
---
# ============================================================
# 7. Confidential OAuth2 Client
# ============================================================
apiVersion: auth.stakater.com/v1alpha1
kind: Client
metadata:
  name: internal-platform
  namespace: dex
spec:
  id: internal-platform
  name: "Internal Platform"
  redirectURIs:
    - "https://platform.example.com/oauth2/callback"
  public: false
  secretRef:
    name: app-client-secret
    key: secret
  logoURL: "https://cdn.example.com/platform-logo.svg"
  enabled: true
---
# ============================================================
# 8. Local Admin User (Break-Glass Access)
# ============================================================
apiVersion: auth.stakater.com/v1alpha1
kind: LocalUser
metadata:
  name: admin
  namespace: dex
spec:
  secretRef:
    name: admin-user-credentials
    key: credentials
  enabled: true
```

## Applying the Manifest

```bash
# Create the namespace if it does not exist
kubectl create namespace dex --dry-run=client -o yaml | kubectl apply -f -

# Apply all resources
kubectl apply -f production-dex-setup.yaml
```

## Post-Deployment Verification

Run the following checks to confirm everything is working:

```bash
# Verify the DexConfig is reconciled
kubectl get dexconfig dex-config

# Verify all custom resources are created
kubectl get clients,connectors,localusers -n dex

# Check the Dex deployment logs for errors
kubectl logs deployment/dex -n dex

# Confirm the OIDC discovery endpoint is reachable
curl -s https://dex.example.com/.well-known/openid-configuration | jq .issuer
```

## Generating the Admin Password Hash

Before applying the manifest, generate a `bcrypt` hash for the admin user and replace the placeholder in the Secret:

```bash
# Using htpasswd
htpasswd -bnBC 10 "" 'your-admin-password' | tr -d ':\n'

# Using Python
python3 -c 'import bcrypt; print(bcrypt.hashpw(b"your-admin-password", bcrypt.gensalt(rounds=10)).decode())'
```

Replace `$2y$10$CHANGE_ME_WITH_REAL_BCRYPT_HASH` in the `admin-user-credentials` Secret with the generated hash.

## Security Checklist

Before going to production, verify the following:

- [ ] All `CHANGE-ME` placeholder values have been replaced with real credentials.
- [ ] PostgreSQL SSL mode is set to `require` or `verify-full`.
- [ ] The Keycloak client is configured as confidential with the correct redirect URI.
- [ ] The admin user password is strong and the `bcrypt` hash uses a cost factor of at least 10.
- [ ] Secrets are managed through a secrets management tool (Sealed Secrets, External Secrets, or Vault).
- [ ] TLS is terminated at the ingress in front of Dex.
- [ ] RBAC restricts who can read Secrets in the `dex` namespace.
- [ ] Token expiry values are appropriate for your security requirements.
- [ ] The `internal-platform` client redirect URI uses https.

## Next Steps

- [Configure additional OAuth2 clients](oauth2-clients.md) for other applications.
- [Add more connectors](multi-env-connectors.md) for additional identity providers.
- [Customize the frontend](frontend-customization.md) to match your organization's branding.
- [Tune token expiry](token-expiry.md) based on your security posture.
