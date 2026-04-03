# Disabling Automatic Dex Restart

By default, the Dex Config Operator automatically restarts the Dex deployment whenever the generated configuration changes. This ensures Dex always runs with the latest settings. In some environments, you may want to disable this behavior and manage restarts through an external tool instead.

## Default Behavior

When the operator detects a configuration change, it performs a rolling restart of the Dex deployment by patching an annotation on the pod template. This requires two flags to be set (which are configured by default):

- `--dex-namespace` -- the namespace where the Dex deployment runs
- `--dex-deployment-name` -- the name of the Dex deployment

## Disabling Automatic Restart

To disable the automatic restart, set both flags to empty strings in the operator deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dex-config-operator
spec:
  template:
    spec:
      containers:
        - name: manager
          args:
            - "--dex-namespace="
            - "--dex-deployment-name="
```

With both flags empty, the operator still updates the Dex configuration Secret, but it does not trigger a deployment rollout. Dex continues running with its previous configuration until it is restarted by another mechanism.

!!! warning
    When automatic restart is disabled, configuration changes do not take effect until Dex is restarted. Ensure you have an alternative restart mechanism in place.

## Alternative: Stakater Reloader

[Stakater Reloader](https://github.com/stakater/Reloader) is a Kubernetes controller that watches for changes to Secrets and ConfigMaps, then triggers rolling restarts of associated workloads. This is a reliable alternative to the operator's built-in restart mechanism.

### Step 1: Install Reloader

Follow the [Reloader installation guide](https://github.com/stakater/Reloader#how-to-use-reloader) to deploy it in your cluster.

### Step 2: Annotate the Dex Deployment

Add the Reloader annotation to the Dex deployment, referencing the Secret that the operator writes the Dex configuration to:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dex
  annotations:
    secret.reloader.stakater.com/reload: "dex-config"
spec:
  template:
    spec:
      containers:
        - name: dex
          # ...
```

Whenever the `dex-config` Secret is updated by the operator, Reloader detects the change and performs a rolling restart of the Dex deployment.

### Step 3: Disable the Operator's Built-In Restart

Combine the Reloader annotation with disabled auto-restart to avoid duplicate restarts:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dex-config-operator
spec:
  template:
    spec:
      containers:
        - name: manager
          args:
            - "--dex-namespace="
            - "--dex-deployment-name="
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dex
  annotations:
    secret.reloader.stakater.com/reload: "dex-config"
spec:
  template:
    spec:
      containers:
        - name: dex
          image: ghcr.io/dexidp/dex:v2.38.0
          args:
            - "dex"
            - "serve"
            - "/etc/dex/config.yaml"
```

## When to Disable Automatic Restart

| Scenario | Recommendation |
|---|---|
| Simple single-operator setup | Keep auto-restart enabled (default). |
| GitOps with Argo CD or Flux | Disable auto-restart; let the GitOps tool manage rollouts. |
| Reloader already in the cluster | Disable auto-restart; use Reloader for consistency. |
| Canary or blue-green deployments | Disable auto-restart; manage rollouts through your deployment strategy. |
| Maintenance windows | Disable auto-restart to batch configuration changes and restart once. |
