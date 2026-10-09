# Continuous Integration and Continuous Delivery

CI/CD for Kubernetes cna be broken down into a few different components:

1. Building container images
2. Pushing container images to a registry
3. Updating Kubernetes manifests with the new image tags
4. Applying the updated manifests to the cluster(s)

CI should also still do things like validating linting, run tests, etc... but those are not Kubernetes specific so have been excluded here for brevity.

## Reference CI Implementation

A working GitHub action can be found in `./github/workflows/image-ci.yml`

**Note:** This workflow also updates the container image tags in `12-deploying-to-multiple-environments/kluctl` with a hacky/brittle string replacement task. A more robust approach would be to use https://fluxcd.io/flux/guides/image-update/ or https://docs.renovatebot.com/modules/manager/regex/ to detect new image tags and update the corresponding manifests.

## Keeping Cluster State Updated

There are a variety of method for keeping the deployed state Kubernetes manifests up to date with those in Git.

1. Manually run `kubectl apply` (or similar): This works, but inevitably someone will either forget to run the command, or do so targeting the wrong cluster.

2. Automatically run `kubectl apply` (or similar) from an automated pipeline: This is better than manual apply as it removes the possibility for human error, but (A) requires storing cluster credentials in the CI system and (B) doesn't detect/reconcile drift if someone manually makes a change to the cluster state

3. Automatically sync the state from version control ("GitOps"): By continuously pulling the latest configurations and applying them the cluster security boundary remains strongest and drift is detected/eliminated.

### GitOps Implementation:

A full GitOps example (using our configurations from `12-deploying-to-multiple-environments`) with kluctl is implemented in `./kluctl-gitops`.

For local testing, create two kind clusters and use separate Kubernetes contexts:

```bash
kind create cluster --name staging
kind create cluster --name production
```

The `deploy-staging-cluster` task uses `kind-staging`, and
`deploy-production-cluster` uses `kind-production`. Both are local simulations
of their respective environments; they do not use the remote staging or GKE
production clusters.

```bash
cd 14-cicd/kluctl-gitops
task deploy-staging-cluster
task deploy-production-cluster
```

The tasks install the Kluctl GitOps components locally and use the staging or production application target respectively. Restore the original remote contexts in `.kluctl.yaml` before using these tasks against real clusters.

The other two popular GitOps tools are:

- ArgoCD (https://argo-cd.readthedocs.io/en/stable)
- FluxCD (https://fluxcd.io/)
