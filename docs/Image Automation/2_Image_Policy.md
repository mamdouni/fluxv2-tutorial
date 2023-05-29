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


Let's strat by creating a new directory to hold our image repositories :

```bash
mkdir imagepolicies
```

```bash
flux create image policy k8s-debugger-uat-policy \
--image-ref=k8s-debugger-imagerepo \
--select-numeric=asc \
	--filter-regex='^uat-(?P<ts>[0-9]{12})-[a-f0-9]{7}$' \
	--filter-extract='$ts' \
--namespace=fluxv2-tutorial-deployment \
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
  name: k8s-debugger-uat-policy
  namespace: fluxv2-tutorial-deployment
spec:
  filterTags:
    extract: $ts
    pattern: ^uat-(?P<ts>[0-9]{12})-[a-f0-9]{7}$
  imageRepositoryRef:
    name: k8s-debugger-imagerepo
  policy:
    numerical:
      order: asc
```


## References

- https://regex101.com/
- https://semver.org/
- https://fluxcd.io/flux/cmd/flux_create_image_policy/
- https://app.pluralsight.com/course-player?clipId=73d1338b-2fac-4673-bacc-903ba08720fb
- https://app.pluralsight.com/course-player?clipId=33855d6d-8ff1-47b3-b68c-eb365256254e
- https://github.com/mamdouni/k8s-debugger
- https://hub.docker.com/repository/docker/mouhamedali/k8s-debugger/tags?page=1&ordering=last_updated

