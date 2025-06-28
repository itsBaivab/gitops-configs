# GitOps Configurations for Goal Tracker

This repository contains Kubernetes manifests for the Goal Tracker application, managed by ArgoCD.

## Simplified Structure

```
├── dev/                      # Development environment manifests
│   ├── namespace.yaml        # Namespace definition
│   ├── frontend.yaml         # Frontend deployment and service (1 replica)
│   ├── backend.yaml          # Backend deployment and service (1 replica)
│   ├── postgres-config.yaml  # PostgreSQL ConfigMap and Secret
│   └── postgres.yaml         # PostgreSQL deployment, service, and PVC (5Gi)
├── test/                     # Test environment manifests
│   ├── namespace.yaml        # Namespace definition
│   ├── frontend.yaml         # Frontend deployment and service (2 replicas)
│   ├── backend.yaml          # Backend deployment and service (2 replicas)
│   ├── postgres-config.yaml  # PostgreSQL ConfigMap and Secret
│   └── postgres.yaml         # PostgreSQL deployment, service, and PVC (5Gi)
└── prod/                     # Production environment manifests
    ├── namespace.yaml        # Namespace definition
    ├── frontend.yaml         # Frontend deployment and service (3 replicas)
    ├── backend.yaml          # Backend deployment and service (3 replicas)
    ├── postgres-config.yaml  # PostgreSQL ConfigMap and Secret
    └── postgres.yaml         # PostgreSQL deployment, service, and PVC (20Gi)
```

## Application Components

### Frontend
- **Image**: `itsbaivab/frontend:latest`
- **Port**: 3000
- **Service**: LoadBalancer type for external access
- **Environment Variables**: 
  - `PORT=3000`
  - `BACKEND_URL=http://backend:8080`

### Backend
- **Image**: `itsbaivab/backend:latest`
- **Port**: 8080
- **Service**: ClusterIP type for internal access
- **Environment Variables**:
  - Database connection details from ConfigMap/Secret
  - `SSL=disable`
  - `PORT=8080`

### Database (PostgreSQL)
- **Image**: `postgres:15`
- **Port**: 5432
- **Storage**: 5Gi PersistentVolume
- **Credentials**: ConfigMap and Secret based

## Environment Differences

| Environment | Frontend Replicas | Backend Replicas | Resources |
|-------------|-------------------|------------------|-----------|
| Dev         | 1                 | 1                | Basic     |
| Test        | 2                 | 2                | Standard  |
| Prod        | 3                 | 3                | Enhanced  |

## ArgoCD Application Configuration

Each environment has an ArgoCD Application that:
- Tracks this repository
- Monitors the respective environment directory (dev/, test/, or prod/)
- Automatically syncs changes
- Self-heals on configuration drift

## Making Changes

1. **Update Environment Configuration**: Modify files directly in the environment folder (dev/, test/, or prod/)
2. **Image Updates**: Update image tags in the respective deployment YAML files
3. **Commit and Push**: ArgoCD will automatically detect and sync changes

## Image Update Process

To update application images:

1. Build and push new images to DockerHub:
   ```bash
   docker build -t itsbaivab/frontend:v1.1.0 ./frontend
   docker build -t itsbaivab/backend:v1.1.0 ./backend
   docker push itsbaivab/frontend:v1.1.0
   docker push itsbaivab/backend:v1.1.0
   ```

2. Update the image tags in the respective environment deployment files:
   ```bash
   # For dev environment
   sed -i 's/itsbaivab/frontend:latest/itsbaivab/frontend:v1.1.0/g' dev/frontend.yaml
   sed -i 's/itsbaivab/backend:latest/itsbaivab/backend:v1.1.0/g' dev/backend.yaml
   ```

3. Commit and push changes

## Testing Configurations

Test configurations locally using kubectl:

```bash
# Test dev configuration
kubectl apply --dry-run=client -f dev/

# Test specific file
kubectl apply --dry-run=client -f prod/frontend.yaml
```

## Troubleshooting

### Common Issues

1. **Image Pull Errors**: Ensure images exist in DockerHub
2. **Resource Limits**: Check if cluster has sufficient resources
3. **Config Errors**: Validate YAML syntax and Kubernetes API versions

### Debugging Commands

```bash
# Check ArgoCD application status
kubectl get applications -n argocd

# View application details
kubectl describe application goal-tracker-dev -n argocd

# Check pod status
kubectl get pods -n goal-tracker

# View pod logs
kubectl logs -f deployment/frontend -n goal-tracker
```

## Security Considerations

- PostgreSQL credentials are stored in Kubernetes Secrets
- Sensitive data should be managed through external secret management in production
- Network policies should be implemented for pod-to-pod communication
- Consider using service mesh for advanced traffic management

## Monitoring and Observability

Future enhancements can include:
- Prometheus metrics collection
- Grafana dashboards
- Distributed tracing with Jaeger
- Log aggregation with ELK stack
