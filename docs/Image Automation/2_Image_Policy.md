# Image Policy

## Introduction

After creating an image repo we need to tell flux which image is the last one and has to be updated.
To achieve this, we need to add a new resource called Image Pollicy.

This resource and the image repository resource are managed by the ``image reflector controller`` which we add as an extra component on the installation.

We have several policies for selecting the image tag :
- Alphabetical  :   Tags are sorted in alphabetical order before last tag is selected
- Numerical     :   Tags are sorted in numerical order before last tag is selected
- SemVer        :   Tags sorted according to SemVer constraints before last tag is selected

Check more details here :

- https://app.pluralsight.com/course-player?clipId=73d1338b-2fac-4673-bacc-903ba08720fb


Let's start by creating a new directory to hold our image repositories :

```bash
mkdir imagepolicies
```

and a new ns :
```bash
kubectl create ns fluxv2-tutorial-deployment-uat
```

Now the policy :

```bash
flux create image policy k8s-debugger-uat-policy \
--image-namespace=k8s-debugger-imagerepo \
--select-numeric=asc \
	--filter-regex='^uat-(?P<ts>[0-9]{12})-[a-f0-9]{7}$' \
	--filter-extract='$ts' \
--namespace=fluxv2-tutorial-deployment-uat \
--export > imagepolicies/k8s-debugger-uat-policy.yaml
```

This will extract the build time from the image tag. You can test your regex here :
- https://regex101.com/

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
  namespace: fluxv2-tutorial-deployment-uat
spec:
  filterTags:
    extract: $ts
    pattern: ^uat-(?P<ts>[0-9]{12})-[a-f0-9]{7}$
  imageRepositoryRef:
    name: k8s-debugger-imagerepo
    namespace: fluxv2-tutorial-deployment
  policy:
    numerical:
      order: asc
```

dont forget to add the kustomization file.

```bash
kustomize create --autodetect
```
## Check out

After pushing, check if the image repo has been created :

```bash
kubectl get imagepolicies.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```

```text
NAME                      LATESTIMAGE
k8s-debugger-uat-policy   docker.io/mouhamedali/k8s-debugger:uat-202305291038-a2726e1
```

As you can see, we have the last tag from our k8s-debugger. Let's build another version and re-check the version.
You can check the app repo to build a new uat (or stage) version.
The build action will be here :
- https://github.com/mamdouni/k8s-debugger/actions

After running, a new tag will be avaialable here :
- https://hub.docker.com/repository/docker/mouhamedali/k8s-debugger/tags?page=1&ordering=last_updated

Well, the job has succeed and the new image tag is ``uat-202305291203-815a0fd``.

Now let's check the flux tag number :

```bash
kubectl get imagepolicies.image.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```

```text
NAME                      LATESTIMAGE
k8s-debugger-uat-policy   docker.io/mouhamedali/k8s-debugger:uat-202305291203-815a0fd
```

Yeah, and if we check the same thing from kubernetes :

```bash
kubectl get deployments.apps k8s-debugger-deployment -n fluxv2-tutorial-deployment -o yaml
```

```text
image: mouhamedali/k8s-debugger
```

Well, Nothing happens because flux can't be sure which component to update. Imagine that you have several components that has the same image. To make it easy to flux to determine which resource to update, you need to use markers.

## References

- https://regex101.com/
- https://semver.org/
- https://fluxcd.io/flux/cmd/flux_create_image_policy/
- https://app.pluralsight.com/course-player?clipId=73d1338b-2fac-4673-bacc-903ba08720fb
- https://app.pluralsight.com/course-player?clipId=33855d6d-8ff1-47b3-b68c-eb365256254e
- https://github.com/mamdouni/k8s-debugger
- https://hub.docker.com/repository/docker/mouhamedali/k8s-debugger/tags?page=1&ordering=last_updated
- https://fluxcd.io/flux/guides/sortable-image-tags/

