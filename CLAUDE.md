# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

GitOps repository for a 3-node Kubernetes cluster running on Orange Pi 5 ARM64 single-board computers (1 master + 2 workers). FluxCD reconciles manifests from this repo to the cluster; some newer workloads are managed by ArgoCD from a separate [argocd](https://github.com/germanium-git/argocd) repository.

**Node IPs:** master `172.31.1.50`, workers `172.31.1.51`, `172.31.1.52`  
**NFS server:** `172.31.1.5:/volume1/k8s` (Synology NAS)

## Repository Structure

```
clusters/my-cluster/    # FluxCD entry point
  apps.yaml             # Kustomization pointing to ./apps
  cluster-services.yaml # Kustomization pointing to ./cluster-services

apps/                   # User workloads (active: dnsutils, telegraf, test-pvc)
cluster-services/       # Cluster infrastructure (all currently commented out)
```

FluxCD reconciles every 10 minutes (`interval: 10m0s`). Changes pushed to `main` are automatically applied. To activate or deactivate a workload, comment/uncomment its entry in the relevant `kustomization.yaml`.

## Deploying Changes

All deployment is GitOps — commit and push to `main`. FluxCD picks up changes automatically.

To force immediate reconciliation:
```bash
flux reconcile kustomization apps
flux reconcile kustomization cluster-services
```

Check reconciliation status:
```bash
flux get kustomizations
flux get helmreleases -A
kubectl get pods -A
```

## Secrets Management

Secrets are encrypted with **Sealed Secrets** (`kubeseal`). Never commit plain Kubernetes `Secret` manifests; always seal them first:

```bash
# Create and seal a secret
kubectl create secret generic my-secret \
    --from-literal=key=value \
    --dry-run=client -o yaml | kubeseal --format=yaml > my-sealedsecret.yaml
```

The file `clusters/my-cluster/default/telegraf/telegraf-secrets.yaml` is gitignored — the sealed version `telegraf-sealedsecret.yaml` lives in `apps/telegraf/`.

## Cluster Services (cluster-services/)

All cluster services are currently **commented out** in `cluster-services/kustomization.yaml`. They are kept as reference manifests.

| Service | Status | Notes |
|---|---|---|
| metallb | Commented out | L2 mode, address pool config in `address-pool.yaml` |
| metric-server | Commented out | Requires `--kubelet-insecure-tls` arg for ARM nodes |
| nfs-provisioner | Commented out | Provisioner for NFS PVCs |
| traefik-ingress | Commented out | Use `ingressroutes.traefik.containo.us/v1alpha1` API, not `traefik.io` |
| prometheus | Commented out | Migrated to ArgoCD |

## Apps (apps/)

| App | Notes |
|---|---|
| dnsutils | DaemonSet for DNS debugging |
| telegraf | SNMP polling → InfluxDB at `apollo.germanium.cz:8086`; requires SNMP MIBs on each node at `/var/tmp/telegraf/mibs` |
| test-pvc | PVC smoke test |
| motioneye | Commented out |

## Traefik IngressRoute

Use only the `ingressroutes.traefik.containo.us/v1alpha1` API version (the newer `traefik.io/v1alpha1` CRD does not work in this cluster).

## FluxCD Bootstrap

If the cluster needs to be bootstrapped from scratch:
```bash
export GITHUB_TOKEN=<PAT>
export GITHUB_USER=germanium-git

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=orange-k8s \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal
```

To rotate the GitHub token: delete the secret and re-bootstrap.
```bash
kubectl -n flux-system delete secret flux-system
# then rerun flux bootstrap with same args
```
