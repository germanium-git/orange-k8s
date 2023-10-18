# Rancher

## Installation

### Create Helm Source with Flux
```
flux create source helm rancher \
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
    --namespace=cattle-system \
    --source=HelmRepository/rancher.flux-system \
    --chart=rancher \
    --chart-version="2.5.7" \
    --release-name=rancher \
    --target-namespace=cattle-system \
    --interval=1m0s \
    --values=values.yaml \
    --export > rancher-helm-release.yaml
```