# Kubernetes Deployment Guide

## Overview

This guide covers deploying the Vibrox Stack to Kubernetes for production environments. It includes cluster setup, service deployment, monitoring, and operational procedures.

## Prerequisites

- Kubernetes cluster (1.21+)
- kubectl configured
- Helm 3.0+ (optional)
- Container registry access
- Persistent storage provisioner
- Ingress controller

## Cluster Requirements

### Minimum Requirements

- **Nodes**: 3 worker nodes
- **CPU**: 4 cores per node
- **Memory**: 8GB RAM per node
- **Storage**: 100GB per node
- **Network**: 1Gbps network connectivity

### Recommended Requirements

- **Nodes**: 5+ worker nodes
- **CPU**: 8+ cores per node
- **Memory**: 16GB+ RAM per node
- **Storage**: 500GB+ SSD per node
- **Network**: 10Gbps network connectivity

## Quick Start

### 1. Prepare Kubernetes Cluster

```bash
# Verify cluster access
kubectl cluster-info

# Check node status
kubectl get nodes

# Verify storage class
kubectl get storageclass
```

### 2. Create Namespace

```bash
# Create namespace for Vibrox Stack
kubectl create namespace vibrox

# Set namespace as default
kubectl config set-context --current --namespace=vibrox
```

### 3. Deploy Services

```bash
# Apply all manifests
kubectl apply -f manifests/

# Check deployment status
kubectl get pods
kubectl get services
kubectl get deployments
```

## Service Deployment

### Database Deployment

```yaml
# manifests/db-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: vibrox
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_USER
          value: postgres
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        - name: POSTGRES_DB
          value: postgres
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
```

### Database Service

```yaml
# manifests/db-service.yml
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: vibrox
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  type: ClusterIP
```

### Persistent Volume Claim

```yaml
# volume-manifests/postgres-pvc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: vibrox
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

### Application Deployment

```yaml
# manifests/app-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vibrox-core
  namespace: vibrox
spec:
  replicas: 3
  selector:
    matchLabels:
      app: vibrox-core
  template:
    metadata:
      labels:
        app: vibrox-core
    spec:
      containers:
      - name: vibrox-core
        image: vibrox-core:latest
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          value: postgres-service
        - name: DB_USER
          value: postgres
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        - name: DB_NAME
          value: postgres
        - name: AUTH_HOST
          value: vibrox-auth-service:8000
        - name: LOGGER_HOST
          value: vibrox-echo-service:9000
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Application Service

```yaml
# manifests/app-service.yml
apiVersion: v1
kind: Service
metadata:
  name: vibrox-core-service
  namespace: vibrox
spec:
  selector:
    app: vibrox-core
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
```

## Configuration Management

### ConfigMaps

```yaml
# manifests/configmap.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: vibrox
data:
  DB_HOST: postgres-service
  DB_USER: postgres
  DB_NAME: postgres
  AUTH_HOST: vibrox-auth-service:8000
  LOGGER_HOST: vibrox-echo-service:9000
  LOG_LEVEL: INFO
```

### Secrets

```yaml
# manifests/secrets.yml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  namespace: vibrox
type: Opaque
data:
  password: c2VydmVy  # base64 encoded 'server'
---
apiVersion: v1
kind: Secret
metadata:
  name: jwt-secret
  namespace: vibrox
type: Opaque
data:
  secret: eW91ci1qd3Qtc2VjcmV0  # base64 encoded JWT secret
```

## Ingress Configuration

### Ingress Controller

```yaml
# manifests/ingress.yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vibrox-ingress
  namespace: vibrox
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - api.vibrox.com
    secretName: vibrox-tls
  rules:
  - host: api.vibrox.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vibrox-core-service
            port:
              number: 8080
```

## Monitoring and Observability

### Horizontal Pod Autoscaler

```yaml
# manifests/hpa.yml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: vibrox-core-hpa
  namespace: vibrox
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vibrox-core
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Service Monitor (Prometheus)

```yaml
# manifests/servicemonitor.yml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: vibrox-monitor
  namespace: vibrox
spec:
  selector:
    matchLabels:
      app: vibrox-core
  endpoints:
  - port: http
    path: /metrics
    interval: 30s
```

## Deployment Workflow

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build and push images
      run: |
        docker build -t vibrox-core:latest ./vibrox-core
        docker push vibrox-core:latest
    
    - name: Deploy to Kubernetes
      run: |
        kubectl apply -f manifests/
        kubectl rollout status deployment/vibrox-core -n vibrox
```

### Rolling Updates

```bash
# Update deployment
kubectl set image deployment/vibrox-core vibrox-core=vibrox-core:v2.0.0 -n vibrox

# Monitor rollout
kubectl rollout status deployment/vibrox-core -n vibrox

# Rollback if needed
kubectl rollout undo deployment/vibrox-core -n vibrox
```

## Operational Procedures

### Backup and Recovery

```bash
# Create database backup
kubectl exec -n vibrox deployment/postgres -- pg_dump -U postgres postgres > backup.sql

# Restore database
kubectl exec -i -n vibrox deployment/postgres -- psql -U postgres postgres < backup.sql

# Backup persistent volumes
kubectl get pvc -n vibrox
kubectl get pv
```

### Scaling Operations

```bash
# Scale application
kubectl scale deployment vibrox-core --replicas=5 -n vibrox

# Check scaling status
kubectl get hpa -n vibrox
kubectl describe hpa vibrox-core-hpa -n vibrox
```

### Troubleshooting

```bash
# Check pod status
kubectl get pods -n vibrox

# View pod logs
kubectl logs -f deployment/vibrox-core -n vibrox

# Describe pod for details
kubectl describe pod <pod-name> -n vibrox

# Execute into pod
kubectl exec -it <pod-name> -n vibrox -- /bin/bash
```

## Security Configuration

### Network Policies

```yaml
# manifests/network-policy.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: vibrox-network-policy
  namespace: vibrox
spec:
  podSelector:
    matchLabels:
      app: vibrox-core
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
```

### RBAC Configuration

```yaml
# manifests/rbac.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: vibrox-sa
  namespace: vibrox
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: vibrox-role
  namespace: vibrox
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: vibrox-role-binding
  namespace: vibrox
subjects:
- kind: ServiceAccount
  name: vibrox-sa
  namespace: vibrox
roleRef:
  kind: Role
  name: vibrox-role
  apiGroup: rbac.authorization.k8s.io
```

## Performance Optimization

### Resource Management

```yaml
# Resource requests and limits
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"
```

### Pod Disruption Budget

```yaml
# manifests/pdb.yml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: vibrox-pdb
  namespace: vibrox
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: vibrox-core
```

## Monitoring Setup

### Prometheus Configuration

```yaml
# prometheus-config.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'vibrox-services'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_label_app]
        regex: vibrox-.*
        action: keep
```

### Grafana Dashboards

```yaml
# grafana-dashboard.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboard
  namespace: monitoring
data:
  vibrox-dashboard.json: |
    {
      "dashboard": {
        "title": "Vibrox Stack Dashboard",
        "panels": [
          {
            "title": "Request Rate",
            "type": "graph",
            "targets": [
              {
                "expr": "rate(http_requests_total[5m])",
                "legendFormat": "{{service}}"
              }
            ]
          }
        ]
      }
    }
```

## Disaster Recovery

### Backup Strategy

```bash
# Automated backup script
#!/bin/bash
NAMESPACE=vibrox
BACKUP_DIR=/backups/$(date +%Y%m%d)

mkdir -p $BACKUP_DIR

# Backup database
kubectl exec -n $NAMESPACE deployment/postgres -- pg_dump -U postgres postgres > $BACKUP_DIR/database.sql

# Backup configurations
kubectl get configmap -n $NAMESPACE -o yaml > $BACKUP_DIR/configmaps.yml
kubectl get secret -n $NAMESPACE -o yaml > $BACKUP_DIR/secrets.yml

# Backup persistent volumes
kubectl get pvc -n $NAMESPACE -o yaml > $BACKUP_DIR/pvcs.yml
```

### Recovery Procedures

```bash
# Restore from backup
kubectl apply -f backups/20240115/configmaps.yml
kubectl apply -f backups/20240115/secrets.yml
kubectl apply -f backups/20240115/pvcs.yml

# Restore database
kubectl exec -i -n vibrox deployment/postgres -- psql -U postgres postgres < backups/20240115/database.sql
```

---

*This guide should be updated when Kubernetes configuration changes or new deployment requirements are added.*
