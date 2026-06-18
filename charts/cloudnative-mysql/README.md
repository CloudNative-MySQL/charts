# cloudnative-mysql

A Helm chart for the CloudNative MySQL operator. The chart deploys the operator, its CustomResourceDefinitions, RBAC, webhook configuration, and everything else needed to run MySQL clusters on Kubernetes.

## Prerequisites

- Kubernetes 1.29+
- Helm 3.13+
- cert-manager must be installed in the cluster
- A default `StorageClass` for `Cluster` PVCs

## Installation modes

The chart supports two modes. Pick the one that matches your environment.

### Cluster-wide (default)

The operator watches all namespaces. Use this when one operator instance manages the whole cluster.

```bash
helm install cnmysql ./charts/cloudnative-mysql \
  --namespace cnmysql-system \
  --create-namespace
```

### Namespaced

The operator watches only the release namespace. Use this for multi-tenant setups where several operators run side by side.

```bash
helm install cnmysql ./charts/cloudnative-mysql \
  --namespace tenant-a \
  --create-namespace \
  --set rbac.namespaced=true
```

Namespaced mode comes with a few differences:

- A `Role`/`RoleBinding` replaces the cluster-wide `ClusterRole`/`ClusterRoleBinding`.
- The `WATCH_NAMESPACE` environment variable is set from the pod namespace.
- The validating webhook only admits `Cluster` resources in the release namespace.
- `ClusterImageCatalog` cannot be used; use the namespaced `ImageCatalog` instead.

## Configuration

The following table lists the most common values. See [`values.yaml`](./values.yaml) for the full set.

| Value | Default | Description |
|---|---|---|
| `manager.image.repository` | `ghcr.io/cloudnative-mysql/cloudnative-mysql` | Operator image repository. |
| `manager.image.tag` | `Chart.appVersion` | Operator image tag. |
| `manager.image.pullPolicy` | `IfNotPresent` | Image pull policy. |
| `manager.replicas` | `1` | Number of controller-manager replicas. |
| `rbac.namespaced` | `false` | Use namespaced `Role`/`RoleBinding` instead of `ClusterRole`/`ClusterRoleBinding`. |
| `crd.enable` | `true` | Install CRDs as part of the release. |
| `crd.keep` | `true` | Keep CRDs when the release is uninstalled. |
| `webhook.enable` | `true` | Enable the validating webhook server. |
| `certManager.enable` | `true` | Create cert-manager `Certificate` and `Issuer` resources for the webhook. |
| `metrics.enable` | `true` | Expose the controller metrics endpoint. |
| `metrics.secure` | `true` | Serve metrics over HTTPS with cert-manager-provided certificates. |
| `prometheus.enable` | `false` | Create a `PodMonitor` for Prometheus Operator. |

## Image overrides

To use a different image or tag, pass:

```bash
helm install cnmysql ./charts/cloudnative-mysql \
  --namespace cnmysql-system \
  --create-namespace \
  --set manager.image.tag=main
```

The chart always passes `--operator-image` to the manager so that bootstrap init containers use the exact same image.

## Registries with authentication

If you need to pull from a private registry, list the pull secret under `manager.imagePullSecrets`:

```yaml
manager:
  imagePullSecrets:
    - name: regcred
```

## Metrics

The metrics endpoint runs on port `8443` by default. When `metrics.secure` is true, cert-manager provides the serving certificate and the chart creates a `ClusterRole` for token review and subject access review.

## Prometheus support

Set `prometheus.enable=true` to create a `PodMonitor` for the operator metrics. The cluster must have the Prometheus Operator CRDs installed.

## Webhooks

The chart deploys a validating webhook for `Cluster/status` updates. cert-manager provisions the certificate. If you disable cert-manager (`certManager.enable=false`), you must provide the certificate yourself.

## Resource limits

Default resource requests and limits are intentionally low:

| Resource | Request | Limit |
|---|---|---|
| CPU | `10m` | `500m` |
| Memory | `64Mi` | `128Mi` |

Adjust them through `manager.resources` for production workloads.

## Uninstall

```bash
helm uninstall cnmysql -n cnmysql-system
```

Because `crd.keep` defaults to `true`, CRDs remain after uninstall. This prevents accidental removal of existing `Cluster` or `Backup` resources. If you also want to remove the CRDs, delete them manually:

```bash
kubectl delete crd clusters.mysql.cloudnative-mysql.io \
  backups.mysql.cloudnative-mysql.io \
  scheduledbackups.mysql.cloudnative-mysql.io \
  databases.mysql.cloudnative-mysql.io \
  imagecatalogs.mysql.cloudnative-mysql.io \
  clusterimagecatalogs.mysql.cloudnative-mysql.io
```

## Multiple releases

If you install the chart more than once, only enable `crd.enable` on the first release. Subsequent releases can reuse the existing CRDs by setting `--set crd.enable=false`. This avoids Helm ownership conflicts on cluster-scoped CRDs.

## Links

- Operator documentation: [cloudnative-mysql.io](https://cloudnative-mysql.io)
- Source code: [CloudNative-MySQL/cloudnative-mysql](https://github.com/CloudNative-MySQL/cloudnative-mysql)
- OCI chart: `oci://ghcr.io/cloudnative-mysql/charts/cloudnative-mysql`
