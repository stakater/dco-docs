# Overview

This page covers the prerequisites for running Dex Config Operator (DCO) and outlines the installation method.

## Prerequisites

| Requirement | Minimum Version |
|---|---|
| kubectl | v1.11.3+ |
| Kubernetes cluster | v1.11.3+ |
| Helm | 3 |

## Installation

DCO is installed via its Helm chart hosted as an OCI artifact:

```bash
helm install dex-config-operator oci://ghcr.io/stakater/charts/dex-config-operator --version 0.0.6
```

See [Installation](installation.md) for detailed instructions, or jump to the [Quick Start](quick-start.md) to get a working configuration in minutes.
