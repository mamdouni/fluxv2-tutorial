# Scan Images

## Introduction

The resource to use to follow the docker image evolutions is called Image Repository on flux. This resource tracks the new updates on an image.

Let's strat by creating a new directory to hold our image repositories :

```bash
mkdir imagerepos
```

And let's create a new image repository to track our ``k8s-debugger`` image.

By the way, you can find the app git repo here :
- https://github.com/mamdouni/k8s-debugger

And it's docker hub here :
- https://hub.docker.com/repository/docker/mouhamedali/k8s-debugger/tags?page=1&ordering=last_updated


```bash
flux create image repository k8s-debugger-imagerepo \
--image=docker.io/mouhamedali/k8s-debugger \
--interval=5m \
--namespace=fluxv2-tutorial-deployment \
--export > imagerepos/k8s-debugger-imagerepo.yaml
```

each five minutes, our imagerepo will pull the last tags of our image. For now, this is reponsability of our resource. We will use
other reources later to perform actions on these tags.

The result :

```yaml
---
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: k8s-debugger-imagerepo
  namespace: fluxv2-tutorial-deployment
spec:
  image: docker.io/mouhamedali/k8s-debugger
  interval: 5m0s
```

dont forget to add the kustomization file.

```bash
kustomize create --autodetect
```

kubectl get imagerepositories.image.toolkit.fluxcd.io

## References
- https://app.pluralsight.com/course-player?clipId=a0387786-122e-403f-866d-946e3c99a9ce
- https://app.pluralsight.com/course-player?clipId=3b3b2853-3cc6-457c-994a-3114d2a7fe54