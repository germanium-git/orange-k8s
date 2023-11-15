# FluxCD

## Installation

First generate a GH token (Repository > Settings > Developer Tools > Personal Access Token > Fine-Grained tokens) with the following repository permissions.

Read and Write access to:
 - Actions
 - Administration
 - Contents
 - Pull requests
 - Webhooks

No Access
- Codespaces secrets
- Secrets
- Workflows

All the rest is set to Read-only.

Install FluxCD cli and bootstrap the cluster.

```
brew install fluxcd/tap/flux

export GITHUB_TOKEN=xxxxxxxxxx
export GITHUB_USER=germanium-git

 flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=orange-k8s \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal
```

## Webhook

Follow the steps decribed in the instructions
https://fluxcd.io/flux/guides/webhook-receivers/

Create a LB service to receive the notification pushed by the GitHub - `flux-receiver-service.yaml`

The repository objects does not have to be created as it aleredy exists since the repo was bootstrapped. Check the object types `source.toolkit.fluxcd.io/v1/gitrepositories` to see the respective GitRepository called flux-system.

Generate a random string and create a secret `webhook-token` with a token field:

```
TOKEN=$(head -c 12 /dev/urandom | shasum | cut -d ' ' -f1)
echo $TOKEN

kubectl -n flux-system create secret generic webhook-token \
--from-literal=token=$TOKEN
```

Create a receiver for GitHub and specify the GitRepository object - `flux-github-receiver.yaml`

```
flux create receiver flux-system \
 --type github \
 --event ping \
 --event push \
 --secret-ref webhook-token \
 --resource GitRepository/flux-system \
 --namespace flux-system \
 --export > flux-github-receiver.yaml
```

Check the message to get the path /hook/xxxxx required inthe next step.
```
flux get receivers -n flux-system
```

Use the URL reported by the receiver and configure the webhook respectively in the GitHub repository (Settings > Webhooks). The URL should look like http://webhook.germanium.cz/hook/xxxxxxxxxxx

![webhook1](pictures/webhook.png)

Don't forget to configure dNAT rule on the router to let notification messages reach the LB service.

## Monitoring & Alerting

Follow the instructions on how to configure the notification controller to use Slack.

https://fluxcd.io/flux/components/notification/providers/#slack

### Sealed secret

Create a sealed secret with the *Bot User OAuth Token* for the fluxcd app with the scopes channels:read and chat:write.

```
kubectl create secret generic flux-slack-notification \
    --namespace=flux-system \
    --from-literal=token=xoxb-1234567890-1234567890-1234567890-1234567890 \
    --dry-run=client -o yaml | kubeseal --format=yaml > flux-sealedsecret.yaml
```

### Slack

#### Send a message with curl

Test if you can post a message in a specific Slack channel by curl.
Follow the instructions from the article [Posting messages using curl](https://api.slack.com/tutorials/tracks/posting-messages-with-curl).

Check if the token works.

```
❯ curl -H "Authorization: Bearer xoxb-not-a-real-token-this-will-not-work" https://slack.com/api/auth.test
{"ok":true,"url":"https:\/\/germanium-workspace.slack.com\/","team":"Germanium","user":"fluxcd2","team_id":"T05UB5W8H63","user_id":"U0633HVJ0JJ","bot_id":"B063CLKD7CL","is_enterprise_install":false}%
```

Send a message.

```
❯ curl -d "text=Hi I am a bot that can post messages to any public channel." -d "channel=fluxcd" -H "Authorization: Bearer token=xoxb-not-a-real-token-this-will-not-work" -X POST https://slack.com/api/chat.postMessage

{"ok":true,"channel":"C062Q0SK98X","ts":"1698678205.323289","message":{"bot_id":"B063CLKD7CL","type":"message","text":"Hi I am a bot that can post messages to any public channel.","user":"U0633HVJ0JJ","ts":"1698678205.323289","app_id":"A063CLGFG04","blocks":[{"type":"rich_text","block_id":"FxaJ1","elements":[{"type":"rich_text_section","elements":[{"type":"text","text":"Hi I am a bot that can post messages to any public channel."}]}]}],"team":"T05UB5W8H63","bot_profile":{"id":"B063CLKD7CL","app_id":"A063CLGFG04","name":"fluxcd","icons":{"image_36":"https:\/\/a.slack-edge.com\/80588\/img\/plugins\/app\/bot_36.png","image_48":"https:\/\/a.slack-edge.com\/80588\/img\/plugins\/app\/bot_48.png","image_72":"https:\/\/a.slack-edge.com\/80588\/img\/plugins\/app\/service_72.png"},"deleted":false,"updated":1698606041,"team_id":"T05UB5W8H63"}}}%
```
