# Token Expiry Configuration

The `DexConfig` resource allows fine-grained control over token and request lifetimes through the `expiry` section. Tuning these values lets you balance security requirements against user experience.

## Configuration Overview

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: DexConfig
metadata:
  name: dex-config
spec:
  issuer: https://dex.example.com
  expiry:
    signingKeys: "6h"
    idTokens: "24h"
    authRequests: "24h"
    deviceRequests: "5m"
    refreshTokens:
      validIfNotUsedFor: "168h"
      absoluteLifetime: "720h"
      reuseInterval: "3s"
      disableRotation: false
  storage:
    type: postgres
    configSecretRef:
      name: postgres-credentials
```

## Field Reference

### Top-Level Expiry Fields

| Field | Description | Default |
|---|---|---|
| `expiry.signingKeys` | How often Dex rotates its signing keys. Shorter values improve security; longer values reduce key-rotation overhead. | `6h` |
| `expiry.idTokens` | Lifetime of issued ID tokens. After expiry, clients must use a refresh token to obtain a new ID token. | `24h` |
| `expiry.authRequests` | How long an authorization request remains valid before the user must restart the login flow. | `24h` |
| `expiry.deviceRequests` | How long a device authorization code remains valid in the device code flow. | `5m` |

### Refresh Token Fields

The `expiry.refreshTokens` section controls refresh token behavior:

| Field | Description | Default |
|---|---|---|
| `validIfNotUsedFor` | Refresh token expires if not used within this duration. Set to `"0"` to disable idle expiry. | `168h` (7 days) |
| `absoluteLifetime` | Maximum lifetime of a refresh token regardless of usage. Set to `"0"` for no absolute limit. | `720h` (30 days) |
| `reuseInterval` | Duration during which a refresh token can be reused without being considered a replay attack. Helps with unreliable networks where a client may retry a token refresh. | `3s` |
| `disableRotation` | When `true`, refresh tokens are not rotated on use. The same token is returned on every refresh. When `false`, each refresh returns a new token and the old one is invalidated. | `false` |

!!! warning
    Setting `disableRotation: true` reduces security. A leaked refresh token remains valid until it expires. Only disable rotation if your clients cannot handle rotating tokens.

## Duration Format

All duration fields accept Go-style duration strings:

| Unit | Suffix | Example |
|---|---|---|
| Seconds | `s` | `"30s"` |
| Minutes | `m` | `"5m"` |
| Hours | `h` | `"24h"` |

Units can be combined: `"1h30m"` is valid.

## Example: Short-Lived Tokens for High Security

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: DexConfig
metadata:
  name: dex-config
spec:
  issuer: https://dex.example.com
  expiry:
    signingKeys: "2h"
    idTokens: "1h"
    authRequests: "10m"
    deviceRequests: "3m"
    refreshTokens:
      validIfNotUsedFor: "24h"
      absoluteLifetime: "168h"
      reuseInterval: "3s"
      disableRotation: false
  storage:
    type: postgres
    configSecretRef:
      name: postgres-credentials
```

## Example: Long-Lived Tokens for Internal Tools

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: DexConfig
metadata:
  name: dex-config
spec:
  issuer: https://dex.example.com
  expiry:
    signingKeys: "12h"
    idTokens: "72h"
    authRequests: "48h"
    deviceRequests: "10m"
    refreshTokens:
      validIfNotUsedFor: "720h"
      absoluteLifetime: "2160h"
      reuseInterval: "10s"
      disableRotation: false
  storage:
    type: postgres
    configSecretRef:
      name: postgres-credentials
```

## Verifying Token Lifetimes

After applying the configuration, you can decode an issued ID token to confirm its expiry. Use a tool such as `jwt.io` or the `jq` CLI:

```bash
# Decode the payload of a JWT (second segment)
echo "$ID_TOKEN" | cut -d. -f2 | base64 -d 2>/dev/null | jq '.exp, .iat'
```

The difference between `exp` and `iat` should match the configured `idTokens` duration.
