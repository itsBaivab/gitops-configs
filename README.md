# GitOps Configurations for Goal Tracker

This repository contains Kubernetes manifests and Kustomize configurations for the Goal Tracker application, managed by ArgoCD.

## Structure

```
├── apps/
│   └── goal-tracker/
│       ├── base/                 # Base Kubernetes manifests
│       │   ├── backend.yaml      # Backend deployment and service
│       │   ├── frontend.yaml     # Frontend deployment and service
│       │   ├── postgres.yaml     # PostgreSQL deployment, service, and PVC
│       │   ├── postgres-config.yaml # ConfigMap and Secret for PostgreSQL
│       │   ├── namespace.yaml    # Namespace definition
│       │   └── kustomization.yaml # Base kustomization
│       └── overlays/            # Environment-specific configurations
│           ├── dev/             # Development environment
│           ├── test/            # Test environment
│           └── prod/            # Production environment
└── environments/               # Environment entry points
    ├── dev/
    ├── test/
    └── prod/
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
- Monitors the respective environment path
- Automatically syncs changes
- Self-heals on configuration drift

## Making Changes

1. **Update Base Configuration**: Modify files in `apps/goal-tracker/base/`
2. **Environment-Specific Changes**: Modify overlay files in `apps/goal-tracker/overlays/<env>/`
3. **Image Updates**: Update image tags in the respective kustomization.yaml files
4. **Commit and Push**: ArgoCD will automatically detect and sync changes

## Image Update Process

To update application images:

1. Build and push new images to DockerHub:
   ```bash
   docker build -t itsbaivab/frontend:v1.1.0 ./frontend
   docker build -t itsbaivab/backend:v1.1.0 ./backend
   docker push itsbaivab/frontend:v1.1.0
   docker push itsbaivab/backend:v1.1.0
   ```

2. Update the kustomization.yaml in the respective environment:
   ```yaml
   images:
   - name: itsbaivab/backend
     newTag: v1.1.0
   - name: itsbaivab/frontend
     newTag: v1.1.0
   ```

3. Commit and push changes

## Testing Configurations

Use `kustomize` to test configurations locally:

```bash
# Test dev configuration
kustomize build environments/dev

# Test specific overlay
kustomize build apps/goal-tracker/overlays/prod
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
