# Deployment Architecture and Infrastructure

## Deployment Overview

This diagram illustrates the deployment architecture, infrastructure components, networking, and deployment strategies for the Vibrox Stack across different environments.

```mermaid
graph TB
    %% External Access
    Internet[Internet] --> LoadBalancer[Load Balancer<br/>NGINX/HAProxy]

    %% Kubernetes Production Deployment
    subgraph "Kubernetes Production Cluster"
        subgraph "Ingress Layer"
            Ingress[Ingress Controller<br/>NGINX Ingress]
            CertManager[Certificate Manager<br/>Let's Encrypt]
        end

        subgraph "Application Layer"
            subgraph "vibrox-core Deployment"
                CorePod1[vibrox-core Pod 1<br/>Port: 8080]
                CorePod2[vibrox-core Pod 2<br/>Port: 8080]
                CoreService[vibrox-core Service<br/>ClusterIP]
            end

            subgraph "vibrox-auth Deployment"
                AuthPod1[vibrox-auth Pod 1<br/>Port: 8000]
                AuthPod2[vibrox-auth Pod 2<br/>Port: 8000]
                AuthService[vibrox-auth Service<br/>ClusterIP]
            end

            subgraph "vibrox-echo Deployment"
                LoggerPod1[vibrox-echo Pod 1<br/>Port: 9000]
                LoggerPod2[vibrox-echo Pod 2<br/>Port: 9000]
                LoggerService[vibrox-echo Service<br/>ClusterIP]
            end
        end

        subgraph "Data Layer"
            subgraph "PostgreSQL Deployment"
                DBPod1[PostgreSQL Pod 1<br/>Port: 5432]
                DBPod2[PostgreSQL Pod 2<br/>Port: 5432]
                DBService[PostgreSQL Service<br/>ClusterIP]
            end

            subgraph "Storage"
                PV1[Persistent Volume 1<br/>100GB SSD]
                PV2[Persistent Volume 2<br/>100GB SSD]
                PVC1[Persistent Volume Claim 1]
                PVC2[Persistent Volume Claim 2]
            end
        end

        subgraph "Monitoring & Logging"
            Prometheus[Prometheus<br/>Metrics Collection]
            Grafana[Grafana<br/>Monitoring Dashboard]
            ELKStack[ELK Stack<br/>Log Aggregation]
        end
    end

    %% Docker Compose Development
    subgraph "Docker Compose Development"
        subgraph "Development Services"
            DevCore[dev-vibrox-core<br/>Port: 8080]
            DevAuth[dev-vibrox-auth<br/>Port: 8000]
            DevLogger[dev-vibrox-echo<br/>Port: 9000]
            DevDB[dev-postgresql<br/>Port: 5432]
        end

        subgraph "Development Volumes"
            DevVol1[./vibrox-vol<br/>Database Data]
            DevVol2[./vibrox-echo/logs<br/>Log Files]
        end
    end

    %% Networking
    subgraph "Network Configuration"
        subgraph "Production Network"
            ProdNetwork[Production Network<br/>10.0.0.0/16]
            ProdSubnet1[Subnet 1<br/>10.0.1.0/24]
            ProdSubnet2[Subnet 2<br/>10.0.2.0/24]
        end

        subgraph "Development Network"
            DevNetwork[Development Network<br/>172.20.0.0/16]
            DevSubnet[Subnet<br/>172.20.1.0/24]
        end
    end

    %% Connections
    LoadBalancer --> Ingress
    Ingress --> CertManager
    Ingress --> CoreService
    Ingress --> AuthService
    Ingress --> LoggerService

    CoreService --> CorePod1
    CoreService --> CorePod2
    AuthService --> AuthPod1
    AuthService --> AuthPod2
    LoggerService --> LoggerPod1
    LoggerService --> LoggerPod2
    DBService --> DBPod1
    DBService --> DBPod2

    CorePod1 --> DBService
    CorePod2 --> DBService
    AuthPod1 --> LoggerService
    AuthPod2 --> LoggerService
    CorePod1 --> AuthService
    CorePod2 --> AuthService

    DBPod1 --> PVC1
    DBPod2 --> PVC2
    PVC1 --> PV1
    PVC2 --> PV2

    LoggerPod1 --> ELKStack
    LoggerPod2 --> ELKStack
    CorePod1 --> Prometheus
    CorePod2 --> Prometheus
    AuthPod1 --> Prometheus
    AuthPod2 --> Prometheus

    Prometheus --> Grafana

    %% Development Connections
    DevCore --> DevDB
    DevAuth --> DevLogger
    DevCore --> DevAuth
    DevCore --> DevLogger

    DevDB --> DevVol1
    DevLogger --> DevVol2

    %% Styling
    classDef external fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef ingress fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef service fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef pod fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef storage fill:#ffebee,stroke:#c62828,stroke-width:2px
    classDef monitoring fill:#f1f8e9,stroke:#33691e,stroke-width:2px
    classDef network fill:#e0f2f1,stroke:#004d40,stroke-width:2px

    class Internet,LoadBalancer external
    class Ingress,CertManager ingress
    class CoreService,AuthService,LoggerService,DBService service
    class CorePod1,CorePod2,AuthPod1,AuthPod2,LoggerPod1,LoggerPod2,DBPod1,DBPod2,DevCore,DevAuth,DevLogger,DevDB pod
    class PV1,PV2,PVC1,PVC2,DevVol1,DevVol2 storage
    class Prometheus,Grafana,ELKStack monitoring
    class ProdNetwork,ProdSubnet1,ProdSubnet2,DevNetwork,DevSubnet network
```

## Deployment Environments

### Production Deployment (Kubernetes)

#### Infrastructure Components

- **Load Balancer**: NGINX/HAProxy for external traffic distribution
- **Ingress Controller**: NGINX Ingress for internal routing
- **Certificate Manager**: Let's Encrypt for SSL/TLS certificates
- **Persistent Storage**: SSD-based persistent volumes for data persistence

#### Service Configuration

```yaml
# Example: vibrox-core deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vibrox-core
spec:
  replicas: 2
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
          image: viburoshin25/vibrox-core:latest
          ports:
            - containerPort: 8080
          env:
            - name: DB_HOST
              value: vibrox-db
            - name: AUTH_HOST
              value: vibrox-auth:8000
            - name: LOGGER_HOST
              value: vibrox-echo:9000
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            periodSeconds: 5
```

#### Networking Configuration

- **Production Network**: 10.0.0.0/16 with multiple subnets
- **Service Discovery**: Kubernetes DNS for internal service communication
- **Load Balancing**: Round-robin load balancing across pods
- **Health Checks**: Liveness and readiness probes for service health

### Development Deployment (Docker Compose)

#### Local Development Setup

```yaml
# docker-compose.yml
services:
  db:
    image: postgres
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=server
      - POSTGRES_DB=postgres
    volumes:
      - vibrox-vol:/var/lib/postgresql/data
    networks:
      - app-network

  logger:
    build: ./vibrox-echo
    ports:
      - "9000:9000"
    volumes:
      - ./vibrox-echo/logs:/app/logs
    networks:
      - app-network

  auth:
    build: ./vibrox-auth
    ports:
      - "8000:8000"
    networks:
      - app-network

  app:
    build: ./vibrox-core
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
      - AUTH_HOST=auth:8000
      - LOGGER_HOST=logger:9000
    depends_on:
      - db
      - logger
      - auth
    networks:
      - app-network
```

#### Development Features

- **Hot Reload**: Air for Go services, nodemon for Node.js
- **Volume Mounting**: Direct file system access for development
- **Port Mapping**: Direct access to services for debugging
- **Network Isolation**: Docker network for service communication

## Infrastructure Components

### Storage Solutions

#### Production Storage

- **Persistent Volumes**: SSD-based storage for high performance
- **Volume Claims**: Dynamic provisioning of storage resources
- **Backup Strategy**: Automated backups with retention policies
- **Data Replication**: Multi-zone data replication for availability

#### Development Storage

- **Local Volumes**: Bind-mounted directories for easy access
- **Data Persistence**: Local PostgreSQL data directory
- **Log Storage**: Local log file storage for debugging

### Monitoring and Observability

#### Production Monitoring

- **Prometheus**: Metrics collection and storage
- **Grafana**: Monitoring dashboards and alerting
- **ELK Stack**: Log aggregation and analysis
- **Health Checks**: Automated health monitoring

#### Development Monitoring

- **Local Logs**: Direct access to service logs
- **Docker Logs**: Container-level logging
- **Health Endpoints**: Manual health check verification

## Deployment Strategies

### Blue-Green Deployment

1. **Blue Environment**: Current production environment
2. **Green Environment**: New version deployment
3. **Traffic Switch**: Gradual traffic migration
4. **Rollback Capability**: Quick rollback to blue environment

### Rolling Updates

- **Zero Downtime**: Continuous service availability
- **Gradual Rollout**: Incremental pod updates
- **Health Verification**: Automatic health checks during rollout
- **Rollback on Failure**: Automatic rollback on health check failures

### Canary Deployment

- **Traffic Splitting**: Percentage-based traffic distribution
- **A/B Testing**: Feature comparison in production
- **Gradual Rollout**: Incremental feature releases
- **Metrics Analysis**: Performance and error rate monitoring

## Security and Compliance

### Network Security

- **TLS/SSL**: End-to-end encryption for all communications
- **Network Policies**: Kubernetes network policies for traffic control
- **Firewall Rules**: Network-level access control
- **VPN Access**: Secure administrative access

### Data Security

- **Encryption at Rest**: Database and volume encryption
- **Encryption in Transit**: TLS encryption for all communications
- **Access Control**: Role-based access control (RBAC)
- **Audit Logging**: Comprehensive audit trail

### Compliance

- **GDPR**: Data protection and privacy compliance
- **SOC 2**: Security and availability compliance
- **ISO 27001**: Information security management
- **PCI DSS**: Payment card industry compliance (if applicable)
