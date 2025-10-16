# helm-chart-org

A curated, versioned collection of Helm charts for Kubernetes. This repository currently packages multiple charts (packaged by Bitnami) in a stable, reproducible layout suitable for GitOps and CI/CD.

- Kubernetes: Ingress controller + Envoy proxy via the Contour chart
- Helm: v3-compatible charts organized by package and upstream chart version

## Repository structure

```
#Contour Example
packages/
  contour/
    v12.6.4/
      contour/            # Helm chart root (Chart.yaml, values.yaml, templates/)
        charts/common/    # Bitnami common library chart
```
## Available charts

| Chart | Chart version | App version | Path |
|------|---------------:|------------:|------|
| Contour (Bitnami) | 12.6.4 | 1.25.2 | `packages/contour/v12.6.4/contour` |

Upstream: https://github.com/bitnami/charts/tree/main/bitnami/contour

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- A LoadBalancer implementation (e.g., MetalLB) if exposing Envoy via `Service.type=LoadBalancer`

## Quick start (local install from this repo)

1) Fetch chart dependencies:

```bash
helm dependency update packages/contour/v12.6.4/contour
```

2) Install Contour + Envoy:

```bash
helm install contour packages/contour/v12.6.4/contour \
  --namespace projectcontour --create-namespace
```

3) Verify:

```bash
kubectl get pods -n projectcontour
kubectl get svc -n projectcontour
```

To override defaults, create a file like `my-values.yaml` and pass `-f my-values.yaml`.

## Configuration

- Values file: `packages/contour/v12.6.4/contour/values.yaml`
- Comprehensive parameter reference: `packages/contour/v12.6.4/contour/README.md`

Examples:

```yaml
# my-values.yaml
envoy:
  service:
    type: LoadBalancer
  logLevel: info
contour:
  ingressClass:
    create: true
    default: true
```

Install with overrides:

```bash
helm install contour packages/contour/v12.6.4/contour -n projectcontour -f my-values.yaml
```

## Upgrading

```bash
helm upgrade contour packages/contour/v12.6.4/contour -n projectcontour -f my-values.yaml
```

If you are upgrading across chart majors or when CRDs change, review the upstream “Upgrading Contour” guide: https://projectcontour.io/resources/upgrading/

## Uninstalling

Warning: Uninstalling may remove CRDs if `contour.manageCRDs=true`. Back up CRs if you need to preserve them:

```bash
kubectl get -o yaml extensionservice,httpproxy,tlscertificatedelegation -A > backup.yaml
helm uninstall contour -n projectcontour
```

## Developing locally

- Lint the chart:

```bash
helm lint packages/contour/v12.6.4/contour
```

- Render manifests for review:

```bash
helm template contour packages/contour/v12.6.4/contour -n projectcontour -f my-values.yaml > rendered.yaml
```

- Dry-run an upgrade:

```bash
helm upgrade --install contour packages/contour/v12.6.4/contour \
  -n projectcontour -f my-values.yaml --dry-run --debug
```

## Adding or updating charts in this repo

- New versions are placed under `packages/<name>/v<chartVersion>/<chartName>`
- Keep `Chart.yaml` and `values.yaml` as provided by upstream (or your fork) and commit any organization-specific defaults in your own override files or a separate overlay
- Run `helm dependency update` after adding or bumping dependencies

## Acknowledgements

- Contour chart maintained by VMware/Bitnami. Trademarks are property of their respective owners.

