# GitOps with Argo CD and Minikube

This repository demonstrates a GitOps workflow using Argo CD and Minikube. It implements a simple nginx application deployment with multiple environments managed through Kustomize.

## Repository Structure

```
.
├── applications/                 # Argo CD Application definitions
│   └── application-set.yaml     # ApplicationSet for automated app creation
├── kustomize/                   # Kustomize configurations
│   ├── base/                    # Base configuration
│   │   ├── deployment.yaml      # Base nginx deployment
│   │   ├── service.yaml         # Base nginx service
│   │   ├── configmap.yaml       # Base configuration
│   │   └── kustomization.yaml   # Base kustomization file
│   └── overlays/                # Environment-specific overlays
└── README.md                    # This file
```

## Prerequisites

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Argo CD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)

## Deployment Steps

1. Start Minikube:
   ```bash
   minikube start
   ```

2. Install Argo CD:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. Wait for Argo CD to be ready:
   ```bash
   kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd
   ```

4. Get Argo CD admin password:
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```

5. Port-forward Argo CD server:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

6. Login to Argo CD (use the password from step 4):
   ```bash
   argocd login localhost:8080 --username admin --insecure
   ```

7. Apply the ApplicationSet:
   ```bash
   kubectl apply -f applications/application-set.yaml
   ```

## Accessing the Application

- Argo CD UI: https://localhost:8080 (use admin username and password from step 4)
- The deployed nginx applications will be available in their respective namespaces

### Accessing Services on Mac

When running on macOS, the Minikube IP might not be directly reachable. To access the services, use:

```bash
# Get the Argo CD server URL
minikube service argocd-server --url -n argocd
# This will output something like: https://127.0.0.1:XXXXX
# Use this URL to access the Argo CD UI in your browser

# Get the nginx service URL
minikube service nginx --url
# This will output something like: http://127.0.0.1:XXXXX
# Use this URL to access the nginx service in your browser
```

These commands will provide you with local URLs that you can use to access the services through port forwarding.

## How It Works

1. The `ApplicationSet` in `applications/application-set.yaml` automatically creates Argo CD Applications for each directory in `kustomize/overlays/`.
2. Each application is configured to:
   - Use the corresponding overlay from the `kustomize/overlays/` directory
   - Deploy to its own namespace (automatically created)
   - Enable automated sync with pruning and self-healing
   - Use the main branch as the target revision

## GitOps Workflow

1. Make changes to the base configuration or overlays in the `kustomize/` directory
2. Commit and push changes to the main branch
3. Argo CD automatically detects changes and syncs the applications
4. Monitor the sync status in the Argo CD UI

## Troubleshooting

- Check Argo CD pod status:
  ```bash
  kubectl get pods -n argocd
  ```
- View application sync status:
  ```bash
  argocd app list
  ```
- Check application logs:
  ```bash
  argocd app logs <application-name>
  ```

## Cleanup

To remove all resources:
```bash
kubectl delete -f applications/application-set.yaml
kubectl delete namespace argocd
minikube stop
```