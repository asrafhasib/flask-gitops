# flask-gitops

This repository is the GitOps source of truth for deploying `flask-cicd-app` to Kubernetes with Argo CD.

## Repository setup

1. Create a GitHub repository named `flask-gitops`.
2. Upload/push all files from this folder to the repository root.
3. Replace `YOUR_GITHUB_USERNAME` in:
   - `apps/flask-app/deployment.yaml`
   - `argocd/application.yaml`
4. In the companion `flask-cicd-app` repository, replace `YOUR_GITHUB_USERNAME/flask-gitops` in `.github/workflows/ci-cd.yaml`.
5. Push this repository to the `main` branch.
6. Push the application repository to `main`. Its GitHub Actions workflow will build the first image and replace the initial image reference in this repository.
7. After the first CI/CD workflow succeeds, apply the Argo CD Application manifest:

```bash
kubectl apply -f argocd/application.yaml
```

Run that command with the intended Kind cluster selected and Argo CD already installed.

## Verify

```bash
kubectl get applications -n argocd
kubectl get all -n flask-app
kubectl rollout status deployment/flask-app -n flask-app
kubectl port-forward -n flask-app svc/flask-app-service 5000:80
```

Then visit `http://localhost:5000/`.

## GitOps behavior

Argo CD watches `apps/flask-app` on the `main` branch. Automated sync, pruning, and self-healing are enabled. Do not make routine application changes directly with `kubectl`; commit desired-state changes to Git instead.

## Registry note

The starter assumes the GHCR image is public. For a private package, configure an image pull secret in the target namespace and reference it from the Deployment's `imagePullSecrets`.
