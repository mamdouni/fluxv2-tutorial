# Helm Releases

Now, let's use the created helm repository to create the helm release object.

## Define parameters

Before creating the release, let's define our chart parameters.

```bash 
mkdir helmreleases
```

```bash 
echo "replicaCount: 3" > ./helmreleases/values.yaml
```

By the way, we are using this Chart :
- https://github.com/mamdouni/k8s-debugger/pkgs/container/k8s-debugger

## Create the helm release

Let's add the release now :
```bash
flux create helmrelease k8s-debugger-release \
--source=HelmRepository/mamdouni-ghcr-helm-source \
--chart=k8s-debugger \
--values=./helmreleases/values.yaml \
--namespace=fluxv2-tutorial-deployment-helm \
--export > ./helmreleases/k8s-debugger-hr.yaml
```

```bash
rm ./helmreleases/values.yaml
```

## Check it out

After pushing the files and waiting for the reconciliation.

```bash
k get helmreleases.helm.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-helm
```

```text
NAME                   AGE     READY   STATUS
google-chat-provider   4m12s   True    Initialized
```

Let's check if there are a created charts :

```bash
k get helmcharts.source.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-helm
```

```text
NAME                   AGE     READY   STATUS
google-chat-provider   4m12s   True    Initialized
```

## References
- https://app.pluralsight.com/course-player?clipId=a19342df-716a-467b-9118-e792a71e47cd
- https://app.pluralsight.com/course-player?clipId=3970b98e-bead-44b2-bcb5-74b257a3cf90
- https://fluxcd.io/flux/components/source/helmcharts/