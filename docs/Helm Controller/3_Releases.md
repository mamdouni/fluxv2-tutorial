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
--chart=k8sdebugger \
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
```

Let's check if there are a created charts :

```bash
k get helmcharts.source.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-helm
```

```text
NAME                                                   CHART         VERSION   SOURCE KIND      SOURCE NAME                 AGE    READY   STATUS
fluxv2-tutorial-deployment-helm-k8s-debugger-release   k8sdebugger   *         HelmRepository   mamdouni-ghcr-helm-source   133m   False   chart pull error: failed to get chart version for remote reference: could not get tags for "k8sdebugger": could not fetch tags for "oci://ghcr.io/mamdouni/charts/k8sdebugger": GET "https://ghcr.io/v2/mamdouni/charts/k8sdebugger/tags/list": unexpected status code 404: name unknown: repository name not known to registry
```

```bash
$ k describe helmcharts.source.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-helm
```

```text
Events:
  Type     Reason          Age                From               Message
  ----     ------          ----               ----               -------
  Warning  ChartPullError  14m (x7 over 93m)  source-controller  chart pull error: failed to get chart version for remote reference: could not get tags for "k8s-debugger": could not fetch tags for "oci://ghcr.io/mamdouni/charts/k8s-debugger": GET "https://ghcr.io/v2/mamdouni/charts/k8s-debugger/tags/list": unexpected status code 404: name unknown: repository name not known to registry
  Warning  ChartPullError  6m12s              source-controller  chart pull error: failed to get chart version for remote reference: could not get tags for "k8sdebugger": could not fetch tags for "oci://ghcr.io/mamdouni/charts/k8sdebugger": GET "https://ghcr.io/v2/mamdouni/charts/k8sdebugger/tags/list": unexpected status code 404: name unknown: repository name not known to registry
```

Well, everything seems to be well configured but i'm facing this error and i don't know why (even after making the package public).
Not much articles about this issue on the net -_- .

I tried to pull the chart and it works fine ...

```bash
$ podman run --rm -it --entrypoint ash alpine/helm

$ helm pull oci://ghcr.io/mamdouni/k8sdebugger

Pulled: ghcr.io/mamdouni/k8sdebugger:1.0.0
Digest: sha256:c2909212f0fe0d11aff0ed27d381deec34ce329a350d6dacb2a1c7a4c09223b6

$ helm show chart oci://ghcr.io/mamdouni/k8sdebugger

Pulled: ghcr.io/mamdouni/k8sdebugger:1.0.0
Digest: sha256:c2909212f0fe0d11aff0ed27d381deec34ce329a350d6dacb2a1c7a4c09223b6
apiVersion: v2
appVersion: 1.0.2
description: A Helm chart for K8s Debugger Application
name: k8sdebugger
type: application
version: 1.0.0
```

It seems to be an issue with the github api.


To check if the release has been installed :

```bash
k get all -n fluxv2-tutorial-deployment-helm
```

check the installed releases :

```bash
helm list
```

If you check the notifications, you'll seel that two resources **HelmRelease** and **HelmChart** has been added.

## References
- https://app.pluralsight.com/course-player?clipId=a19342df-716a-467b-9118-e792a71e47cd
- https://app.pluralsight.com/course-player?clipId=3970b98e-bead-44b2-bcb5-74b257a3cf90
- https://fluxcd.io/flux/components/source/helmcharts/