# Tekton build and deploy

A push to `main` builds the root `Dockerfile` (`runtime` stage) with the shared `buildah-with-args` Task and pushes it to `repos.eu-central-1.myidp.cloud/litellm/litellm:orvex-<first 12 of the commit sha>`. The `litellm` Harbor project is public so the chart needs no pull secret.

After the build, the `pin-gitops` task sets `image.repository` and `image.tag` in `orvexai/my-idp` `platform/apps/litellm/values.yaml`. ArgoCD then rolls the `litellm` app out.

`orvexai/my-idp-apps` `applications/eu-central-1/litellm-ci.yaml` syncs this directory into `tekton-pipelines`. The Trigger is picked up by the platform `github-webhook-listener` through the `ci.my-idp/discover` label, so no webhook or EventListener lives here.

To build without deploying, run the Pipeline by hand with `deploy=false`.
