
### Installation


### Create Helm Source with Flux
```
flux create source helm traefik \
    --url https://traefik.github.io/charts \
    --interval 1m0s \
    --export > traefik-helm-repo.yaml
```


### Create Helm Release with Flux

```
flux create helmrelease traefik \
    --namespace=traefik-v2 \
    --source=HelmRepository/traefik.flux-system \
    --chart=traefik \
    --chart-version="2.10.4" \
    --release-name=traefik \
    --target-namespace=traefik-v2 \
    --interval=1m0s \
    --export > traefik-helm-release2.yaml



https://doc.traefik.io/traefik/getting-started/install-traefik/#use-the-helm-chart

```
helm install --namespace=traefik-v2 \
    traefik traefik/traefik
```


### Dashboard

Follow the instructioins from

https://traefik.io/blog/install-and-configure-traefik-with-helm/


### Check current values

```
helm get manifest traefik -n traefik-v2
```


