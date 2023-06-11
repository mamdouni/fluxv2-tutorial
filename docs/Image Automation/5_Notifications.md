# Notifications

As for the default namespace notifications, here we will add notifications for the two staging and production namespaces.

## Creating a bot

Check the docs to create a google chat room and a webhook.

- [Notifications](../Automated%20Deployment/2_Notifications.md)

## Staging Alerts

And a notification :

```bash
flux create alert test-flux-cd-google-staging-alert \
--event-severity=info \
--event-source=GitRepository/*,Kustomization/* \
--provider-ref=google-chat-provider \
--namespace=fluxv2-tutorial-deployment-uat \
--export > notifications/alerts/test-flux-cd-google-staging-alert.yaml
```

## Check it out

After pushing the files and waiting for the reconciliation.

```bash
k get providers.notification.toolkit.fluxcd.io
```

```text
NAME                   AGE     READY   STATUS
google-chat-provider   4m12s   True    Initialized
```

```bash
k get alerts.notification.toolkit.fluxcd.io
```

```text
NAME                            AGE     READY   STATUS
test-flux-cd-google-bot-alert   4m26s   True    Initialized
```

## References
- https://fluxcd.io/flux/components/notification/provider/#google-chat
- https://app.pluralsight.com/course-player?clipId=81e91c49-4686-46d1-9ff3-18f99ca431f1
- https://app.pluralsight.com/course-player?clipId=daf44986-e124-4a96-bb42-68aa87df104a
