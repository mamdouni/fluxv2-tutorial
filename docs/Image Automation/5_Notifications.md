# Notifications

As for the default namespace notifications, here we will add notifications for the two staging and production namespaces.

## Creating a bot

Check the docs to create a google chat room and a webhook.

- [Notifications](../Automated%20Deployment/2_Notifications.md)

## Staging Alerts

We can't use the provider from the default namespace because i think it's impossible as in the documentation said : 
- https://fluxcd.io/flux/components/notification/alert/#writing-an-alert-spec

Add a secret :

```bash
k create secret generic google-chat-webhook-secret --namespace=fluxv2-tutorial-deployment-uat --from-file=address=./webhook-address.txt
```

```bash
rm webhook-address.txt
```

Add a provider :

```bash
flux create alert-provider google-chat-provider-staging \
--type=googlechat \
--secret-ref=google-chat-webhook-secret \
--channel='Test Flux CD' \
--username=FluxBot \
--namespace=fluxv2-tutorial-deployment-uat \
--export > notifications/providers/google-chat-provider-staging.yaml
```

And an alert :

```bash
flux create alert test-flux-cd-google-staging-alert \
--event-severity=info \
--event-source=GitRepository/*,Kustomization/* \
--provider-ref=google-chat-provider-staging \
--namespace=fluxv2-tutorial-deployment-uat \
--export > notifications/alerts/test-flux-cd-google-staging-alert.yaml
```

## Check it out

After pushing the files and waiting for the reconciliation.

```bash
k get alerts.notification.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```

```text
NAME                            AGE     READY   STATUS
test-flux-cd-google-bot-alert   4m26s   True    Initialized
```

## References
- https://fluxcd.io/flux/components/notification/provider/#google-chat
- https://fluxcd.io/flux/components/notification/alert/#writing-an-alert-spec
- https://app.pluralsight.com/course-player?clipId=81e91c49-4686-46d1-9ff3-18f99ca431f1
- https://app.pluralsight.com/course-player?clipId=daf44986-e124-4a96-bb42-68aa87df104a
