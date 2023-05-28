# Authentication

For this tutorial, we will use the already created repo for the flux installation (which is this repo).
But for our team k8s manifests, we will create a new repo to have a better control and not mix the users permissions.

## The team deployment repository

First, we need to create a new git repo. For our case it will be :
- https://github.com/mamdouni/fluxv2-tutorial-deployment

create a new namespace for this environment :

```bash
k create ns fluxv2-tutorial-deployment
```
## Generate Public/Private Keys

Now, we need to make trust between flux and the git repo. To achieve this, we will use a deployment key.

A GitHub deploy key is an SSH key that gives read –and optionally write– access to a single repository on GitHub. It makes it easy to pull your app's code to a server automatically.
- https://dylancastillo.co/how-to-use-github-deploy-keys/

To generate the public (or the deploy) key, we will use the flux cli.
```bash
flux create secret git fluxv2-tutorial-deployment-secret \
--url=ssh://git@github.com/mamdouni/fluxv2-tutorial-deployment.git \
--namespace=fluxv2-tutorial-deployment
```

The public key will be shown on the command output :
```log
✚ deploy key: ecdsa-sha2-nistp384 AAAAE2VjZHNhLXNoYTItbmlzdHAzODQAAAAIbmlzdHAzODQAAABhBDL3ALBlv0ZxIf8X5zPHmXdELTNsHEb4hMGFkXmAsMjnk+SFcovsA+efc7I9TI+jUC98vKaEHKbLx5em8n2IszzLSJyw0EsTAhlyyPWT5PAICM2w2EVnv/VPYILuWAtrUw==
```

The private key will be added to the ``fluxv2-tutorial-deployment`` namespace. To show it, you can run this command :
```bash
kubectl get secrets --namespace fluxv2-tutorial-deployment fluxv2-tutorial-deployment-secret -o yaml | yq '.data'
or
```
```bash
kubectl get secrets --namespace fluxv2-tutorial-deployment fluxv2-tutorial-deployment-secret -o json | jq '.data'
```
or
```bash
kubectl get secrets --namespace fluxv2-tutorial-deployment fluxv2-tutorial-deployment-secret -o yaml
```

Now you can add the public to your github repo.
- https://github.com/mamdouni/fluxv2-tutorial-deployment/settings/keys

Don't forget to check the ``Allow write access`` so this key can be used to push to this repository (Deploy keys always have pull access) .

