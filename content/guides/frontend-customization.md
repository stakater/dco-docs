# Frontend Customization

The `DexConfig` resource exposes frontend settings that control the appearance of the Dex login and consent screens. These settings let you match the Dex UI to your organization's branding.

## Configuration Options

| Field | Description | Example |
|---|---|---|
| `issuer` | Display name shown on the login screen. | `"My Organization"` |
| `frontend.logoURL` | URL to a logo image displayed on the login page. | `"https://example.com/logo.png"` |
| `frontend.theme` | Visual theme for the UI. | `"light"`, `"dark"`, or `"auto"` |
| `frontend.dir` | Text direction for right-to-left language support. | `"ltr"` or `"rtl"` |
| `frontend.extra` | A map of arbitrary key-value pairs passed to custom templates. | `{"company": "Acme Corp"}` |

## Basic Example

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: DexConfig
metadata:
  name: dex-config
spec:
  issuer: "My Organization"
  frontend:
    logoURL: "https://cdn.example.com/logo.png"
    theme: "auto"
```

## Full Example

The following configuration demonstrates all available frontend fields:

```yaml
apiVersion: auth.stakater.com/v1alpha1
kind: DexConfig
metadata:
  name: dex-config
spec:
  issuer: "Acme Corp Identity Platform"
  frontend:
    logoURL: "https://cdn.acme.com/branding/logo.svg"
    theme: "dark"
    dir: "ltr"
    extra:
      company: "Acme Corp"
      supportURL: "https://support.acme.com"
      environment: "production"
  storage:
    type: postgres
    configSecretRef:
      name: postgres-credentials
```

Apply the configuration:

```bash
kubectl apply -f dex-config.yaml
```

## Theme Options

| Theme | Behavior |
|---|---|
| `light` | Always uses the light color scheme. |
| `dark` | Always uses the dark color scheme. |
| `auto` | Follows the user's operating system or browser preference. |

If no theme is specified, Dex uses its default theme.

## Right-to-Left Support

For languages such as Arabic or Hebrew, set `dir` to `"rtl"`:

```yaml
spec:
  frontend:
    dir: "rtl"
```

## Custom Template Data

The `extra` field accepts arbitrary string key-value pairs. These values are available to custom Dex templates if you have configured a custom web directory in the Dex deployment:

```yaml
spec:
  frontend:
    extra:
      company: "Acme Corp"
      helpEmail: "help@acme.com"
      footerText: "Powered by Acme Identity"
```

## Issuer Display Name

The `issuer` field at the top level of `spec` serves two purposes:

1. It sets the OIDC issuer URL that appears in discovery documents and tokens.
1. It provides the display name shown on the login screen.

```yaml
spec:
  issuer: "https://dex.example.com"
```

!!! note
    The `issuer` value must be a valid URL when used for OIDC discovery. The display name shown on the UI is derived from this value by Dex.
