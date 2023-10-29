# Rancher

## Installation

### Create Helm Source with Flux
```
flux create source helm rancher \
    --url https://releases.rancher.com/server-charts/stable \
    --interval 1m0s \
    --export > rancher-helm-repo.yaml
```

### Create Helm Release with Flux

```
flux create helmrelease rancher \
    --namespace=cattle-system \
    --source=HelmRepository/rancher.flux-system \
    --chart=rancher \
    --chart-version="2.7.6" \
    --release-name=rancher \
    --target-namespace=cattle-system \
    --interval=1m0s \
    --values=values.yaml \
    --export > rancher-helm-release.yaml
```