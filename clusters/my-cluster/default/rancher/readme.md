# Rancher

## Installation

### Create Helm Source with Flux
```
flux create source helm rancher-server \
    --url https://releases.rancher.com/server-charts/stable \
    --interval 1m0s \
    --export > rancher-helm-repo.yaml
```

helm install rancher-<CHART_REPO>/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org

### Create Helm Release with Flux

```
flux create helmrelease rancher \
    --source=HelmRepository/rancher-server \
    --chart=rancher-server \
    --release-name=rancher \
    --target-namespace=rancher \
    --interval=5m0s \
    --values=values.yaml \
    --export > rancher-helm-release.yaml
```