# Operator Configuration Reference

This page documents every tuneable for the Dex Config Operator — command-line flags, environment variables, and Helm chart values.

## Command-Line Flags

The operator binary accepts the following flags. Each flag can also be set through its corresponding environment variable where noted.

| Flag | Default | `Env` Variable | Description |
|---|---|---|---|
| `--dex-namespace` | `dex` | `DEX_NAMESPACE` | Namespace where the Dex Deployment lives. |
| `--dex-deployment-name` | `dex` | `DEX_DEPLOYMENT` | Name of the Dex Deployment resource. |
| `--config-secret-name` | `dex-config` | `DEX_CONFIG_SECRET` | Name of the Secret that stores the generated Dex configuration. |
| `--config-secret-namespace` | *(same as `--dex-namespace`)* | — | Namespace for the generated config Secret. Defaults to the value of `--dex-namespace` when omitted. |
| `--watch-namespace` | `""` (all namespaces) | — | Restrict CRD watches to a single namespace. An empty string watches all namespaces. |
| `--leader-elect` | `false` | — | Enable leader election so only one replica reconciles at a time. Required for high-availability deployments. |
| `--health-probe-bind-address` | `:8081` | — | Address the health-probe endpoint binds to (`/healthz` and `/readyz`). |
| `--metrics-bind-address` | `0` | — | Address the Prometheus metrics endpoint binds to. `0` disables metrics. |
| `--metrics-secure` | `true` | — | Serve the metrics endpoint over https instead of plain http. |
| `--enable-http2` | `false` | — | Enable http/2 on the webhook and metrics servers. Disabled by default to mitigate http/2-specific vulnerabilities. |

## Environment Variables

Environment variables offer an alternative to flags and take precedence when both are set.

| Variable | Description |
|---|---|
| `DEX_NAMESPACE` | Equivalent to `--dex-namespace`. Namespace containing the Dex Deployment. |
| `DEX_DEPLOYMENT` | Equivalent to `--dex-deployment-name`. Name of the Dex Deployment. |
| `DEX_CONFIG_SECRET` | Equivalent to `--config-secret-name`. Name of the generated config Secret. |
| `SECRET_CHANGE_ACTION` | Strategy the operator uses when the config Secret changes. Accepted values: `PatchDeployment` (default) — patches the Deployment to trigger a rolling restart; `DeleteDeployment` — deletes the Deployment and lets the parent controller recreate it. |

### SECRET_CHANGE_ACTION Strategies

`PatchDeployment`
:   Adds or updates an annotation on the Dex Deployment's pod template, which triggers a rolling restart. This is the default and recommended strategy.

`DeleteDeployment`
:   Deletes the Dex Deployment entirely. The owning controller (e.g., a Helm release or GitOps tool) is expected to recreate it. Use this only when the deployment controller cannot detect annotation-based changes.

## Helm Chart Values

When deploying through the Stakater Helm chart, the following values map to the flags and environment variables above.

| Value | Default | Description |
|---|---|---|
| `image.repository` | `ghcr.io/stakater/dex-config-operator` | Container image repository. |
| `image.tag` | *(chart `appVersion`)* | Container image tag. Defaults to the version bundled with the chart. |
| `resources` | `{}` | CPU and memory requests/limits for the operator pod. |
| `env.dexConfigSecret` | `dex-config` | Maps to `DEX_CONFIG_SECRET`. |
| `env.dexDeployment` | `dex` | Maps to `DEX_DEPLOYMENT`. |
| `env.dexNamespace` | `dex` | Maps to `DEX_NAMESPACE`. |
| `env.secretChangeAction` | `PatchDeployment` | Maps to `SECRET_CHANGE_ACTION`. |

### Example `values.yaml`

```yaml
image:
  repository: ghcr.io/stakater/dex-config-operator
  tag: "v0.1.0"

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi

env:
  dexNamespace: auth
  dexDeployment: dex-server
  dexConfigSecret: dex-generated-config
  secretChangeAction: PatchDeployment
```
