# Control-plane flags that reduce etcd disk writes

orange1 is the only control-plane node and runs etcd on its NVMe. On 2026-11-20 that disk was at 45%
of its rated write endurance (13.7 TB written in 2.8 years), and etcd wrote about 5.5 GiB a day to it.
The flags below cut that without weakening crash safety. The full ClusterConfiguration is in
[`kubeadm-clusterconfiguration.yaml`](kubeadm-clusterconfiguration.yaml).

| Component | Flag | Default | Set to | Why |
|---|---|---|---|---|
| etcd | `--backend-batch-interval` | 100ms | `1s` | etcd saves its database file (bbolt) at most this often, and each save costs about 33 KiB on disk however few changes it holds. With about 3 changes a second, almost every change got its own save. Every change is still written and fsynced to the WAL first, and etcd replays the WAL after a crash, so nothing is lost on a power cut. |
| kube-controller-manager, kube-scheduler | `--leader-elect-retry-period` | 2s | `10s` | With one control-plane node there is nobody to elect against, but each renewed its lease every 2 s (1 write/s together). |
| | `--leader-elect-renew-deadline` | 10s | `40s` | Must be longer than the retry period. |
| | `--leader-elect-lease-duration` | 15s | `60s` | Must be longer than the renew deadline. After a restart the component may wait up to 60 s before it takes over the lease again; running pods are not affected. |

Leader election stays on, so a second control-plane node can be added later without changing anything.

Related changes in the argocd repo: Istio and Envoy Gateway removed (PR #186), leader election turned off
in the nfs-atlas provisioner and the JupyterHub user scheduler (PR #189).

## Applying (or re-applying after a rebuild)

Do it from a workstation with `kubectl` and SSH access to orange1. The API server is unavailable for about
half a minute while etcd restarts; running workloads are not affected.

```bash
# 0. A fresh backup first (includes an etcd snapshot)
kubectl create job --from=cronjob/backup -n backup backup-manual-$(date +%s)

# 1. Store the flags in kubeadm-config, so "kubeadm upgrade" keeps them
kubectl -n kube-system create configmap kubeadm-config \
  --from-file=ClusterConfiguration=backup-restore/node-config/kubeadm-clusterconfiguration.yaml \
  --dry-run=client -o yaml | kubectl replace -f -

# 2. On orange1: keep a copy of the manifests OUTSIDE /etc/kubernetes/manifests
#    (the kubelet runs every file in that folder)
ssh -i ~/.ssh/orangePiMac petr@172.31.1.50
sudo cp -a /etc/kubernetes/manifests /root/manifests-$(date +%F)

# 3. etcd first; the kubelet restarts the pod when the file changes
sudo sed -i 's|^\(\s*\)- --snapshot-count=10000$|&\n\1- --backend-batch-interval=1s|' /etc/kubernetes/manifests/etcd.yaml
# wait until the API server answers again (about 30 s)
until sudo kubectl --kubeconfig /etc/kubernetes/admin.conf get --raw /readyz >/dev/null 2>&1; do sleep 5; done

# 4. Controller manager and scheduler
for f in kube-controller-manager kube-scheduler; do
  sudo sed -i 's|^\(\s*\)- --leader-elect=true$|&\n\1- --leader-elect-lease-duration=60s\n\1- --leader-elect-renew-deadline=40s\n\1- --leader-elect-retry-period=10s|' \
    /etc/kubernetes/manifests/$f.yaml
done
```

## Checking

```bash
# Flags are live
kubectl -n kube-system get pod etcd-orange1 kube-controller-manager-orange1 kube-scheduler-orange1 \
  -o jsonpath='{range .items[*]}{.metadata.name}: {.spec.containers[0].command}{"\n"}{end}' | grep -o -- '--\(backend\|leader\)[^",]*'

# Leases are renewed every ~10 s, not every 2 s
kubectl -n kube-system get lease kube-controller-manager kube-scheduler -w

# etcd database saves per second (on orange1); was about 1.9/s before
curl -s http://127.0.0.1:2381/metrics | grep '^etcd_disk_backend_commit_duration_seconds_count'
```

## Rolling back

On orange1, copy the saved manifests back (`sudo cp /root/manifests-<date>/*.yaml /etc/kubernetes/manifests/`)
and remove the `extraArgs` from `kubeadm-config`.
