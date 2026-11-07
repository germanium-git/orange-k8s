# Orange Pi K8s cluster

Documentation for the 3-node Kubernetes cluster on Orange Pi 5 (ARM64): hardware, OS and Kubernetes
installation, and the cluster upgrade procedure.

- [hardware.md](hardware.md) – boards, cluster build
- [Installation-k8s.md](Installation-k8s.md) – Kubernetes installation
- [installation-terminal.md](installation-terminal.md) – terminal / tooling setup
- `/k8s-upgrade` (Claude Code command in `.claude/commands/`) – Kubernetes version upgrade

## Workloads: Argo CD

**All workloads are deployed by Argo CD** from the [argocd](https://github.com/germanium-git/argocd) repository.

## History: FluxCD (removed 2026-11-07)

> **FluxCD was removed from the cluster on 7 November 2026.** This repository is no longer reconciled by
> anything; all workloads are deployed by Argo CD from [argocd](https://github.com/germanium-git/argocd).

This repository used to be the FluxCD GitOps source (`clusters/my-cluster`, `apps/`, `cluster-services/`).
Over time everything moved to Argo CD; on 2026-11-07 the last Flux workloads moved as well and Flux was removed:

| Former Flux workload | Now |
|---|---|
| `apps/telegraf` (SNMP → InfluxDB on apollo) | `argocd/apps/telegraf` – telegraf 1.40.3, MIBs in a ConfigMap (no hostPath), Prometheus alerts |
| `apps/dnsutils` | `argocd/apps/dnsutils` |
| `apps/test-pvc` | deleted (unused) |
| `apps/motioneye`, `cluster-services/*` | already in Argo CD earlier |

Removed with Flux on 2026-11-07: the `flux-system` namespace and controllers, the Flux CRDs and RBAC, the
receiver LoadBalancer `172.31.1.31`, this repository's Flux deploy key and webhook, and the SRX destination-NAT
rule/policy that forwarded the webhook to the receiver. The old manifests remain in the git history.
