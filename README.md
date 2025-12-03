# Helm Chart Organization

**Helm Chart Organization** is a curated, versioned collection of Helm charts and Custom Resource Definitions (CRDs) for Kubernetes, organized for GitOps workflows and CI/CD pipelines. This repository serves as the canonical source of charts and CRDs consumed by [01cloud-development](https://github.com/BerryBytes/01cloud-development/) for bootstrapping and upgrading Kubernetes environments.

---

## Table of Contents

- [Overview](#overview)
- [Available Packages](#available-packages)
- [Quick Start](#quick-start)
- [Usage Examples](#usage-examples)
- [Deployment Guidelines](#deployment-guidelines)
- [Best Practices](#best-practices)
- [Contributing](#contributing)

---

## Overview

This repository provides a **Git-friendly layout** for managing Helm charts and CRDs with the following benefits:

- **Version Pinning**: Each package maintains multiple versions for reproducible deployments
- **CRD Management**: CRDs are versioned alongside charts to ensure upgrade compatibility
- **GitOps Ready**: Designed for consumption by automation tools and CI/CD pipelines
- **Kubernetes Compatibility**: Organized by Kubernetes version for compatibility assurance

### Purpose
- Provide pinned/upstream chart versions and CRDs in a Git-friendly layout for GitOps, CI pipelines, and offline installs
- Serve as the canonical source of charts/CRDs consumed by **01cloud-development** when bootstrapping and upgrading environments
- Enable reproducible deployments with exact chart content reviewed and stored in Git

---

### Key Architecture Concepts

- **Package Versioning**: Each package maintains multiple versions under `packages/<name>/v<semver>/`
- **CRD Versioning**: CRDs are versioned alongside chart versions to ensure upgrade reproducibility
- **Kubernetes Compatibility**: `v1.19/` through `v1.26/` directories contain version-specific indexes
- **GitOps Integration**: Repository layout designed for consumption by automation tools

---

## Available Packages

The repository contains the following charts:

| Package | Description |
|---------|-------------|
| **cert-manager** | TLS certificate management |
| **contour** | Ingress controller with Envoy proxy |
| **prometheus** | Monitoring stack with operator |
| **sealed-secrets** | Encrypted secret management |
| **velero** | Backup and restore |
| **tekton** | CI/CD pipelines |
| **external-secrets** | External secret management |
| **flagger** | Progressive delivery |
| **openebs** | Container storage |
| **dns-controller** | Custom DNS controller |
| **lb-controller** | Load balancer controller |
| **reloader** | Configuration reload automation |
| **zerone-jobs** | Custom job definitions |

---

## Quick Start

### Prerequisites

- Kubernetes cluster (v1.19+)
- **Helm 3** installed
- **kubectl** installed
- Cluster administrator privileges

### Installation Examples

#### Installing Charts Locally

```bash
# For charts with expanded directories
helm dependency update packages/contour/v12.6.4/contour
helm install contour packages/contour/v12.6.4/contour \
  --namespace projectcontour --create-namespace

# For packaged charts
helm install cert-manager packages/cert-manager/v1.18.2/cert-manager-v1.18.2.tgz \
  --namespace cert-manager --create-namespace
```

#### Applying CRDs

```bash
# Apply CRDs before installing operators
kubectl apply -f packages/cert-manager/v1.18.2/cert-manager.crds.yaml
kubectl apply -f packages/prometheus/v24.5.0/prometheus-operator.crds.yaml
```

---

## Usage Examples

### Integration with 01cloud-development

During deployment to a Kubernetes cluster, **01cloud-development** performs the following:

1. **Reads pinned chart versions** from package directories
2. **Applies CRDs** in controlled order before chart installations
3. **Installs/upgrades Helm charts** using exact content stored in Git
4. **Ensures deployments** use reviewed and auditable chart versions

---

## Deployment Guidelines

### Recommended Deployment Order

When deploying to a cluster, follow this order:

1. **Apply CRDs** from packages that provide them (cert-manager, prometheus-operator, sealed-secrets, etc.)
2. **Install core operators** (cert-manager, prometheus-operator, sealed-secrets)
3. **Install platform charts** (contour, ingress, monitoring stacks)
4. **Deploy application resources** that depend on the above

### Default Namespace Mappings

Default namespaces are defined in `packages/namespaces/namespace.json`:

- cert-manager → `zerone-cert-manager`
- contour → `zerone-projectcontour`
- prometheus → `zerone-monitoring`
- sealed-secrets → `zerone-sealed-secrets`
- velero → `velero`
- tekton → `tekton-pipelines`

---

## Best Practices

### For Maintainers

- **Add new chart versions** under `packages/<name>/v<semver>/` and include any CRD YAML in the same folder
- **Keep upstream files** like `Chart.yaml` and `values.yaml` unchanged where possible
- **Provide override files** separately instead of modifying upstream files
- **Update automation** to reference new version paths when adding versions

### For Users

- **Review CRD changes** carefully when upgrading across major versions
- **Test upgrades** in development environments first
- **Apply CRDs** before installing operators to avoid dependency issues
- **Follow deployment order** to prevent installation failures

### Working with Chart Versions

#### Adding New Chart Versions

1. Create new version directory: `packages/<name>/v<semver>/`
2. Add the packaged chart as `.tgz` file
3. Extract and add CRDs as `<name>.crds.yaml` if applicable
4. Update relevant `v<k8s-version>/index.yaml` and `crds.txt` files

#### Upgrading Charts

- Review CRD changes carefully when upgrading across major versions
- Some CRD modifications are not backwards-compatible
- Test upgrades in development environments first
- Update automation/GitOps references to new version paths

---

## Repository Hosting

Charts are served from GitHub Pages at `https://github.com/berrybytes/helm-chart-org/` with direct links to packaged charts and CRDs in the repository structure.

---

## Contributing

Interested in contributing? Please follow these guidelines:

- Ensure all new packages follow the established directory structure
- Include proper versioning for both charts and CRDs
- Test compatibility with target Kubernetes versions
- Update relevant index files and documentation

---

## Credits

Special thanks to [Berrybytes](https://www.berrybytes.com) for maintaining this project and enabling seamless Kubernetes deployments!
