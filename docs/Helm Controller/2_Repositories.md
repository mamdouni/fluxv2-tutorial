# Configuring Helm repository

In this demo we will configure a new helm repository souci using the github container registry.

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