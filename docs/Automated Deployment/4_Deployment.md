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
flux create source git fluxv2-tutorial-deployment-source \
--url=ssh://git@github.com/mamdouni/fluxv2-tutorial-deployment.git \
--branch=main \
--secret-ref=fluxv2-tutorial-deployment-secret \
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
    name: fluxv2-tutorial-deployment-secret
  url: ssh://git@github.com/mamdouni/fluxv2-tutorial-deployment.git
```


## Kustomization Customs Resource

```bash
flux create kustomization fluxv2-tutorial-deployment-kustomization \
--source=GitRepository/fluxv2-tutorial-deployment-source.fluxv2-tutorial-deployment
--path=./ \
--prune=true \
--target-namespace=fluxv2-tutorial-deployment \
--namespace=fluxv2-tutorial-deployment \
--verbose \
--export > kustomizations/fluxv2-tutorial-deployment-kustomization.yaml
```

- GitRepository/fluxv2-tutorial-deployment-source.fluxv2-tutorial-deployment : GitRepository/${The source name}.${The namespace}
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

## Check it out

To force the flux reconciliation, you can use this command :
```bash
flux reconcile source git flux-system && flux reconcile kustomization flux-system
```

Check the nginx deployment :
```bash
k get all -n fluxv2-tutorial-deployment
```

```log
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-85996f8dbd-lfpm8   1/1     Running   0          88s
pod/nginx-deployment-85996f8dbd-xdb66   1/1     Running   0          88s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2/2     2            2           88s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-85996f8dbd   2         2         2       88s
```

Check the git source controller :

```bash
k get gitrepositories.source.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```

```log
NAME                                URL                                                            AGE     READY   STATUS
fluxv2-tutorial-deployment-source   ssh://git@github.com/mamdouni/fluxv2-tutorial-deployment.git   3m24s   True    stored artifact for revision 'main@sha1:785c439f55d544064e0d4237122b3746b0e5abc4'
```

Check the kustomization controller :

```bash
k get kustomizations.kustomize.toolkit.fluxcd.io -n fluxv2-tutorial-deployment
```

```log
NAME                                       AGE     READY   STATUS
fluxv2-tutorial-deployment-kustomization   4m16s   True    Applied revision: main@sha1:785c439f55d544064e0d4237122b3746b0e5abc4
```