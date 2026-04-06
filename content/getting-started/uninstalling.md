# Uninstalling

This page explains how to remove Dex Config Operator and its associated resources from your cluster.

## Step 1 — Remove custom resources

Delete all DCO custom resources before removing the operator. This ensures finalizers run and any managed resources are cleaned up properly.

```bash
kubectl delete clients --all -A
kubectl delete connectors --all -A
kubectl delete dexconfigs --all -A
```

Verify that no resources remain:

```bash
kubectl get clients -A
kubectl get connectors -A
kubectl get dexconfigs -A
```

## Step 2 — Remove the operator

Use the method that matches your original installation.

### Helm

```bash
helm uninstall dex-config-operator
```

### Kustomize

```bash
make undeploy
```

### Raw manifests

```bash
kubectl delete -f dist/install.yaml
```

## Step 3 — Remove CRDs

!!! warning
    Deleting CRDs removes **all** instances of those resources across the cluster. Make sure Step 1 is complete before proceeding.

```bash
make uninstall
```

Alternatively, delete the CRDs directly:

```bash
kubectl delete crd dexconfigs.auth.stakater.com
kubectl delete crd connectors.auth.stakater.com
kubectl delete crd clients.auth.stakater.com
```

## Step 4 — Clean up namespace (optional)

If the operator namespace is no longer needed:

```bash
kubectl delete namespace dex-config-operator-system
```

## Verification

Confirm that no operator pods, CRDs, or namespaces remain:

```bash
kubectl get crds | grep stakater
kubectl get namespace dex-config-operator-system
```

Both commands should return no results.
