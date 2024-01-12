# Orange Pi K8s cluster

## FluxCD

The applications running on the cluster are deployed by FluxCD.\
Disclaimer: ...That is no longer the case, as ArgoCD has stepped in as FluxCD's successor (more information can be found at the bottom of this page)

### Personal Access Token

![token](https://raw.githubusercontent.com/germanium-git/orange-k8s/main/pictures/image.png)


#### User permissions

- This token does not have any user permissions.

#### Repository permissions

-  Read access to Dependabot alerts, actions variables, codespaces, codespaces lifecycle admin, codespaces metadata, commit statuses, dependabot secrets, deployments, discussions, environments, issues, merge queues, metadata, pages, repository advisories, secret scanning alerts, and security events

- Read and Write access to actions, administration, code, pull requests, and repository hooks

### Bootstrap

```
export GITHUB_TOKEN=github_pat_11A..........lUR3
export GITHUB_USER=germanium-git

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=orange-k8s \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal
► connecting to github.com
► cloning branch "main" from Git repository "https://github.com/germanium-git/orange-k8s.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ component manifests are up to date
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
✔ public key: ecdsa-sha2-nistp384 AAA..........UssQ==
✔ configured deploy key "flux-system-main-flux-system-./clusters/my-cluster" for "https://github.com/germanium-git/orange-k8s"
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ sync manifests are up to date
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
```


### Update GH token

Follow the instructions from [Rotate github personal access token #2161](https://github.com/fluxcd/flux2/discussions/2161#discussioncomment-1726813)

*To rotate the bootstrap key be it a token or a deploy key:\
delete the auth secret from the cluster with `kubectl -n flux-system delete secret flux-system`\
rerun flux bootstrap with the same args as before
flux will generate a new secret and will update the deploy key if you’re using SSH deploy keys.*

### Webhook

:TODO

## ArgoCD

Some of the applications running on the cluster have been deployed by ArgoCD from another repository [argocd](https://github.com/germanium-git/argocd)
