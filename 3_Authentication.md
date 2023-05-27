# Authentication

For this tutorial, we will use the already created repo for the flux installtion (which is this repo).
But for our team k8s manifests, we will create a new repo to have a better control and not mix the users permissions.

First, we need to create a new git repo. For our case it will be :
- https://github.com/mamdouni/fluxv2-tutorial-deployment

Now, we need to make trust between flux and the git repo. To achieve this, we will use a deployment key.

A GitHub deploy key is an SSH key that gives read –and optionally write– access to a single repository on GitHub. It makes it easy to pull your app's code to a server automatically.
- https://dylancastillo.co/how-to-use-github-deploy-keys/

To generate the public (or the deploy) key, we will use the flux cli.
```bash
flux create secret git fluxv2-tutorial-deployment \
--url=ssh://git@github.com/mamdouni/fluxv2-tutorial-deployment.git \
--namespace=fluxv2-tutorial-deployment
```

The public key will be shown on the command output :
```log
✚ deploy key: ecdsa-sha2-nistp384 AAAAE2VjZHNhLXNoYTItbmlzdHAzODQAAAAIbmlzdHAzODQAAABhBC6fJbK5E215ZNSF0BN1Q3/vRjnEDbucAgzhSF7ls4bgSCgkQBnsWfuRCwZl/J5ESd5zskbYtFShYNYhzqq/10VybrRQmunzb6NmrlIr+6lzcIl1VRxZoigV/gSLRkY4LQ==
```

The private key will be added to the ``fluxv2-tutorial-deployment`` namespace. To show it, you can run this command :
```bash
kubectl get secrets --namespace fluxv2-tutorial-deployment fluxv2-tutorial-deployment -o yaml | yq '.data'
or
```
```bash
kubectl get secrets --namespace fluxv2-tutorial-deployment fluxv2-tutorial-deployment -o json | jq '.data'
```
or
```bash
kubectl get secrets --namespace fluxv2-tutorial-deployment fluxv2-tutorial-deployment -o yaml
```

Now you can add the public to your github repo.
- https://github.com/mamdouni/fluxv2-tutorial-deployment/settings/keys

Don't forget to check the ``Allow write access`` so this key can be used to push to this repository (Deploy keys always have pull access) .

