# Image Automation

## Introduction

The job of the image update automation is reflect the image updates to the sources (git in our case).
This means, that each time flux perform an image update, he will reflect this on your git manifest files by pushing the new tag to it.

## Staging Environment

Well, as a first step, we will handle the update of the ``uat`` tag. That's why we need a new configuration for the staging environment.

### New namespace

```bash
kubectl create ns fluxv2-tutorial-deployment-uat
```

### New git source

```bash
flux create secret git fluxv2-tutorial-deployment-secret \
--url=ssh://git@github.com/mamdouni/fluxv2-tutorial-deployment.git \
--namespace=fluxv2-tutorial-deployment-uat
```

Add the public key to the github repo.

```bash
flux create source git fluxv2-tutorial-deployment-source \
--url=ssh://git@github.com/mamdouni/fluxv2-tutorial-deployment.git \
--branch=main \
--secret-ref=fluxv2-tutorial-deployment-secret \
--namespace=fluxv2-tutorial-deployment-uat \
--export > sources/fluxv2-tutorial-deployment-source-uat.yaml
```

### New kusomization controller

```bash
flux create kustomization fluxv2-tutorial-deployment-kustomization \
--source=GitRepository/fluxv2-tutorial-deployment-source.fluxv2-tutorial-deployment-uat
--path=. \
--prune=true \
--target-namespace=fluxv2-tutorial-deployment-uat \
--namespace=fluxv2-tutorial-deployment-uat \
--export > kustomizations/fluxv2-tutorial-deployment-kustomization-uat.yaml
```

### Automate image update

First of all, you need to add the marker to image you need to update by specifing the namespace and the policy name :
- https://github.com/mamdouni/fluxv2-tutorial-deployment/blob/28f2cfcc12b4523416ea86fd575c25042acc8687/k8s-debugger.yaml#LL17C8-L17C8

Now let's create our image update resource

```bash
flux create image update k8s-debugger-image-update \
--git-repo-ref=fluxv2-tutorial-deployment-source \
--git-repo-path=. \
--checkout-branch=stable \
--push-branch=stable \
--author-name=flux \
--author-email=flux@users.noreply.github.com \
--commit-template="$(cat ./docs/templates/msg_template)" \
--namespace=fluxv2-tutorial-deployment-uat \
--export > imageautos/k8s-debugger-uat-automation.yaml
```

- checkout branch : the branch where to look for manifest files
- checkout push : the branch to which flux will apply the modification. a new branch for example to check what flux did to the checkout branch.

The result :

```bash
cat imageautos/k8s-debugger-uat-automation.yaml
```

```yaml
---
---
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: k8s-debugger-image-update
  namespace: fluxv2-tutorial-deployment-uat
spec:
  git:
    checkout:
      ref:
        branch: stable
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
      branch: stable
  interval: 1m0s
  sourceRef:
    kind: GitRepository
    name: fluxv2-tutorial-deployment-source
  update:
    path: .
    strategy: Setters
```



## References

- https://app.pluralsight.com/course-player?clipId=abfc7f29-f092-4a2e-ae0a-f5aab3ebac20
- https://app.pluralsight.com/course-player?clipId=bf402016-a213-40a0-a852-5859f6c90132
- https://fluxcd.io/flux/components/image/imageupdateautomations/#commit-message-template-data