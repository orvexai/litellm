# Tekton build

Builds the `runtime` image from the root `Dockerfile` on every push to the fork and pushes it to `ghcr.io/orvexai/litellm:<first 12 chars of the commit sha>`.

Pushes to `main` also deploy that image with `helm upgrade --install` using the `helm/litellm-helm` chart. The release is called `litellm` in the `litellm` namespace and keeps its existing values through `--reuse-values`. The upgrade is `--atomic`, so a failed rollout rolls back. Other branches only build.

The Dockerfile relies on BuildKit features, so the build step runs rootless BuildKit instead of Kaniko or Buildah.

Prerequisites on the cluster are Tekton Pipelines and Tekton Triggers, with the `github` and `cel` interceptors installed.

```bash
kubectl create secret generic github-webhook-secret --from-literal=secret="$(openssl rand -hex 20)"
kubectl create secret generic registry-credentials \
  --from-file=config.json=$HOME/.docker/config.json
kubectl apply -f tekton/pipeline.yaml -f tekton/triggers.yaml -f tekton/deploy-rbac.yaml
```

The manifests assume the Tekton resources live in the `default` namespace and the app in the `litellm` namespace. If yours differ, change the subject namespace in `deploy-rbac.yaml` and the `release` and `namespace` params in `pipeline.yaml`. The cluster also needs an image pull secret for ghcr.io if the package is private.

Expose the `el-litellm-github` service through an Ingress, then add a GitHub webhook pointing at it. Use the content type `application/json`, only the push event, and the same secret as `github-webhook-secret`.

To change the registry, edit the `image` param in the `litellm-build` TriggerTemplate.
