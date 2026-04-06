# Installation

## Prerequisites

- Kubernetes cluster v1.11.3+
- `kubectl` configured with cluster access
- Helm 3

## Install via Helm

```bash
helm install dex-config-operator oci://ghcr.io/stakater/charts/dex-config-operator --version 0.0.6
```

To customize the installation, create a `values-override.yaml` file and pass it during install:

```bash
helm install dex-config-operator oci://ghcr.io/stakater/charts/dex-config-operator --version 0.0.6 -f values-override.yaml
```

## Verification

Confirm the operator is running:

```bash
kubectl get pods -l app.kubernetes.io/name=dex-config-operator
```

Verify the CRDs are registered:

```bash
kubectl get crds | grep stakater
```

You should see entries for `dexconfigs.auth.stakater.com`, `connectors.auth.stakater.com`, `clients.auth.stakater.com`, and `localusers.auth.stakater.com`.

## Next steps

With the operator running, proceed to the [Quick Start](quick-start.md) to create your first DexConfig, Connector, and Client resources.
