# orange-k8s
Orange Pi K8s cluster


## FluxCD

The applications running on the cluster are deployed by FluxCD.

### Personal Access Token

![token](/pictures/image.png)

### User permissions

- This token does not have any user permissions.

### Repository permissions

-  Read access to Dependabot alerts, actions variables, codespaces, codespaces lifecycle admin, codespaces metadata, commit statuses, dependabot secrets, deployments, discussions, environments, issues, merge queues, metadata, pages, repository advisories, secret scanning alerts, and security events

- Read and Write access to actions, administration, code, pull requests, and repository hooks

### Bootstrap

```
export GITHUB_TOKEN=github_pat_11.....kThW
export GITHUB_USER=germanium-git

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=orange-k8s \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal
```