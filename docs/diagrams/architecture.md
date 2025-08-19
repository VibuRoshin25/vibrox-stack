# System Architecture Diagrams

This document contains comprehensive architecture diagrams for the Vibrox Stack, providing visual representations of the system components, data flow, and deployment architecture.

## 🏗️ High-Level System Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        Web[Web Applications]
        Mobile[Mobile Apps]
        API[API Clients]
    end
    
    subgraph "API Gateway / Load Balancer"
        LB[Load Balancer<br/>NGINX/HAProxy]
    end
    
    subgraph "Application Layer"
        Core[vibrox-core<br/>User Management<br/>Go + REST/gRPC<br/>Port: 8080]
        Auth[vibrox-auth<br/>Authentication<br/>Node.js + gRPC<br/>Port: 8000]
        Logger[vibrox-echo<br/>Logging Service<br/>Go + gRPC<br/>Port: 9000]
    end
    
    subgraph "Data Layer"
        DB[(PostgreSQL<br/>Primary Database<br/>Port: 5432)]
        Logs[(Log Files<br/>Local Storage)]
    end
    
    subgraph "Infrastructure"
        Docker[Docker Compose<br/>Local Development]
        K8s[Kubernetes<br/>Production]
        Volumes[Persistent Volumes]
    end
    
    Web --> LB
    Mobile --> LB
    API --> LB
    
    LB --> Core
    LB --> Auth
    
    Core --> Auth
    Core --> Logger
    Core --> DB
    
    Auth --> Logger
    Auth --> DB
    
    Logger --> Logs
    
    Docker --> Volumes
    K8s --> Volumes
```

## 🔄 Service Communication Flow

```mermaid
sequenceDiagram
    participant Client
    participant Core as vibrox-core
    participant Auth as vibrox-auth
    participant DB as PostgreSQL
    participant Logger as vibrox-echo
    
    Note over Client,Logger: User Registration Flow
    Client->>Core: POST /api/auth/register
    Core->>Auth: gRPC: ValidateRegistration
    Auth->>DB: Check existing user
    DB-->>Auth: User not found
    Auth-->>Core: Registration valid
    Core->>DB: Create user
    DB-->>Core: User created
    Core->>Logger: gRPC: LogEvent(INFO, "User registered")
    Core-->>Client: 201: User created
    
    Note over Client,Logger: User Login Flow
    Client->>Core: POST /api/auth/login
    Core->>Auth: gRPC: AuthenticateUser
    Auth->>DB: Validate credentials
    DB-->>Auth: User found
    Auth->>Auth: Generate JWT token
    Auth-->>Core: JWT token
    Core->>Logger: gRPC: LogEvent(INFO, "User logged in")
    Core-->>Client: 200: Login successful + token
    
    Note over Client,Logger: Protected API Call
    Client->>Core: GET /api/users (with JWT)
    Core->>Auth: gRPC: ValidateToken
    Auth-->>Core: Token valid
    Core->>DB: Fetch users
    DB-->>Core: User list
    Core->>Logger: gRPC: LogEvent(INFO, "Users fetched")
    Core-->>Client: 200: User list
```

## 🗄️ Database Schema Architecture

```mermaid
erDiagram
    USERS {
        int id PK
        string username UK
        string email UK
        string password_hash
        timestamp created_at
        timestamp updated_at
    }
    
    SESSIONS {
        int id PK
        int user_id FK
        string token_hash UK
        timestamp expires_at
        timestamp created_at
    }
    
    LOGS {
        int id PK
        string level
        string message
        int user_id FK
        string service
        timestamp created_at
    }
    
    USERS ||--o{ SESSIONS : "has"
    USERS ||--o{ LOGS : "generates"
```

## 🐳 Docker Compose Architecture

```mermaid
graph TB
    subgraph "Docker Compose Network"
        subgraph "Services"
            App[vibrox-core<br/>Container]
            Auth[vibrox-auth<br/>Container]
            Logger[vibrox-echo<br/>Container]
            DB[PostgreSQL<br/>Container]
        end
        
        subgraph "Volumes"
            DBVol[Database Volume<br/>vibrox-vol]
            LogVol[Log Volume<br/>vibrox-echo-vol]
        end
        
        subgraph "Ports"
            P8080[Port 8080<br/>vibrox-core]
            P8000[Port 8000<br/>vibrox-auth]
            P9000[Port 9000<br/>vibrox-echo]
            P5432[Port 5432<br/>PostgreSQL]
        end
    end
    
    App --> Auth
    App --> Logger
    App --> DB
    Auth --> Logger
    Auth --> DB
    
    DB --> DBVol
    Logger --> LogVol
    
    App --> P8080
    Auth --> P8000
    Logger --> P9000
    DB --> P5432
```

## ☸️ Kubernetes Deployment Architecture

```mermaid
graph TB
    subgraph "Kubernetes Cluster"
        subgraph "Namespace: vibrox"
            subgraph "Deployments"
                CoreDeploy[vibrox-core<br/>Deployment<br/>Replicas: 3]
                AuthDeploy[vibrox-auth<br/>Deployment<br/>Replicas: 2]
                LoggerDeploy[vibrox-echo<br/>Deployment<br/>Replicas: 2]
                DBDeploy[PostgreSQL<br/>StatefulSet<br/>Replicas: 1]
            end
            
            subgraph "Services"
                CoreSvc[app-service<br/>ClusterIP]
                AuthSvc[auth-service<br/>ClusterIP]
                LoggerSvc[log-service<br/>ClusterIP]
                DBSvc[db-service<br/>ClusterIP]
            end
            
            subgraph "Ingress"
                Ingress[NGINX Ingress<br/>Load Balancer]
            end
            
            subgraph "Persistent Volumes"
                DBPV[Database PV<br/>Storage: 10Gi]
                LogPV[Log PV<br/>Storage: 5Gi]
            end
            
            subgraph "ConfigMaps & Secrets"
                Config[ConfigMap<br/>Environment Variables]
                Secrets[Secrets<br/>Passwords & Keys]
            end
        end
    end
    
    subgraph "External"
        Client[Client Applications]
        Storage[Cloud Storage]
    end
    
    Client --> Ingress
    Ingress --> CoreSvc
    Ingress --> AuthSvc
    
    CoreSvc --> CoreDeploy
    AuthSvc --> AuthDeploy
    LoggerSvc --> LoggerDeploy
    DBSvc --> DBDeploy
    
    CoreDeploy --> AuthSvc
    CoreDeploy --> LoggerSvc
    CoreDeploy --> DBSvc
    
    AuthDeploy --> LoggerSvc
    AuthDeploy --> DBSvc
    
    DBDeploy --> DBPV
    LoggerDeploy --> LogPV
    
    CoreDeploy --> Config
    AuthDeploy --> Config
    LoggerDeploy --> Config
    
    CoreDeploy --> Secrets
    AuthDeploy --> Secrets
    DBDeploy --> Secrets
    
    LoggerDeploy --> Storage
```

## 🔐 Security Architecture

```mermaid
graph TB
    subgraph "Security Layers"
        subgraph "Network Security"
            Firewall[Network Firewall]
            WAF[Web Application Firewall]
        end
        
        subgraph "Application Security"
            Auth[Authentication<br/>JWT Tokens]
            Authz[Authorization<br/>Role-based Access]
            Validation[Input Validation]
            Encryption[Data Encryption]
        end
        
        subgraph "Infrastructure Security"
            RBAC[Kubernetes RBAC]
            NetworkPolicy[Network Policies]
            Secrets[Secret Management]
        end
    end
    
    subgraph "Components"
        Client[Client Applications]
        Core[vibrox-core]
        Auth[vibrox-auth]
        DB[PostgreSQL]
    end
    
    Client --> Firewall
    Firewall --> WAF
    WAF --> Auth
    WAF --> Core
    
    Auth --> Auth
    Core --> Authz
    Core --> Validation
    
    Auth --> Encryption
    Core --> Encryption
    DB --> Encryption
    
    Auth --> RBAC
    Core --> NetworkPolicy
    Auth --> Secrets
    Core --> Secrets
```

## 📊 Monitoring & Observability

```mermaid
graph TB
    subgraph "Application Layer"
        Core[vibrox-core]
        Auth[vibrox-auth]
        Logger[vibrox-echo]
    end
    
    subgraph "Monitoring Stack"
        Prometheus[Prometheus<br/>Metrics Collection]
        Grafana[Grafana<br/>Visualization]
        AlertManager[Alert Manager<br/>Alerting]
    end
    
    subgraph "Logging Stack"
        Fluentd[Fluentd<br/>Log Aggregation]
        Elasticsearch[Elasticsearch<br/>Log Storage]
        Kibana[Kibana<br/>Log Visualization]
    end
    
    subgraph "Tracing"
        Jaeger[Jaeger<br/>Distributed Tracing]
    end
    
    Core --> Prometheus
    Auth --> Prometheus
    Logger --> Prometheus
    
    Core --> Fluentd
    Auth --> Fluentd
    Logger --> Fluentd
    
    Core --> Jaeger
    Auth --> Jaeger
    Logger --> Jaeger
    
    Prometheus --> Grafana
    Prometheus --> AlertManager
    
    Fluentd --> Elasticsearch
    Elasticsearch --> Kibana
```

## 🔄 Data Flow Architecture

```mermaid
flowchart TD
    Start([Client Request]) --> Auth{Authentication Required?}
    
    Auth -->|Yes| Validate[Validate JWT Token]
    Auth -->|No| Process[Process Request]
    
    Validate --> Valid{Token Valid?}
    Valid -->|No| Unauthorized[Return 401 Unauthorized]
    Valid -->|Yes| Process
    
    Process --> Type{Request Type?}
    
    Type -->|User Management| UserOps[User Operations]
    Type -->|Authentication| AuthOps[Authentication Operations]
    Type -->|Logging| LogOps[Logging Operations]
    
    UserOps --> DB[(Database)]
    AuthOps --> DB
    LogOps --> Logs[(Log Files)]
    
    DB --> Response[Generate Response]
    Logs --> Response
    
    Response --> Log[Log Event]
    Log --> End([Return Response])
    
    Unauthorized --> End
```

## 🚀 Deployment Pipeline

```mermaid
graph LR
    subgraph "Development"
        Code[Code Changes]
        Tests[Unit Tests]
        Build[Build Images]
    end
    
    subgraph "Staging"
        DeployStaging[Deploy to Staging]
        IntegrationTests[Integration Tests]
        PerformanceTests[Performance Tests]
    end
    
    subgraph "Production"
        DeployProd[Deploy to Production]
        HealthChecks[Health Checks]
        Monitoring[Monitoring]
    end
    
    Code --> Tests
    Tests --> Build
    Build --> DeployStaging
    DeployStaging --> IntegrationTests
    IntegrationTests --> PerformanceTests
    PerformanceTests --> DeployProd
    DeployProd --> HealthChecks
    HealthChecks --> Monitoring
```

---

*These diagrams should be updated when architectural changes are made. Use `/gen-arch-diagram` to regenerate architecture diagrams from the current codebase.*
