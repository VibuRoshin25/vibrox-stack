# Vibrox Stack Architecture Diagram

## System Overview

The Vibrox Stack is a microservices-based application with three core services, a PostgreSQL database, and orchestration through Docker Compose and Kubernetes.

```mermaid
graph TB
    %% External Clients
    Client[Client Applications] --> API[API Gateway/Load Balancer]

    %% Core Services
    API --> Core[vibrox-core<br/>User Management<br/>Go + REST/gRPC<br/>Port: 8080]
    API --> Auth[vibrox-auth<br/>JWT Authentication<br/>Node.js + gRPC<br/>Port: 8000]
    API --> Logger[vibrox-echo<br/>Centralized Logging<br/>Go + gRPC<br/>Port: 9000]

    %% Internal Communication
    Core -.->|gRPC| Auth
    Core -.->|gRPC| Logger
    Auth -.->|gRPC| Logger

    %% Database
    Core --> DB[(PostgreSQL<br/>Database<br/>Port: 5432)]

    %% Infrastructure
    subgraph "Infrastructure Layer"
        Docker[Docker Compose<br/>Local Development]
        K8s[Kubernetes<br/>Production Deployment]
        Volumes[Persistent Volumes<br/>Data Storage]
    end

    %% Service Dependencies
    Core -.->|depends on| DB
    Core -.->|depends on| Auth
    Core -.->|depends on| Logger

    %% Styling
    classDef service fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef database fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef infra fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef client fill:#f1f8e9,stroke:#33691e,stroke-width:2px

    class Core,Auth,Logger service
    class DB database
    class Docker,K8s,Volumes infra
    class Client,API client
```

## Service Details

### vibrox-core (User Management Service)

- **Technology**: Go with Gin framework
- **Protocols**: REST API + gRPC client
- **Port**: 8080
- **Responsibilities**:
  - User CRUD operations
  - Business logic orchestration
  - Database interactions
  - Authentication integration
  - Logging integration

### vibrox-auth (Authentication Service)

- **Technology**: Node.js with gRPC
- **Protocols**: gRPC server
- **Port**: 8000
- **Responsibilities**:
  - JWT token generation
  - Token validation
  - User authentication
  - Security token management

### vibrox-echo (Logging Service)

- **Technology**: Go with gRPC
- **Protocols**: gRPC server
- **Port**: 9000
- **Responsibilities**:
  - Centralized logging
  - Log aggregation
  - Log persistence
  - Log retrieval

### PostgreSQL Database

- **Port**: 5432
- **Purpose**: Primary data store for user data and application state
- **Persistence**: Docker volumes for data persistence

## Communication Patterns

1. **Synchronous gRPC Calls**: Services communicate via gRPC for internal operations
2. **REST API**: External clients interact via REST endpoints
3. **Database Queries**: Direct database access for data persistence
4. **Environment-based Configuration**: Service discovery via environment variables

## Deployment Options

### Docker Compose (Development)

- Single-node deployment
- Local volume mounting
- Easy development setup
- Service discovery via Docker networking

### Kubernetes (Production)

- Multi-node deployment
- Persistent volume claims
- Load balancing
- Health checks and auto-scaling
- Service mesh ready
