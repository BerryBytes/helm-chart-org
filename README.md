# helm-chart-org

A curated, versioned collection of Helm charts for Kubernetes. This repository currently packages multiple charts in a stable, reproducible layout suitable for GitOps and CI/CD.

- Kubernetes: Ingress controller + Envoy proxy via the Contour chart
- Helm: v3-compatible charts organized by package and upstream chart version

## Repository structure
## helm-chart-org

This repository is a curated, versioned collection of Helm packages and CRD manifests used by the 01cloud-development application when it is deployed to Kubernetes.

### Purpose
- Provide pinned/upstream chart versions and CRDs in a Git-friendly layout for GitOps, CI pipelines, and offline installs.
- Serve as the canonical source of charts/CRDs consumed by `01cloud-development` when bootstrapping and upgrading environments.

### Repository layout (high level)

```
packages/
  <package-name>/
    v<version>/
      <chart-root>/          # the Helm chart (Chart.yaml, values.yaml, templates/)
      <package>.crds.yaml    # optional, CRDs bundled for this version
namespaces/
  namespace.json            # optional top-level namespace definitions used by deployments
v<k8s-version>/
  crds.txt                  # legacy / aggregated CRD lists per Kubernetes compatibility (if present)
```

### What this repo contains (examples)
- cert-manager (multiple versions) — includes CRDs like `cert-manager.crds.yaml`
- contour (multiple versions) — includes `contour.crds.yaml` and chart folders
- prometheus — operator CRDs and chart versions
- sealed-secrets, velero, tekton, openebs, external-secrets, flagger, and others

### How 01cloud-development uses this repo
- During deployment to a Kubernetes cluster, `01cloud-development` reads the pinned chart versions and CRD manifests from this repository and applies them in a controlled order:
  - First, CRDs required by operators are applied (for example cert-manager CRDs, prometheus-operator CRDs). CRDs are stored near the chart version that introduced them so upgrades are auditable.
  - Next, the Helm charts under `packages/<name>/v<version>/` are installed or upgraded via Helm (or the repo's GitOps tooling) using the pinned chart contents.
  - The repository is intended to be consumed by automation (CI/CD pipelines or GitOps controllers). This ensures deployments use the exact chart content reviewed and stored in Git.

### Quick examples

- Install a chart locally from the repo using Helm (example: Contour):

```bash
helm dependency update packages/contour/v12.6.4/contour
helm install contour packages/contour/v12.6.4/contour \
  --namespace projectcontour --create-namespace
```

- Apply CRDs for a package (example: cert-manager):

```bash
kubectl apply -f packages/cert-manager/v1.18.2/cert-manager.crds.yaml
```

### Recommended deployment ordering for automation
1. Apply required CRDs from packages that provide them.
2. Install core operators (cert-manager, prometheus-operator, sealed-secrets, etc.).
3. Install platform charts that depend on operators (contour, ingress, monitoring stacks).
4. Deploy `01cloud-development` application resources that rely on the above.

### Notes about CRDs
- CRDs are intentionally stored alongside the chart version that introduced or tested them to make upgrades reproducible and auditable.
- When upgrading across major chart versions or operator releases that change CRDs, review CRD diffs carefully. Some CRD modifications are non-backwards-compatible.

### Best practices for maintainers
- Add new chart versions under `packages/<name>/v<semver>/` and include any CRD YAML in the same folder.
- Keep `Chart.yaml` and `values.yaml` from upstream where possible. If you need organization defaults, provide separate override files (do not modify upstream files unless necessary).
- Update CI/CD or GitOps automation to reference the new version path (for example `packages/velero/v5.1.0/velero`).

### Files of interest
- `packages/` — main chart and CRD content
- `namespaces/namespace.json` — optional default namespaces used by deployments

