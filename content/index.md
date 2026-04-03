# Dex Config Operator

Dex Config Operator (DCO) is a Kubernetes operator that manages [Dex](https://dexidp.io/) OIDC configurations declaratively through Custom Resource Definitions (CRDs). Instead of manually editing Dex configuration files, define your connectors, clients, and storage backends as Kubernetes resources and let DCO handle the rest.

**API Group:** `auth.stakater.com/v1alpha1`

## Why DCO?

- **Security-first** -- Sensitive data lives in Kubernetes Secrets, never in CRDs.
- **Event-driven** -- Configuration regenerates automatically when CRDs or referenced Secrets change.
- **Auto-restart** -- Dex deployments receive rolling restarts on every config update, with zero manual intervention.
- **GitOps-friendly** -- CRDs are plain YAML, ready for version control and CI/CD pipelines.

## Get started

| Section | Description |
|---|---|
| [Overview](overview/key-features.md) | Key features and architecture |
| [Getting Started](getting-started/) | Installation and first configuration |
| [Guides](guides/) | Step-by-step walkthroughs for common tasks |
| [Reference](reference/) | CRD specifications and API details |
| [Integrations](integrations/) | Using DCO with external identity providers and storage backends |
