# Configuring Helm repository

In this demo we will configure a new helm repository souci using the github container registry.

## Helm Alerts

The first thing that we will do is to add the helm notifications.
Add a secret :

```bash
k create secret generic testfluxcd-google-chat-webhook-secret \
--namespace=fluxv2-tutorial-deployment-helm \
--from-file=address=./webhook-address.txt
```

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

## Add a new namespace

Let's add a new namespace for the Helm demo :

```bash 
k create ns fluxv2-tutorial-deployment-helm
```

```bash 
k get ns
```

## Add a secret to your oci registry

Add a secret :

```bash 
k create secret docker-registry ghrc-mamdouni-charts-auth \
--docker-server=ghrc.io \
--docker-username=mamdouni \
--docker-password="$(cat ./ghrc-token)" \
--namespace=fluxv2-tutorial-deployment-helm
```

You can generate a classic token from here :
- https://github.com/settings/tokens

with these permissions :
````text
read:packages
write:packages
delete:packages
````

```bash 
$ k get secret -n fluxv2-tutorial-deployment-helm
```

```text
NAME                        TYPE                             DATA   AGE
ghrc-mamdouni-charts-auth   kubernetes.io/dockerconfigjson   1      23s
```

## Add an Helm Source Controller

```bash
flux create source helm mamdouni-ghrc-helm-source \
--url=oci://ghrc.io/mamdouni/charts \
--secret-ref=ghrc-mamdouni-charts-auth \
--namespace=fluxv2-tutorial-deployment-helm \
--export > sources/mamdouni-ghrc-helm-source.yaml
```

## Check it out

```bash
$ k get helmrepositories.source.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-helm
```

```text
NAME                        URL                             AGE   READY   STATUS
mamdouni-ghrc-helm-source   oci://ghrc.io/mamdouni/charts   53s   True    Helm repository is ready
```

## References
- https://app.pluralsight.com/course-player?clipId=42e9cd18-6667-4ab5-bb8d-00966aa7b19c