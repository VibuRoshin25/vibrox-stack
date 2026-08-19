# Deployment Guide

This guide covers deploying the Vibrox Stack in different environments, from local development to production.

## 🎯 Deployment Options

| Environment           | Method         | Use Case                    |
| --------------------- | -------------- | --------------------------- |
| **Local Development** | Docker Compose | Development and testing     |
| **Staging**           | Kubernetes     | Pre-production testing      |
| **Production**        | Kubernetes     | Live production environment |

## 🐳 Docker Compose Deployment

### Docker Compose Prerequisites

- Docker and Docker Compose installed
- Git with submodules

### Quick Start

```bash
# Clone the repository
git clone --recurse-submodules https://github.com/VibuRoshin25/vibrox-stack.git
cd vibrox-stack

# Start all services
docker-compose up --build

# Start in detached mode
docker-compose up -d --build
```

### Service Endpoints

| Service         | Port | Description            |
| --------------- | ---- | ---------------------- |
| **vibrox-core** | 8080 | User management API    |
| **vibrox-auth** | 8000 | Authentication service |
| **vibrox-echo** | 9000 | Logging service        |
| **PostgreSQL**  | 5432 | Database               |

### Environment Configuration

The Docker Compose setup uses the following environment variables:

```yaml
# Database Configuration
POSTGRES_USER: postgres
POSTGRES_PASSWORD: server
POSTGRES_DB: postgres

# Service Configuration
DB_HOST: db
DB_USER: postgres
DB_PASSWORD: server
DB_NAME: postgres
AUTH_HOST: auth:8000
LOGGER_HOST: logger:9000
```

### Volume Management

```bash
# View volumes
docker volume ls

# Clean up volumes
docker-compose down -v
docker volume prune

# Backup database
docker-compose exec db pg_dump -U postgres postgres > backup.sql
```

## ☸️ Kubernetes Deployment

### Kubernetes Prerequisites

- Kubernetes cluster (local or cloud)
- kubectl configured
- Helm (optional, for advanced deployments)

### Cluster Setup

#### Local Development (Minikube)

```bash
# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start cluster
minikube start

# Enable ingress
minikube addons enable ingress
```

#### Cloud Deployment

- **AWS EKS**: Use eksctl or AWS Console
- **Google GKE**: Use gcloud CLI or Console
- **Azure AKS**: Use Azure CLI or Portal

### Deploy to Kubernetes

```bash
# Apply all manifests
kubectl apply -f manifests/

# Check deployment status
kubectl get pods
kubectl get services
kubectl get deployments

# View logs
kubectl logs -f deployment/vibrox-core
kubectl logs -f deployment/vibrox-auth
kubectl logs -f deployment/vibrox-echo
```

### Service Configuration

#### Database (PostgreSQL)

```bash
# Deploy database
kubectl apply -f manifests/db-deployment.yml
kubectl apply -f manifests/db-service.yml

# Create persistent volume
kubectl apply -f volume-manifests/
```

#### Application Services

```bash
# Deploy core services
kubectl apply -f manifests/app-deployment.yml
kubectl apply -f manifests/app-service.yml

# Deploy authentication
kubectl apply -f manifests/auth-deployment.yml
kubectl apply -f manifests/auth-service.yml

# Deploy logging
kubectl apply -f manifests/log-deployment.yml
kubectl apply -f manifests/log-service.yml
```

### Environment Variables

Create ConfigMaps and Secrets for environment configuration:

```bash
# Create ConfigMap
kubectl create configmap vibrox-config \
  --from-literal=DB_HOST=postgres-service \
  --from-literal=DB_NAME=postgres \
  --from-literal=AUTH_HOST=auth-service:8000 \
  --from-literal=LOGGER_HOST=log-service:9000

# Create Secret
kubectl create secret generic vibrox-secrets \
  --from-literal=DB_PASSWORD=your-secure-password \
  --from-literal=JWT_SECRET=your-jwt-secret
```

### Ingress Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vibrox-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: vibrox.local
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 8080
          - path: /auth
            pathType: Prefix
            backend:
              service:
                name: auth-service
                port:
                  number: 8000
```

## 🔧 Production Considerations

### Security

1. **Secrets Management**

   ```bash
   # Use Kubernetes secrets or external secret managers
   kubectl create secret generic db-credentials \
     --from-literal=username=postgres \
     --from-literal=password=secure-password
   ```

2. **Network Policies**

   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: vibrox-network-policy
   spec:
     podSelector: {}
     policyTypes:
       - Ingress
       - Egress
     ingress:
       - from:
           - namespaceSelector: {}
         ports:
           - protocol: TCP
             port: 8080
   ```

3. **RBAC Configuration**

   ```bash
   # Create service accounts
   kubectl create serviceaccount vibrox-core
   kubectl create serviceaccount vibrox-auth
   kubectl create serviceaccount vibrox-echo
   ```

### Monitoring and Logging

1. **Prometheus Integration**

   ```yaml
   # Add Prometheus annotations to deployments
   annotations:
     prometheus.io/scrape: "true"
     prometheus.io/port: "8080"
     prometheus.io/path: "/metrics"
   ```

2. **Log Aggregation**

   ```bash
   # Configure log forwarding
   kubectl apply -f manifests/logging/
   ```

### Scaling

```bash
# Horizontal Pod Autoscaling
kubectl autoscale deployment vibrox-core --cpu-percent=70 --min=2 --max=10
kubectl autoscale deployment vibrox-auth --cpu-percent=70 --min=2 --max=10
kubectl autoscale deployment vibrox-echo --cpu-percent=70 --min=2 --max=10
```

## 🔄 Deployment Workflow

### CI/CD Pipeline

```yaml
# Example GitHub Actions workflow
name: Deploy to Kubernetes
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build and push images
        run: |
          docker build -t vibrox-core ./vibrox-core
          docker build -t vibrox-auth ./vibrox-auth
          docker build -t vibrox-echo ./vibrox-echo
      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f manifests/
```

### Blue-Green Deployment

```bash
# Deploy new version
kubectl apply -f manifests/v2/

# Switch traffic
kubectl patch service app-service -p '{"spec":{"selector":{"version":"v2"}}}'

# Rollback if needed
kubectl patch service app-service -p '{"spec":{"selector":{"version":"v1"}}}'
```

## 🔍 Troubleshooting

### Common Issues

1. **Service Discovery**

   ```bash
   # Check DNS resolution
   kubectl run test --image=busybox --rm -it --restart=Never -- nslookup auth-service
   ```

2. **Database Connectivity**

   ```bash
   # Test database connection
   kubectl run test --image=postgres --rm -it --restart=Never -- psql -h postgres-service -U postgres
   ```

3. **Resource Issues**

   ```bash
   # Check resource usage
   kubectl top pods
   kubectl describe pod <pod-name>
   ```

### Debugging Commands

```bash
# Get pod details
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name> -f

# Execute commands in pods
kubectl exec -it <pod-name> -- /bin/bash

# Port forward for local access
kubectl port-forward service/app-service 8080:8080
```

## 📊 Health Checks

### Liveness and Readiness Probes

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Monitoring Endpoints

- `/health` - Service health check
- `/ready` - Readiness check
- `/metrics` - Prometheus metrics

---

_For detailed service-specific deployment instructions, see the individual service documentation._
