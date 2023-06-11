# Notifications

To receive notifications, you have to declare a provider like Google chat , Slack ...

After that, you can declare an Alert resource which will filter on the type of alerts you want (git repo alerts, kustomization alerts) and the severity of this alert.
The alert will use the declared provider to sent the notification.

Check these videos for a full demo :
- https://app.pluralsight.com/course-player?clipId=81e91c49-4686-46d1-9ff3-18f99ca431f1
- https://app.pluralsight.com/course-player?clipId=daf44986-e124-4a96-bb42-68aa87df104a

## Creating a bot

For our case, we will use a google chat bot to send alerts and notifications to inform the team about a release deployment for example.

Check on the net on how to create a google chat bot in a google space.
- https://youtu.be/Aeyzq-i7GTw
- https://youtu.be/_GtD_xIllJU

After creating the bot, its webhook url will be shown so copy it.
![2023-05-30_22h10_42](https://github.com/mamdouni/fluxv2-tutorial/assets/61866853/e93e7098-f12d-472b-81fa-d4a255ae45a8)

Now, you can add it to a new text file :
```bash
cat > webhook-address.txt
```

And use it to create the secret :
```bash
k create secret generic google-chat-webhook-secret --from-file=address=./webhook-address.txt
```

```text
rm webhook-address.txt
```

## Creating a Provider

Let's create two new directories.

```bash
mkdir -p notifications/{providers,alerts}
```

And now let's generate a provider for Google Chat.

```bash
flux create alert-provider google-chat-provider \
--type=googlechat \
--secret-ref=google-chat-webhook-secret \
--channel='Test Flux CD' \
--username=FluxBot \
--namespace=default \
--export > notifications/providers/google-chat-provider.yaml
```

## Creating a Notification

And now a notification :

```bash
flux create alert test-flux-cd-google-bot-alert \
--event-severity=info \
--event-source=GitRepository/*,Kustomization/* \
--provider-ref=google-chat-provider \
--namespace=default \
--export > notifications/alerts/test-flux-cd-google-bot-alert.yaml
```

## Check it out

After pushing the files and waiting for the reconciliation.

```bash
k get providers.notification.toolkit.fluxcd.io
```

```bash
k get alerts.notification.toolkit.fluxcd.io
```

## References
- https://fluxcd.io/flux/components/notification/provider/#google-chat
- https://app.pluralsight.com/course-player?clipId=81e91c49-4686-46d1-9ff3-18f99ca431f1
- https://app.pluralsight.com/course-player?clipId=daf44986-e124-4a96-bb42-68aa87df104a
