# Developer Onboarding Guide

Welcome to the Vibrox Stack! This guide will help you get up and running with the development environment and understand how to contribute to the project.

## 🎯 Quick Start

1. **Prerequisites**: Install required tools
2. **Setup**: Clone and configure the project
3. **Development**: Start the development environment
4. **Contribution**: Learn how to contribute

## 📋 Prerequisites

### Required Tools

- **Git**: Version control system
- **Docker**: Containerization platform
- **Docker Compose**: Multi-container orchestration
- **Go** (1.19+): For vibrox-core and vibrox-echo services
- **Node.js** (18+): For vibrox-auth service
- **PostgreSQL**: Database (optional, Docker provides this)

### Installation

#### Docker & Docker Compose

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install docker.io docker-compose

# macOS
brew install docker docker-compose

# Windows
# Download Docker Desktop from https://www.docker.com/products/docker-desktop
```

#### Go

```bash
# Ubuntu/Debian
sudo apt install golang-go

# macOS
brew install go

# Windows
# Download from https://golang.org/dl/
```

#### Node.js

```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# macOS
brew install node

# Windows
# Download from https://nodejs.org/
```

## 🚀 Project Setup

### 1. Clone the Repository

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/VibuRoshin25/vibrox-stack.git
cd vibrox-stack

# If already cloned, update submodules
git submodule update --init --recursive
```

### 2. Verify Installation

```bash
# Check Docker
docker --version
docker-compose --version

# Check Go
go version

# Check Node.js
node --version
npm --version
```

### 3. Environment Configuration

Each service may have its own environment configuration. Check the individual service directories:

- `vibrox-core/` - Go service configuration
- `vibrox-auth/` - Node.js service configuration
- `vibrox-echo/` - Go service configuration

## 🐳 Development Environment

### Start All Services

```bash
# Start the entire stack
docker-compose up --build

# Start in detached mode
docker-compose up -d --build

# View logs
docker-compose logs -f
```

### Individual Service Development

#### vibrox-core (Go)

```bash
cd vibrox-core
go mod download
go run main.go
```

#### vibrox-auth (Node.js)

```bash
cd vibrox-auth
npm install
npm start
```

#### vibrox-echo (Go)

```bash
cd vibrox-echo
go mod download
go run main.go
```

### Database Access

```bash
# Connect to PostgreSQL
docker-compose exec db psql -U postgres -d postgres

# Or use a GUI client with these credentials:
# Host: localhost
# Port: 5432
# User: postgres
# Password: server
# Database: postgres
```

## 🔧 Development Workflow

### 1. Code Organization

```
vibrox-stack/
├── vibrox-core/          # User management service (Go)
├── vibrox-auth/          # Authentication service (Node.js)
├── vibrox-echo/          # Logging service (Go)
├── manifests/            # Kubernetes manifests
├── volume-manifests/     # Volume configurations
├── docker-compose.yml    # Local development
└── docs/                 # Documentation
```

### 2. Making Changes

1. **Create a feature branch**:

   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes** in the appropriate service directory

3. **Test your changes**:

   ```bash
   # Test the entire stack
   docker-compose up --build

   # Test individual services
   cd vibrox-core && go test ./...
   cd vibrox-auth && npm test
   ```

4. **Commit your changes**:

   ```bash
   git add .
   git commit -m "feat: add your feature description"
   ```

5. **Push and create a pull request**

### 3. Testing

#### Unit Tests

```bash
# Go services
cd vibrox-core
go test ./...

cd vibrox-echo
go test ./...

# Node.js service
cd vibrox-auth
npm test
```

#### Integration Tests

```bash
# Start the stack
docker-compose up -d

# Run integration tests
# (Add your integration test commands here)

# Clean up
docker-compose down
```

## 📚 Learning Resources

### Architecture

- [Architecture Overview](../architecture/overview.md)
- [Service Documentation](../services/)
- [ADR Index](../adr/README.md)

### Development

- [Contributing Guidelines](contributing.md)
- [Testing Guide](testing.md)
- [Development Setup](development-setup.md)

### Deployment

- [Docker Compose Setup](../deployment/docker-compose.md)
- [Kubernetes Setup](../deployment/kubernetes.md)

## 🔍 Troubleshooting

### Common Issues

#### Docker Issues

```bash
# Reset Docker
docker system prune -a
docker volume prune

# Check Docker status
docker info
docker-compose config
```

#### Database Issues

```bash
# Reset database
docker-compose down -v
docker-compose up -d db
```

#### Service Communication Issues

```bash
# Check service logs
docker-compose logs vibrox-core
docker-compose logs vibrox-auth
docker-compose logs vibrox-echo

# Check network connectivity
docker-compose exec vibrox-core ping vibrox-auth
```

### Getting Help

1. **Check the logs**: Use `docker-compose logs` to debug issues
2. **Review documentation**: Check the relevant service documentation
3. **Search issues**: Look for similar issues in the repository
4. **Ask questions**: Create an issue with detailed information

## 🎉 Next Steps

1. **Explore the codebase**: Familiarize yourself with the service structure
2. **Run the tests**: Ensure everything is working correctly
3. **Make a small change**: Try modifying a service to understand the workflow
4. **Read the documentation**: Review architecture and service documentation
5. **Join the community**: Participate in discussions and code reviews

## 📝 Development Guidelines

- **Code Style**: Follow language-specific conventions (Go fmt, ESLint for Node.js)
- **Documentation**: Update documentation when making changes
- **Testing**: Write tests for new features and bug fixes
- **Commits**: Use conventional commit messages
- **Reviews**: Request code reviews for all changes

---

_Need help? Check the [Contributing Guidelines](contributing.md) or create an issue in the repository._
