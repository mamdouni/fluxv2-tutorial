# Image Automation on the Staging environement

## Introduction

What will be different from the ``Staging`` configuration is only the image policy which will use a SemVer.

## Image Policy

```bash
flux create image policy k8s-debugger-policy \
--image-ref=k8s-debugger-imagerepo \
--select-semver='>=1.x.x' \
--namespace=fluxv2-tutorial-deployment \
--export > imagepolicies/k8s-debugger-policy.yaml
```

The result :

```bash
cat imagepolicies/k8s-debugger-uat-policy.yaml
```

```yaml
---
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: k8s-debugger-policy
  namespace: fluxv2-tutorial-deployment
spec:
  imageRepositoryRef:
    name: k8s-debugger-imagerepo
  policy:
    semver:
      range: '>=1.x.x'

```

well this is not sufficient, check the real file from the sources to get a more updated version.

## Automate image update

First of all, you need to add the marker to image you need to update by specifing the namespace and the policy name :
- https://github.com/mamdouni/fluxv2-tutorial-deployment/blob/7ae304d0e8d4e611e46cadb4e7fa1082c15b6eeb/k8s-debugger.yaml#L17

Now let's create our image update resource

```bash
flux create image update k8s-debugger-image-update \
--git-repo-ref=fluxv2-tutorial-deployment-source \
--git-repo-path=. \
--checkout-branch=main \
--push-branch=main \
--author-name=flux \
--author-email=flux@users.noreply.github.com \
--commit-template="$(cat ./docs/templates/msg_template)" \
--namespace=fluxv2-tutorial-deployment \
--export > imageautos/k8s-debugger-automation.yaml
```

The result :

```bash
cat imageautos/k8s-debugger-automation.yaml
```

```yaml
---
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: k8s-debugger-image-update
  namespace: fluxv2-tutorial-deployment
spec:
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        email: flux@users.noreply.github.com
        name: flux
      messageTemplate: "Automated image update\r\n\r\nAutomation name: {{ .AutomationObject
        }}\r\n\r\nFiles:\r\n{{ range $filename, $_ := .Updated.Files -}}\r\n- {{ $filename
        }}\r\n{{ end -}}\r\n\r\nObjects:\r\n{{ range $resource, $_ := .Updated.Objects
        -}}\r\n- {{ $resource.Kind }} {{ $resource.Name }}\r\n{{ end -}}\r\n\r\nImages:\r\n{{
        range .Updated.Images -}}\r\n- {{.}}\r\n{{ end -}}    "
    push:
      branch: main
  interval: 1m0s
  sourceRef:
    kind: GitRepository
    name: fluxv2-tutorial-deployment-source
  update:
    path: .
    strategy: Setters
```

## Check it out
```bash
kubectl get secrets --namespace fluxv2-tutorial-deployment fluxv2-tutorial-deployment-secret -o yaml
```
```text
apiVersion: v1
data:
  identity: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JRzJBZ0VBTUJBR0J5cUdTTTQ5QWdFR0JTdUJCQUFpQklHZU1JR2JBZ0VCQkRBL3cvS3B1eW91YXFoSmIrSUoKTTdxWlU5dU5wQ3huYmoweDBOWVpJY3QrODk1ZkFDZVkxT01EK3c3RlJnaHE4UytoWkFOaUFBUXk5d0N3Wm
```

```bash
k get gitrepositories.source.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```
```text
NAME                                URL                                                            AGE   READY   STATUS
fluxv2-tutorial-deployment-source   ssh://git@github.com/mamdouni/fluxv2-tutorial-deployment.git   24h   True    stored artifact for revision 'main@sha1:7ae304d0e8d4e611e46cadb4e7fa1082c15b6eeb'
```

```bash
k get kustomizations.kustomize.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```
```text
NAME                                       AGE   READY   STATUS
fluxv2-tutorial-deployment-kustomization   24h   True    Applied revision: main@sha1:7ae304d0e8d4e611e46cadb4e7fa1082c15b6eeb
```

The kustomization controller follows the source revision.

```bash
kubectl get imagerepositories.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```
```text
NAME                     LAST SCAN              TAGS
k8s-debugger-imagerepo   2023-05-29T14:50:12Z   6
```

```bash
kubectl get imagepolicies.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```
```text
NAME                  LATESTIMAGE
k8s-debugger-policy   docker.io/mouhamedali/k8s-debugger:1.0.0-202305291659-815a0fd
```

```bash
kubectl get imageupdateautomations.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```
```text
NAME                        LAST RUN
k8s-debugger-image-update   2023-05-29T17:04:22Z
```

Now, i will build a new ``corporate``. You can check from the repo in the references to know how to do it.
After building the new test release, i ended up with this tag ``1.1.0-202305291710-014c11f``.

```bash
kubectl get imagerepositories.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```
```text
NAME                     LAST SCAN              TAGS
k8s-debugger-imagerepo   2023-05-29T17:14:03Z   8
```

New tag has been scanned. The full list of our tags for the corporate version are as below :
```text
1.0.0-202305291659-815a0fd
1.1.0-202305291710-014c11f
1.0.2-202305291713-014c11f
```

By the way, the ``1.0.2-202305291713-014c11f`` has been build after the ``1.1.0-202305291710-014c11f`` but flux will not select it as it not the max of the versions.

```bash
kubectl get imagepolicies.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```
```text
NAME                  LATESTIMAGE
k8s-debugger-policy   docker.io/mouhamedali/k8s-debugger:1.1.0-202305291710-014c11f
```

Nice, the image policy selected my new tag.

```bash
kubectl get imageupdateautomations.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```
```text
NAME                        LAST RUN
k8s-debugger-image-update   2023-05-29T17:16:30Z
```

The image automation succeeded to push the new image tag with an awesome commit message :
- https://github.com/mamdouni/fluxv2-tutorial-deployment/commit/495f06f509004766936c21e6f6f6432dedc067e9

After this, the kustomization controller will deploy these changes respecting the gitpods principales and the flux cd process.

Last check is on the deployed app :
![2023-05-29_19h19_37](https://github.com/mamdouni/fluxv2-tutorial-deployment/assets/61866853/15201836-7e5c-460d-b86d-989776d4f5d8)

## Debug

To debug image policy error, use this command :
```bash
k logs image-reflector-controller-666f9d9bf7-b89z8 -n flux-system
```
## References

- https://app.pluralsight.com/course-player?clipId=abfc7f29-f092-4a2e-ae0a-f5aab3ebac20
- https://app.pluralsight.com/course-player?clipId=bf402016-a213-40a0-a852-5859f6c90132
- https://fluxcd.io/flux/components/image/imageupdateautomations/#commit-message-template-data
- https://github.com/mamdouni/k8s-debugger
- https://regex101.com/r/Ly7O1x/3/