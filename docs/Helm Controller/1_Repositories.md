# Configuring Helm repository

In this demo we will configure a new helm repository souci using the github container registry.

The first thing that we will do is to add the helm notifications.

```bash 
$ cat notifications/alerts/test-flux-cd-google-bot-alert.yaml
```

```text
---
apiVersion: notification.toolkit.fluxcd.io/v1beta2
kind: Alert
metadata:
  name: test-flux-cd-google-bot-alert
  namespace: default
spec:
  eventSeverity: info
  eventSources:
  - kind: GitRepository
    name: '*'
  - kind: Kustomization
    name: '*'
  - kind: HelmRepository
    name: '*'
  providerRef:
    name: google-chat-provider
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

## References
- https://app.pluralsight.com/course-player?clipId=42e9cd18-6667-4ab5-bb8d-00966aa7b19c