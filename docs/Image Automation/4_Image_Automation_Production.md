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
kubectl get secrets --namespace fluxv2-tutorial-deployment-uat fluxv2-tutorial-deployment-secret -o yaml
```
```text
apiVersion: v1
data:
  identity: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JRzJBZ0VBTUJBR0J5cUdTTTQ5QWdFR0JTdUJCQUFpQklHZU1JR2JBZ0VCQkRERGNrVEdHTExQcEhrOWU3akkKT0swUnRScWVNc0taVjdqQk9nQVdxYXFXbllxWGtaMTNaUUpMbnRpSUpJOUFHbW1oWkFOaUFBUWlVbXFm
```

```bash
k get gitrepositories.source.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```
```text
NAME                                URL                                                            AGE   READY   STATUS
fluxv2-tutorial-deployment-source   ssh://git@github.com/mamdouni/fluxv2-tutorial-deployment.git   97s   True    stored artifact for revision 'stable@sha1:28f2cfcc12b4523416ea86fd575c25042acc8687'
```

Yes, we have the link now to the stable branch.

```bash
k get kustomizations.kustomize.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```
```text
NAME                                       AGE     READY   STATUS
fluxv2-tutorial-deployment-kustomization   2m22s   True    Applied revision: stable@sha1:28f2cfcc12b4523416ea86fd575c25042acc8687
```

The kustomization controller follows the source revision.

I tried to use the same image repo from ``fluxv2-tutorial-deployment`` but i had this ACL error :
```bash
k logs image-reflector-controller-666f9d9bf7-b89z8 -n flux-system
```
```text
{"level":"error","ts":"2023-05-29T14:00:49.563Z","msg":"failed to get the referred ImageRepository: access denied: 'fluxv2-tutorial-deployment/k8s-debugger-imagerepo' can't be accessed due to missing ACL labels on 'accessFrom'","controller":"imagepolicy","controllerGroup":"image.toolkit.fluxcd.io","controllerKind":"ImagePolicy","ImagePolicy":{"name":"k8s-debugger-policy","namespace":"fluxv2-tutorial-deployment-uat"},"namespace":"fluxv2-tutorial-deployment-uat","name":"k8s-debugger-policy","reconcileID":"4385bccb-70d4-40cb-b561-484d0e5938ea","error":"AccessDenied"}
```

I will create my own imagerepo for the uat.

```bash
kubectl get imagerepositories.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```
```text
NAME                     LAST SCAN              TAGS
k8s-debugger-imagerepo   2023-05-29T14:07:33Z   4
```

Image Repo still here. May be we need to move this to a shared namespace.

```bash
kubectl get imagepolicies.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```
```text
NAME                  LATESTIMAGE
k8s-debugger-policy   docker.io/mouhamedali/k8s-debugger:uat-202305291352-815a0fd
```

```bash
kubectl get imageupdateautomations.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```
```text
NAME                        LAST RUN
k8s-debugger-image-update   2023-05-29T14:08:39Z
```

Now, i will build a new ``uat`` or ``staging`` version. You can check from the repo in the references to know how to do it.
After building the new test release, i ended up with this tag ``uat-202305291410-815a0fd``.

```bash
kubectl get imagerepositories.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```
```text
NAME                     LAST SCAN              TAGS
k8s-debugger-imagerepo   2023-05-29T14:11:35Z   5
```

New tag has been scanned.


```bash
kubectl get imagepolicies.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```
```text
NAME                  LATESTIMAGE
k8s-debugger-policy   docker.io/mouhamedali/k8s-debugger:uat-202305291410-815a0fd
```

Nice, the image policy selected my new tag.

```bash
kubectl get imageupdateautomations.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment-uat
```
```text
NAME                        LAST RUN
k8s-debugger-image-update   2023-05-29T14:10:40Z
```

The image automation succeeded to push the new image tag with an awesome commit message :
- https://github.com/mamdouni/fluxv2-tutorial-deployment/commit/f9603e9f14c4db5ee7583a0f93f587427abc0647

After this, the kustomization controller will deploy these changes respecting the gitpods principales and the flux cd process.

Last check is on the deployed app :
![2023-05-29_16h21_34](https://github.com/mamdouni/fluxv2-tutorial/assets/61866853/421ccfe5-34a7-47b5-b89b-0ee23b34d5c8)

Works fine, let's do the production environment.

## References

- https://app.pluralsight.com/course-player?clipId=abfc7f29-f092-4a2e-ae0a-f5aab3ebac20
- https://app.pluralsight.com/course-player?clipId=bf402016-a213-40a0-a852-5859f6c90132
- https://fluxcd.io/flux/components/image/imageupdateautomations/#commit-message-template-data
- https://github.com/mamdouni/k8s-debugger