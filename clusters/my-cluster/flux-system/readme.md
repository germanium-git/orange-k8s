# FluxCD

## Monitoring & Alerting

Follow the instructions on how to configure the notification controller to use Slack.

https://fluxcd.io/flux/components/notification/providers/#slack

### Sealed secret

Create a sealed secret with the Bot User OAuth Token for the fluxcd app with the scopes channels:read and chat:write.

```
kubectl create secret generic flux-slack-notification \
    --namespace=flux-system \
    --from-literal=slack_webhook=xoxb-1234567890-1234567890-1234567890-1234567890 \
    --dry-run=client -o yaml | kubeseal --format=yaml > flux-sealedsecret.yaml
```
