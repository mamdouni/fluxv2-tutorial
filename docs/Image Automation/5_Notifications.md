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

## Production Alerts

Add a secret :

```bash
k create secret generic google-chat-webhook-secret --namespace=fluxv2-tutorial-deployment --from-file=address=./webhook-address.txt
```

```bash
rm webhook-address.txt
```

Add a provider :

```bash
flux create alert-provider google-chat-provider-production \
--type=googlechat \
--secret-ref=google-chat-webhook-secret \
--channel='Test Flux CD' \
--username=FluxBot \
--namespace=fluxv2-tutorial-deployment \
--export > notifications/providers/google-chat-provider-production.yaml
```

And an alert :

```bash
flux create alert test-flux-cd-google-production-alert \
--event-severity=info \
--event-source=GitRepository/*,Kustomization/* \
--provider-ref=google-chat-provider-production \
--namespace=fluxv2-tutorial-deployment-uat \
--export > notifications/alerts/test-flux-cd-google-production-alert.yaml
```

## Check it out

### Staging

After pushing the files and waiting for the reconciliation.

```bash
k get providers.notification.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```

```text
NAME                           AGE   READY   STATUS
google-chat-provider-staging   48s   True    Initialized
```

```bash
k get alerts.notification.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```

```text
NAME                                AGE   READY   STATUS
test-flux-cd-google-staging-alert   29m   True    Initialized
```

Well, it took two much to initialize with an error :


```bash
k describe alerts.notification.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```

```text
  Observed Generation:     2
Events:
  Type     Reason     Age    From                     Message
  ----     ------     ----   ----                     -------
  Warning  Failed     10m    notification-controller  provider fluxv2-tutorial-deployment-uat/google-chat-provider-staging is not ready
  Normal   Succeeded  4m47s  notification-controller  Reconciliation finished
```

![2023-06-11_14h47_57](https://github.com/mamdouni/fluxv2-tutorial/assets/61866853/0c6ac623-67f5-4a36-940b-e3e2f475eab6)

### Production

Well, same commands but with a different namespace.


```bash
k get providers.notification.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```

```bash
k get alerts.notification.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```

## References
- https://fluxcd.io/flux/components/notification/provider/#google-chat
- https://fluxcd.io/flux/components/notification/alert/#writing-an-alert-spec
- https://app.pluralsight.com/course-player?clipId=81e91c49-4686-46d1-9ff3-18f99ca431f1
- https://app.pluralsight.com/course-player?clipId=daf44986-e124-4a96-bb42-68aa87df104a
