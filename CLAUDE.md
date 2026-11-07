# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Documentation repository for a 3-node Kubernetes cluster running on Orange Pi 5 ARM64 single-board computers
(1 master + 2 workers). It is **not** a GitOps source any more: all workloads are deployed by **Argo CD** from
the [argocd](https://github.com/germanium-git/argocd) repository. FluxCD, which used to reconcile this repo,
was removed from the cluster on 2026-11-07 (manifests remain in git history).

**Node IPs:** master `172.31.1.50`, workers `172.31.1.51`, `172.31.1.52` (all arm64 – check images for arm64)
**NFS server:** `172.31.1.6:/volume1/nfs` (Synology NAS, StorageClass `nfs-atlas`)

## Contents

- `hardware.md`, `Installation-k8s.md`, `installation-terminal.md` – cluster build and installation notes
- `.claude/commands/k8s-upgrade.md` – `/k8s-upgrade` procedure for Kubernetes version upgrades
