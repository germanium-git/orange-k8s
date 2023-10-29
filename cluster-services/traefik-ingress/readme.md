# Traefik

## Installation

### Create Helm Source with Flux
```
flux create source helm traefik \
    --url https://traefik.github.io/charts \
    --interval 1m0s \
    --export > traefik-helm-repo.yaml
```

### Create Helm Release with Flux

To get the traefic controller instaleld follow the instructions from https://doc.traefik.io/traefik/getting-started/install-traefik/#use-the-helm-chart

```
helm install --namespace=traefik-v2 \
    traefik traefik/traefik
```
However, in my case I wanted fluxcd to install the traefik usint its helm controller.
First, create a manifest for the HelmRelease and add its name to the `kustomization.yaml` to have fluxcd do the job.

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


Don't forget to update the values so that the inbgress route for dashboard is not created by helm. Bu default there's another ingress route created with `ingressroutes.traefik.io/v1alpha1` API that did not work. Rather then trting to make it work I created ingress route using `ingressroutes.traefik.containo.us/v1alpha1` API - `ingressroute-dashboard.yaml` that makes the the dashboard accessible. More information on how to make the dashboard accessible can be found in the following paragraph.

### Dashboard

Follow the instructioins from the article https://traefik.io/blog/install-and-configure-traefik-with-helm/.
Create a secret + middleware using `middleware-dashboard.yaml`.

### Check current values

```
helm get manifest traefik -n traefik-v2
```

## Notice

## Colision between  two CRDs

There're two APIs coming along the installation of the traefik as shown below
- ingressroutes.traefik.containo.us/v1alpha1 (fully oparational)
- ingressroutes.traefik.io/v1alpha1 (does not work)

For some reasomn the IngressRoute object created by `ingressroutes.traefik.io` do now work.
