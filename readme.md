# ArgoCD Tool Exploration for DevOps IA

## Prerequisites

- Minikube installed
- kubectl CLI configured
- GitHub repository for manifests

***

## 1. Minikube Setup

```bash
minikube start --driver=docker
# Starts a single-node Kubernetes cluster in Docker.

minikube status
# Verifies cluster components: control plane, kubelet, apiserver.

kubectl get nodes
# Lists cluster nodes to confirm readiness.

kubectl get pods -A
# Shows all namespace pods to ensure system components are running.
```

## 2. Install ArgoCD

```bash
kubectl create namespace argocd
# Creates dedicated namespace for ArgoCD.

kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
# Installs ArgoCD CRDs, deployments, StatefulSet, services.

kubectl get pods -n argocd
# Confirms ArgoCD server, repo-server, controller, etc., are Running.
```

## 3. Access ArgoCD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Forwards local port 8080 to ArgoCD server port 443.

kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
# Retrieves initial admin password.

argocd login localhost:8080 --username admin --password <password>
# Logs into ArgoCD CLI.
```

## 4. Git Repository Manifests

- **deployment.yaml**: NGINX Deployment (replicas: 1)
- **service.yaml**: ClusterIP Service exposing NGINX

## 5. Define ArgoCD Application

```yaml
# app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: aditya-demo-app
  namespace: argocd-devops-demo
spec:
  project: default
  source:
    repoURL: https://github.com/Kothari-Aditya/devops-ia-argo-cd-demo.git
    targetRevision: HEAD
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: false
    syncOptions:
    - CreateNamespace=true
```

## 6. Deploy ArgoCD Application

```bash
kubectl create namespace argocd-devops-demo
# Namespace for application.

kubectl apply -f fix-rbac.yaml
# Grants cluster-admin role to ArgoCD controller SA.

kubectl rollout restart statefulset argocd-application-controller -n argocd
# Restarts controller to apply RBAC changes.

kubectl apply -f app.yaml
# Registers Application with ArgoCD.

kubectl get applications -n argocd-devops-demo
# Confirms 'aditya-demo-app' is registered.
```

## 7. Demonstrations

### 7.1 Initial Sync \& Health

### 7.2 Auto-Sync on Commit

```bash
# Edit deployment.yaml: replicas: 5 -> 7
git add deployment.yaml && \
git commit -m "Scale replicas to 7" && \
git push
```

### 7.3 Manual Sync (Rollback)

```bash
kubectl patch application aditya-demo-app \
  -n argocd-devops-demo --type merge \
  -p '{"spec":{"syncPolicy":null}}'
# Disables auto-sync.

# Revert replicas to 2, commit & push
git add deployment.yaml && \
git commit -m "Scale replicas to 2" && \
git push
```

## Author Details

**Name**: Aditya Kothari
**Department**: Computer Engineering
**Roll No.**: 16010122329
**Batch**: C-3