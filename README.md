# Orange Pi K8s cluster

Documentation for the 3-node Kubernetes cluster on Orange Pi 5 (ARM64): hardware, OS and Kubernetes
installation, and the cluster upgrade procedure.

- [hardware.md](hardware.md) – boards, cluster build
- [Installation-k8s.md](Installation-k8s.md) – Kubernetes installation
- [installation-terminal.md](installation-terminal.md) – terminal / tooling setup
- `/k8s-upgrade` (Claude Code command in `.claude/commands/`) – Kubernetes version upgrade

## Workloads: Argo CD

**All workloads are deployed by Argo CD** from the [argocd](https://github.com/germanium-git/argocd) repository.

## History: FluxCD (retired Nov 2026)

This repository used to be the FluxCD GitOps source (`clusters/my-cluster`, `apps/`, `cluster-services/`).
Everything was migrated to Argo CD; the last workload, the SNMP **telegraf**, moved in Nov 2026
(`argocd/apps/telegraf`). Flux was uninstalled, its deploy key and webhook removed from this repo, and the
SRX NAT rule for the Flux receiver deleted. The old manifests remain in the git history.
