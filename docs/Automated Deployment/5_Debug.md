
## Debug

To force the flux reconciliation, you can use this command :
```bash
flux reconcile source git flux-system && flux reconcile kustomization flux-system
```

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