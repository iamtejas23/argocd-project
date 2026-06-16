# ArgoCD Learning Project

A hands-on GitOps demo using ArgoCD to continuously sync a Kubernetes e-commerce app.

## How It Works (GitOps Flow)

```
GitHub Repo (this repo)
        │
        │  ArgoCD watches `manifests/` on every push
        ▼
  ArgoCD (in-cluster)
        │
        │  applies / syncs
        ▼
Kubernetes cluster  →  ecommerce namespace
```

1. You push a change (e.g. bump replicas or image tag) to `manifests/`
2. ArgoCD detects the drift between Git and the cluster
3. ArgoCD automatically syncs the cluster back to match Git

---

## Repository Structure

```
argocd-project/
├── manifests/              # ← ArgoCD watches this directory
│   ├── namespace.yaml      # ecommerce namespace
│   ├── deployment.yaml     # Deployment (2 replicas)
│   └── service.yaml        # LoadBalancer Service on port 80
│
└── argocd/
    └── application.yaml    # ArgoCD Application CRD (apply once)
```

---

## Setup Steps

### 1. Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Access the ArgoCD UI

```bash
# Port-forward the ArgoCD server
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get the initial admin password
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

Open `https://localhost:8080` — login with `admin` and the password above.

### 3. Register This Repo with ArgoCD

```bash
kubectl apply -f argocd/application.yaml
```

ArgoCD will immediately start syncing `manifests/` to your cluster.

---

## Learning Exercises

### Exercise 1 — Scale replicas
Edit `manifests/deployment.yaml`, change `replicas: 2` to `replicas: 4`, commit and push.
Watch ArgoCD auto-sync the cluster.

### Exercise 2 — Image update
Change the image tag in `manifests/deployment.yaml` (e.g. `iamtejas23/e-kart:v2`).
ArgoCD will roll out the new version.

### Exercise 3 — Self-heal
Manually delete the deployment in the cluster:
```bash
kubectl delete deployment ecommerce -n ecommerce
```
ArgoCD will recreate it within seconds because `selfHeal: true`.

### Exercise 4 — Disable auto-sync
Remove the `automated` block from `argocd/application.yaml` and re-apply.
Now changes only sync when you click **Sync** in the UI or run:
```bash
argocd app sync ecommerce-app
```

---

## Useful Commands

```bash
# List all ArgoCD apps
argocd app list

# Check app status
argocd app get ecommerce-app

# Manual sync
argocd app sync ecommerce-app

# Watch live resources
kubectl get all -n ecommerce
```

---

## Workflow GIF

![ArgoCD Dashboard](https://i.imgur.com/xWOfBWb.gif)
