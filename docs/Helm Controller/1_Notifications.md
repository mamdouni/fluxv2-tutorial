
# Helm Notifications

## Add a new namespace

Let's add a new namespace for the Helm demo :

```bash 
k create ns fluxv2-tutorial-deployment-helm
```

```bash 
k get ns
```

## Notifications

The first thing that we will do is to add the helm notifications.
Add a secret :

```bash
k create secret generic testfluxcd-google-chat-webhook-secret \
--namespace=fluxv2-tutorial-deployment-helm \
--from-file=address=./webhook-address.txt
```

PN : do not add blank spaces to the secret file please otherwise you'll have some strange errors.

Add a provider :

```bash
flux create alert-provider testfluxcd-google-chat-provider \
--type=googlechat \
--secret-ref=testfluxcd-google-chat-webhook-secret \
--channel='Test Flux CD' \
--username=FluxBot \
--namespace=fluxv2-tutorial-deployment-helm \
--export > notifications/providers/testfluxcd-google-chat-provider-helm.yaml
```

And an alert :

```bash
flux create alert testfluxcd-google-chat-alert \
--event-severity=info \
--event-source=HelmRepository/* \
--provider-ref=testfluxcd-google-chat-provider \
--namespace=fluxv2-tutorial-deployment-helm \
--export > notifications/alerts/testfluxcd-google-alert-helm.yaml
```

```bash
k get providers.notification.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-helm
```

```text
NAME                              AGE     READY   STATUS
testfluxcd-google-chat-provider   3m12s   True    Initialized
```


```bash
k get alerts.notification.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-helm
```

Few moments later :

```text
NAME                           AGE     READY   STATUS
testfluxcd-google-chat-alert   6m31s   True    Initialized
```