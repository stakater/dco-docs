# Overview

This page covers the prerequisites for running Dex Config Operator (DCO) and outlines the available installation methods.

## Prerequisites

Before installing DCO, ensure your environment meets the following requirements:

| Requirement | Minimum Version |
|---|---|
| Go | 1.24+ |
| Docker | 17.03+ |
| kubectl | v1.11.3+ |
| Kubernetes cluster | v1.11.3+ |

!!! note
    Go is only required if you plan to build DCO from source. Pre-built container images and manifests do not require a local Go installation.

## Installation options

DCO can be installed using any of the following methods:

### Helm chart

The recommended approach for production deployments. The Helm chart is located at `charts/dex-config-operator/` in the project repository and provides configurable values for resource limits, replicas, and other operational settings.

```bash
helm install dex-config-operator charts/dex-config-operator/
```

### Kustomize

Suitable for environments that already use Kustomize for manifest management. Deploy with a custom image reference:

```bash
make deploy IMG=<registry>/dex-config-operator:tag
```

### Raw manifests

The simplest option for quick setups or CI pipelines. Apply the consolidated manifest directly:

```bash
kubectl apply -f dist/install.yaml
```

## Next steps

Proceed to [Installation](installation.md) for detailed step-by-step instructions for each method, or jump to the [Quick Start](quick-start.md) to get a working configuration in minutes.
