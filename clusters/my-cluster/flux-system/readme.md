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

### Slack

#### Send a message with curl

Test if you can post a message in a specific Slack channel by curl.
Follow the instructions from the article [Posting messages using curl](https://api.slack.com/tutorials/tracks/posting-messages-with-curl).


```
❯ curl -H "Authorization: Bearer xoxb-not-a-real-token-this-will-not-work" https://slack.com/api/auth.test
{"ok":true,"url":"https:\/\/germanium-workspace.slack.com\/","team":"Germanium","user":"fluxcd2","team_id":"T05UB5W8H63","user_id":"U0633HVJ0JJ","bot_id":"B063CLKD7CL","is_enterprise_install":false}%
```


```
❯ curl -d "text=Hi I am a bot that can post messages to any public channel." -d "channel=fluxcd" -H "Authorization: Bearer token=xoxb-not-a-real-token-this-will-not-work" -X POST https://slack.com/api/chat.postMessage

{"ok":true,"channel":"C062Q0SK98X","ts":"1698678205.323289","message":{"bot_id":"B063CLKD7CL","type":"message","text":"Hi I am a bot that can post messages to any public channel.","user":"U0633HVJ0JJ","ts":"1698678205.323289","app_id":"A063CLGFG04","blocks":[{"type":"rich_text","block_id":"FxaJ1","elements":[{"type":"rich_text_section","elements":[{"type":"text","text":"Hi I am a bot that can post messages to any public channel."}]}]}],"team":"T05UB5W8H63","bot_profile":{"id":"B063CLKD7CL","app_id":"A063CLGFG04","name":"fluxcd","icons":{"image_36":"https:\/\/a.slack-edge.com\/80588\/img\/plugins\/app\/bot_36.png","image_48":"https:\/\/a.slack-edge.com\/80588\/img\/plugins\/app\/bot_48.png","image_72":"https:\/\/a.slack-edge.com\/80588\/img\/plugins\/app\/service_72.png"},"deleted":false,"updated":1698606041,"team_id":"T05UB5W8H63"}}}%
```
