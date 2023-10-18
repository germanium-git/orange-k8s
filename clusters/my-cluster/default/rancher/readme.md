# Rancher

## Installation

### Create Helm Source with Flux
```
flux create source helm rancher-server \
    --url https://releases.rancher.com/server-charts/stable \
    --interval 1m0s \
    --export > rancher-helm-repo.yaml
```

