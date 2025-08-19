# Vibrox Stack Documentation

Welcome to the Vibrox Stack documentation. This documentation provides comprehensive information about the Vibrox microservices suite, including architecture, deployment, and development guides.

## 📚 Documentation Categories

### 🏗️ Architecture & Design
- **[Architecture Overview](architecture/overview.md)** - System design and component relationships
- **[Service Architecture](architecture/services.md)** - Detailed service breakdown and responsibilities
- **[Data Flow](architecture/data-flow.md)** - Request/response patterns and service communication

### 📋 Architecture Decision Records (ADRs)
- **[ADR Index](adr/README.md)** - All architecture decisions and their rationale
- **[Service Communication](adr/service-communication.md)** - gRPC vs REST decisions
- **[Database Strategy](adr/database-strategy.md)** - PostgreSQL and data management decisions
- **[Deployment Strategy](adr/deployment-strategy.md)** - Docker Compose and Kubernetes decisions

### 🚀 Deployment & Operations
- **[Deployment Guide](deployment/README.md)** - How to deploy the Vibrox stack
- **[Docker Compose Setup](deployment/docker-compose.md)** - Local development with Docker Compose
- **[Kubernetes Setup](deployment/kubernetes.md)** - Production deployment with Kubernetes
- **[Environment Configuration](deployment/environment.md)** - Environment variables and configuration

### 🔧 Development & Onboarding
- **[Developer Onboarding](onboarding/README.md)** - Getting started as a developer
- **[Development Setup](onboarding/development-setup.md)** - Local development environment
- **[Contributing Guidelines](onboarding/contributing.md)** - How to contribute to the project
- **[Testing Guide](onboarding/testing.md)** - Testing strategies and practices

### 🔍 Service Documentation
- **[vibrox-core](services/vibrox-core.md)** - User management service (Go)
- **[vibrox-auth](services/vibrox-auth.md)** - JWT authentication service (Node.js)
- **[vibrox-echo](services/vibrox-echo.md)** - Centralized logging service (Go)

### 🔐 Security
- **[Security Overview](security/overview.md)** - Security architecture and practices
- **[Authentication Flow](security/authentication.md)** - JWT authentication implementation
- **[API Security](security/api-security.md)** - API security measures

### 📊 Diagrams
- **[System Architecture](diagrams/architecture.md)** - High-level system architecture
- **[Service Dependencies](diagrams/dependencies.md)** - Service dependency relationships
- **[Deployment Architecture](diagrams/deployment.md)** - Deployment infrastructure
- **[Data Flow](diagrams/data-flow.md)** - Request/response flow diagrams

## 🎯 Quick Start

1. **For Developers**: Start with [Developer Onboarding](onboarding/README.md)
2. **For DevOps**: Check [Deployment Guide](deployment/README.md)
3. **For Architects**: Review [Architecture Overview](architecture/overview.md)

## 📝 Documentation Maintenance

This documentation is maintained as code alongside the source code. To update documentation:

- Use `/update-docs` to analyze and suggest documentation updates
- Use `/sync-docs` to synchronize all documentation with current codebase
- Use `/check-docs` to validate documentation freshness

## 🔗 Related Resources

- [Main Repository](https://github.com/VibuRoshin25/vibrox-stack)
- [vibrox-core](https://github.com/VibuRoshin25/vibrox-core)
- [vibrox-auth](https://github.com/VibuRoshin25/vibrox-auth)
- [vibrox-echo](https://github.com/VibuRoshin25/vibrox-echo)

---

*Last updated: $(date)*
