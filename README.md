# Goal Tracker Application - ArgoCD Deployment

This directory contains Kubernetes manifests for deploying the Goal Tracker application using ArgoCD.

## Architecture

The application consists of three main components:

1. **Frontend** - React application (itsbaivab/frontend:latest)
2. **Backend** - API server (itsbaivab/backend:latest)
3. **PostgreSQL** - Database (postgres:15)

## Files Overview

- `namespace.yaml` - Creates the goal-tracker namespace
- `postgres-config.yaml` - ConfigMap and Secret for PostgreSQL
- `postgres-pvc.yaml` - Persistent Volume Claim for PostgreSQL data
- `postgres.yaml` - PostgreSQL deployment and service
- `backend-config.yaml` - ConfigMap and Secret for backend service
- `backend.yaml` - Backend deployment and service
- `frontend-config.yaml` - ConfigMap for frontend service
- `frontend.yaml` - Frontend deployment, service, and ingress
- `kustomization.yaml` - Kustomize configuration
- `argocd-application.yaml` - ArgoCD Application manifest

## Deployment Steps

### 1. Update ArgoCD Application Manifest

Edit `argocd-application.yaml` and replace the repository URL with your actual Git repository:

```yaml
source:
  repoURL: https://github.com/your-username/your-repo-name.git
```

### 2. Apply the ArgoCD Application

```bash
kubectl apply -f argocd-application.yaml
```

### 3. Monitor the Deployment

```bash
# Check ArgoCD application status
kubectl get applications -n argocd

# Check pods in the goal-tracker namespace
kubectl get pods -n goal-tracker

# Check services
kubectl get svc -n goal-tracker
```

## Accessing the Application

### Via Port Forward (for testing)

```bash
# Frontend
kubectl port-forward -n goal-tracker svc/frontend 3000:3000

# Backend API
kubectl port-forward -n goal-tracker svc/backend 8080:8080

# PostgreSQL (for debugging)
kubectl port-forward -n goal-tracker svc/postgres 5432:5432
```

### Via Ingress

The application includes an Ingress configuration for the frontend. Make sure you have an Ingress controller (like NGINX) installed in your cluster.

Add the following to your `/etc/hosts` file (or equivalent):
```
<your-cluster-ip> goal-tracker.local
```

Then access the application at: http://goal-tracker.local

## Configuration

### Environment Variables

**Backend:**
- `DB_HOST`: postgres
- `DB_PORT`: 5432
- `DB_NAME`: goalsdb
- `DB_USERNAME`: postgres
- `DB_PASSWORD`: postgres (stored in Secret)
- `SSL`: disable
- `PORT`: 8080

**Frontend:**
- `PORT`: 3000
- `BACKEND_URL`: http://backend:8080

### Resource Limits

Each service has resource requests and limits configured:
- Requests: 256Mi memory, 250m CPU
- Limits: 512Mi memory, 500m CPU

### Health Checks

Both frontend and backend services include liveness and readiness probes for better reliability.

## Troubleshooting

### Check Application Status in ArgoCD

```bash
kubectl describe application goal-tracker-app -n argocd
```

### Check Pod Logs

```bash
kubectl logs -n goal-tracker -l app=frontend
kubectl logs -n goal-tracker -l app=backend
kubectl logs -n goal-tracker -l app=postgres
```

### Restart Deployments

```bash
kubectl rollout restart deployment/frontend -n goal-tracker
kubectl rollout restart deployment/backend -n goal-tracker
kubectl rollout restart deployment/postgres -n goal-tracker
```

## Updating the Application

To update the application images:

1. Push new images to Docker Hub with appropriate tags
2. Update the image tags in `kustomization.yaml`
3. Commit and push changes to your Git repository
4. ArgoCD will automatically sync the changes (if auto-sync is enabled)

Alternatively, you can manually sync from the ArgoCD UI or CLI:

```bash
argocd app sync goal-tracker-app
```
