# CloudNative MySQL Helm charts

This repository contains the official Helm chart for [CloudNative MySQL](https://cloudnative-mysql.io), a Kubernetes operator for Percona Server for MySQL.

The chart lives under [`charts/cloudnative-mysql`](./charts/cloudnative-mysql). It can install the operator in two ways:

- **Cluster-wide** (default): the operator watches every namespace and manages `Cluster` resources anywhere in the cluster.
- **Namespaced**: the operator watches only the release namespace, which lets multiple independent operators coexist on the same cluster.

## Requirements

- Kubernetes 1.29 or newer
- Helm 3.13 or newer
- cert-manager installed in the target cluster (the chart creates `Certificate` and `Issuer` objects for webhook TLS)

## Quick start

Add this repository or use the chart directly from the local path.

### Install cluster-wide

```bash
helm install cnmysql ./charts/cloudnative-mysql \
  --namespace cnmysql-system \
  --create-namespace
```

The operator image defaults to `ghcr.io/cloudnative-mysql/cloudnative-mysql:<chart-appVersion>`.

### Install namespaced

```bash
helm install cnmysql ./charts/cloudnative-mysql \
  --namespace tenant-a \
  --create-namespace \
  --set rbac.namespaced=true
```

In namespaced mode the release is constrained to `tenant-a`. The chart creates a `Role` instead of a `ClusterRole`, injects the `WATCH_NAMESPACE` environment variable, and scopes the validating webhook with a namespace selector.

## Image overrides

To point the chart at a custom image:

```bash
helm install cnmysql ./charts/cloudnative-mysql \
  --namespace cnmysql-system \
  --create-namespace \
  --set manager.image.repository=ghcr.io/cloudnative-mysql/cloudnative-mysql \
  --set manager.image.tag=main
```

## Uninstall

```bash
helm uninstall cnmysql -n cnmysql-system
```

CRDs are kept by default (`crd.keep=true`) so uninstalling a release does not remove existing `Cluster` or `Backup` resources.

## OCI releases

The chart is packaged and pushed to GitHub Container Registry on every pushed tag that starts with `v` and on workflow dispatch. Install from OCI:

```bash
helm install cnmysql oci://ghcr.io/cloudnative-mysql/charts/cloudnative-mysql \
  --namespace cnmysql-system \
  --create-namespace
```

## Repository layout

```
charts/
└── cloudnative-mysql/         # the Helm chart
    ├── Chart.yaml
    ├── values.yaml
    ├── README.md
    └── templates/
```

## Contributing

Follow the operator repository conventions. Before committing, run:

```bash
helm lint ./charts/cloudnative-mysql
helm lint ./charts/cloudnative-mysql --set rbac.namespaced=true
```

Changes to templates should be validated against both installation modes.

## License

Apache 2.0. See the operator repository for the full license.
