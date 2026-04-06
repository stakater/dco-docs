# Complete Production Setup

This guide walks through a full end-to-end deployment of Dex using the Dex Config Operator. The example uses in-memory storage, a Keycloak OIDC connector for enterprise SSO, a confidential OAuth2 client, and a local admin user for break-glass access.

## Prerequisites

- A running Kubernetes cluster
- The Dex Config Operator installed
- A Keycloak realm configured with a client for Dex
- `kubectl` access to the cluster

## Complete Manifest

The following manifest contains all resources needed for this setup. Apply it as a single file or split it into separate files as your workflow requires.

```yaml
# ============================================================
# 1. Keycloak OIDC Connector Credentials
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
# 2. Confidential Client Secret
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
# 3. Local Admin User Credentials
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
# 4. DexConfig — Global Configuration
# ============================================================
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
  expiry:
    signingKeys: "6h"
    idTokens: "8h"
  frontend:
    logoURL: "https://cdn.example.com/logo.svg"
    theme: "auto"
---
# ============================================================
# 5. Keycloak OIDC Connector
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
  configSecretRef:
    name: keycloak-oidc-credentials
  enabled: true
---
# ============================================================
# 6. Confidential OAuth2 Client
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
  enabled: true
---
# ============================================================
# 7. Local Admin User (Break-Glass Access)
# ============================================================
apiVersion: auth.stakater.com/v1alpha1
kind: LocalUser
metadata:
  name: admin
  namespace: dex
spec:
  userID: "1"
  credentialSecretRef:
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

```bash
# Verify all custom resources are healthy
kubectl get dexconfig dex-config
kubectl get clients,connectors,localusers -n dex

# Check Dex logs
kubectl logs deployment/dex -n dex

# Confirm OIDC discovery endpoint
curl -s https://dex.example.com/.well-known/openid-configuration | jq .issuer
```

## Generating the Admin Password Hash

Generate a `bcrypt` hash for the admin user and replace the placeholder in the Secret:

```bash
# Using htpasswd
htpasswd -bnBC 10 "" 'your-admin-password' | tr -d ':\n'

# Using Python
python3 -c 'import bcrypt; print(bcrypt.hashpw(b"your-admin-password", bcrypt.gensalt(rounds=10)).decode())'
```

Replace `$2y$10$CHANGE_ME_WITH_REAL_BCRYPT_HASH` in the `admin-user-credentials` Secret with the generated hash.

## Next Steps

- [Configure additional OAuth2 clients](oauth2-clients.md) for other applications.
- [Set up different storage backends](postgresql-storage.md) for production workloads.
- [Tune token expiry](token-expiry.md) based on your security posture.
