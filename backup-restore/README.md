# Backup and restore

Backups, restore procedures and power resilience for the `orange` cluster.

**Full documentation is on XWiki:**
[Projects / Orange K8s / Backup and Restore](https://wiki.germanium.cz/bin/view/Projects/Orange%20K8s/Backup%20and%20Restore/)
(LAN only). It covers how the system works, how it was set up, the restore runbooks and the restore drill log.

## What lives where

| What | Where |
|---|---|
| Documentation and restore runbooks | XWiki (link above) |
| Kubernetes manifests for the backup jobs | `apps/backup` in [germanium-git/argocd](https://github.com/germanium-git/argocd) |
| Node configuration (NUT client, kubelet shutdown settings) | `node-config/` in this folder (added in step 6) |
| Secrets (restic password, Google Drive token) | Bitwarden, and SealedSecrets in the argocd repo |

No passwords, tokens or keys are stored in this folder.

## If the wiki is down

XWiki runs on this cluster, so it may be unavailable exactly when a restore is needed. The XWiki pages are
included in the nightly backup. To read them without the cluster:

1. Get the restic password from Bitwarden.
2. Restore the latest XWiki database dump from the Google Drive repository with `restic`.
3. Load it into any PostgreSQL instance and read the page content from the `xwikidoc` table.

The restore runbooks will also be exported to this folder as Markdown once they are tested (step 5).
