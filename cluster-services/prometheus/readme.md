# Prometheus

## Installation

### Create Helm Source with Flux
```
flux create source helm prometheus \
    --url https://prometheus-community.github.io/helm-charts \
    --interval 1m0s \
    --export > prometheus-helm-repo.yaml
```


helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -f values.yaml

```
flux create helmrelease rancher \
    --namespace=default \
    --source=HelmRepository/prometheus.flux-system \
    --chart=kube-prometheus-stack \
    --chart-version="51.2.0" \
    --release-name=prometheus \
    --target-namespace=default \
    --interval=1m0s \
    --values=values.yaml \
    --export > prometheus-helm-release.yaml
```

## Customization

### Sealed secret with the Slack webhook

Add the file to kustomization `prometheus-slack-sealedsecret.yaml`

#### Create a sealed secret

```
kubectl create secret generic alertmanager-slack-notification \
    --from-literal=slack_webhook=https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX \
    --dry-run=client -o yaml | kubeseal --format=yaml > telegraf-sealedsecret.yaml
```
#### Check the value

```
echo webhook: $(kubectl get secret --namespace default alertmanager-slack-notification -o jsonpath="{.data.slack_webhook}" | base64 -d)
```

### Alert Manager Configuration

Add the file to kustomization `prometheus-alertconfig.yaml` to get notified when an alert is fired in Slack
