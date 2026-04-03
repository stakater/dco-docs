# Installation

This page provides detailed installation steps for each supported method. See [Overview](overview.md) for prerequisites.

## Step 1 -- Install CRDs

Regardless of which deployment method you choose, the Custom Resource Definitions must be installed first:

```bash
make install
```

Verify that the CRDs are registered:

```bash
kubectl get crds | grep stakater
```

You should see entries for `dexconfigs.auth.stakater.com`, `connectors.auth.stakater.com`, and `clients.auth.stakater.com`.

## Step 2 -- Deploy the operator

Choose one of the following methods.

### Option A -- Helm chart

1. Clone the repository and navigate to the project root:

    ```bash
    git clone https://github.com/stakater/dex-config-operator.git
    cd dex-config-operator
    ```

1. Install the chart with default values:

    ```bash
    helm install dex-config-operator charts/dex-config-operator/
    ```

    To customize the installation, create a `values-override.yaml` file and pass it during install:

    ```bash
    helm install dex-config-operator charts/dex-config-operator/ -f values-override.yaml
    ```

1. Verify the deployment:

    ```bash
    kubectl get pods -l app.kubernetes.io/name=dex-config-operator
    ```

### Option B -- Kustomize

1. Build and push the operator image to your registry:

    ```bash
    make docker-build docker-push IMG=<registry>/dex-config-operator:tag
    ```

1. Deploy the operator:

    ```bash
    make deploy IMG=<registry>/dex-config-operator:tag
    ```

1. Verify the deployment:

    ```bash
    kubectl get pods -n dex-config-operator-system
    ```

### Option C -- Raw manifests

1. Apply the consolidated manifest:

    ```bash
    kubectl apply -f dist/install.yaml
    ```

1. Verify the deployment:

    ```bash
    kubectl get pods -n dex-config-operator-system
    ```

## Verification

After deploying with any method, confirm the operator is running and ready:

```bash
kubectl get deployment -A | grep dex-config-operator
```

Check the operator logs for startup errors:

```bash
kubectl logs -l control-plane=controller-manager -n dex-config-operator-system
```

## Next steps

With the operator running, proceed to the [Quick Start](quick-start.md) to create your first DexConfig, Connector, and Client resources.
