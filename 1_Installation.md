# Installation

## Token generation

You can check this video for the full installation :
- https://app.pluralsight.com/course-player?clipId=5e85279b-951e-4000-9844-aa7aedd8ce01

First of all, you need to generate a token with admin permission :
- https://github.com/settings/tokens

By the way, this token will create a read only deploy key. check the deploy keys from the repo settings.
You can check the pub/private keys content using this command (after the set up of course) :
```bash
kubectl -n flux-system get secret flux-system -o json | jq '.data | map_values(@base64d)'
```

Export it into an environment variable :

```bash
export GITHUB_TOKEN=your-token
```

## Set Up

And run this command :
```bash
flux bootstrap github \
--repository fluxv2-tutorial \
--owner mamdouni \
--personal true \
--components-extra=image-reflector-controller,image-automation-controller
```

Check the logs after installation :
```log
► confirming components are healthy
✔ helm-controller: deployment ready
✔ image-automation-controller: deployment ready
✔ image-reflector-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
✗ bootstrap failed with 1 health check failure(s)
```

Check the repo and you'll find two commits from flux with 3 files :
- flux-system/gotk-components.yaml  : which contains all the flux components
- flux-system/gotk-sync.yaml        : which binds flux to its github repo 
- flux-system/kustomization.yaml    : which runs the two above files

Check now your k8s environment :
```bash
k get all --namespace flux-system
```

```log
NAME                                               READY   STATUS    RESTARTS   AGE
pod/helm-controller-65fc454c55-xw76n               1/1     Running   0          19m
pod/image-automation-controller-6499959f59-d45dw   1/1     Running   0          19m
pod/image-reflector-controller-8446dc766b-cbx56    1/1     Running   0          19m
pod/kustomize-controller-6f8f78f589-8x9fk          1/1     Running   0          19m
pod/notification-controller-6cc8544455-8jvl7       1/1     Running   0          19m
pod/source-controller-c54dc854f-pb6r2              1/1     Running   0          19m

NAME                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/notification-controller   ClusterIP   10.109.234.202   <none>        80/TCP    19m
service/source-controller         ClusterIP   10.111.162.2     <none>        80/TCP    19m
service/webhook-receiver          ClusterIP   10.98.228.67     <none>        80/TCP    19m

NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/helm-controller               1/1     1            1           19m
deployment.apps/image-automation-controller   1/1     1            1           19m
deployment.apps/image-reflector-controller    1/1     1            1           19m
deployment.apps/kustomize-controller          1/1     1            1           19m
deployment.apps/notification-controller       1/1     1            1           19m
deployment.apps/source-controller             1/1     1            1           19m

NAME                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/helm-controller-65fc454c55               1         1         1       19m
replicaset.apps/image-automation-controller-6499959f59   1         1         1       19m
replicaset.apps/image-reflector-controller-8446dc766b    1         1         1       19m
replicaset.apps/kustomize-controller-6f8f78f589          1         1         1       19m
replicaset.apps/notification-controller-6cc8544455       1         1         1       19m
replicaset.apps/source-controller-c54dc854f              1         1         1       19m
```

Check the source-controller or the kustomization-controller and you'll find that the reconciliation will happen each 10 minutes (10m0s). You can change of course this parameter.

To force the flux reconciliation, you can use this command :
```bash
flux reconcile source git flux-system && flux reconcile kustomization flux-system
```

## Debug

To check erros, you can check the log of the source or the kustomization pods or you can use this command :
```bash
flux logs --level=error
```

```bash
k get pods --all-namespaces
```

```bash
 k logs source-controller-68c754f876-xgkdh -n flux-system
```

```bash
k logs kustomize-controller-787d5575fc-f25nw -n flux-system
```

You can also check the devoxx demo here :
- https://github.com/kalioz/fluxv2-demo/blob/main/docs/00.requirements.md
