# EdgeShop GitOps Configuration

This repository contains ArgoCD Application definitions for managing deployments across the EdgeShop platform.

## Repository Structure

```
.
├── argocd/
│   ├── app-of-apps.yaml       # Main application that manages all apps
│   └── apps/                  # Directory containing individual app definitions
│       ├── mobile-task-dev.yaml
│       └── [other-apps].yaml
└── README.md
```

## How It Works

### App of Apps Pattern

The `app-of-apps.yaml` is a special ArgoCD Application that manages all other applications. It:
- Points to the `argocd/apps/` directory in this repo
- Automatically discovers and manages all Application definitions in that directory
- Enables centralized management of all deployments

### Adding New Applications

1. Create a new YAML file in `argocd/apps/` directory
2. Follow the pattern in `mobile-task-dev.yaml`
3. Commit and push to this repository
4. ArgoCD will automatically detect and deploy the new application

## Bootstrap Instructions

### Initial Setup

1. **Clone this repository** (or create it if it doesn't exist):
   ```bash
   git clone https://github.com/kirtquist/edgeshop-gitops.git
   cd edgeshop-gitops
   ```

2. **Apply the App of Apps to ArgoCD**:
   ```bash
   kubectl apply -f argocd/app-of-apps.yaml -n argocd
   ```

3. **Verify in ArgoCD UI**:
   - Open ArgoCD UI
   - You should see the `app-of-apps` application
   - It will automatically sync and create all applications defined in `argocd/apps/`

### After Infrastructure Recreation

When you run `pulumi destroy` and then `pulumi up`:

1. Infrastructure is recreated
2. ArgoCD is redeployed
3. Apply the App of Apps again:
   ```bash
   kubectl apply -f argocd/app-of-apps.yaml -n argocd
   ```
4. All applications will automatically redeploy from their source repositories

## Application Definition Template

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <app-name>-<environment>
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
  labels:
    app.kubernetes.io/name: <app-name>
    environment: <environment>
spec:
  project: default
  source:
    repoURL: https://github.com/<org>/<repo>.git
    targetRevision: main
    path: <path-to-manifests>
  destination:
    server: https://kubernetes.default.svc
    namespace: <target-namespace>
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## Current Applications

- **mobile-task-dev**: Mobile task application (dev environment)
  - Source: `https://github.com/kirtquist/mobile-task-application.git`
  - Path: `k8s/overlays/dev`
  - Namespace: `dev`

## Best Practices

1. **One Application per File**: Each YAML file should define one ArgoCD Application
2. **Naming Convention**: Use `<app-name>-<environment>` format (e.g., `mobile-task-dev`)
3. **Labels**: Always include labels for filtering and organization
4. **Sync Policy**: Use automated sync with prune and selfHeal for dev environments
5. **Namespace Management**: Use `CreateNamespace=true` to auto-create namespaces

## Troubleshooting

### Applications Not Appearing

1. Check if App of Apps is synced:
   ```bash
   kubectl get application app-of-apps -n argocd
   ```

2. Check ArgoCD logs:
   ```bash
   kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server
   ```

### Applications Not Syncing

1. Verify repository access in ArgoCD
2. Check if the path exists in the source repository
3. Verify target namespace exists or can be created

## Repository Management

- **Main Branch**: Contains production-ready configurations
- **Feature Branches**: Use for testing new application definitions
- **Pull Requests**: Review application definitions before merging
