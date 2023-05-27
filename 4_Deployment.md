# Deployment

Now let's deploy our first application which is a simple nginx server.
First of all, lets create our flux source custom resource wich binds the flux to the github repo declared in the last demo.

Let's create two directories to bette organize our flux repo. From the root directory :

```bash
mkdir {sources,kustomizations}
```

```bash
ls -ltr
```

```log
drwxr-xr-x 1 Mohamed-Ali 197121    0 mai   27 20:11 flux-system/
drwxr-xr-x 1 Mohamed-Ali 197121    0 mai   27 22:22 sources/
drwxr-xr-x 1 Mohamed-Ali 197121    0 mai   27 22:22 kustomizations/
```

## Git Customs Resource

Now let's create the git flux source :
```bash
flux create source git fluxv2-tutorial-deployment \
--url=ssh://git@github.com/mamdouni/fluxv2-tutorial-deployment.git \
--branch=main \
--secret-ref=fluxv2-tutorial-deployment \
--namespace=fluxv2-tutorial-deployment \
--export > sources/fluxv2-tutorial-deployment-source.yaml
```

For the namspace :
```log
 -n, --namespace string               If present, the namespace scope for this CLI request (default "flux-system")
```

Let's see the file content :
```bash
cat sources/fluxv2-tutorial-deployment-source.yaml
```

```yaml
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: fluxv2-tutorial-deployment
  namespace: fluxv2-tutorial-deployment
spec:
  interval: 1m0s
  ref:
    branch: main
  secretRef:
    name: fluxv2-tutorial-deployment
  url: ssh://git@github.com/mamdouni/fluxv2-tutorial-deployment.git
```


## Kustomization Customs Resource

```bash
flux create kustomization fluxv2-tutorial-deployment \
--source=GitRepository/fluxv2-tutorial-deployment.fluxv2-tutorial-deployment
--path=./ \
--prune=true \
--target-namespace=fluxv2-tutorial-deployment \
--namespace=fluxv2-tutorial-deployment \
--verbose \
--export > kustomizations/fluxv2-tutorial-deployment-kustomization.yaml
```

- GitRepository/fluxv2-tutorial-deployment.fluxv2-tutorial-deployment : GitRepository/${The source name}.${The namespace}
- path : The path to look for kustomizations and manifest yaml files
- prune : inform the kustomization controller to garbage collect any resources that are removed from the deployment repo
- target-namespace : this will override the namespaces declared in the manifest files
- namespace : the home for the kustomization resource will be also the fluxv2-tutorial-deployment namespace

Now let's create kustomizations files :

```bash
cd sources && kustomize create --autodetect
```

```bash
cd .. && cd kustomizations && kustomize create --autodetect && ..
```