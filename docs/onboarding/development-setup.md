# Development Setup Guide

## Overview

This guide provides step-by-step instructions for setting up a local development environment for the Vibrox Stack. It covers all necessary tools, configurations, and workflows for effective development.

## Prerequisites

### Required Software

| Software           | Version | Installation                                                            |
| ------------------ | ------- | ----------------------------------------------------------------------- |
| **Git**            | 2.30+   | [Git Installation](https://git-scm.com/downloads)                       |
| **Docker**         | 20.10+  | [Docker Installation](https://docs.docker.com/get-docker/)              |
| **Docker Compose** | 2.0+    | [Docker Compose Installation](https://docs.docker.com/compose/install/) |
| **Go**             | 1.21+   | [Go Installation](https://golang.org/dl/)                               |
| **Node.js**        | 18+     | [Node.js Installation](https://nodejs.org/)                             |
| **PostgreSQL**     | 15+     | [PostgreSQL Installation](https://www.postgresql.org/download/)         |

### System Requirements

- **RAM**: Minimum 4GB, Recommended 8GB+
- **Storage**: At least 10GB free space
- **Network**: Internet connection for pulling images and dependencies
- **OS**: Linux, macOS, or Windows (WSL2 recommended for Windows)

## Installation Guide

### 1. Git Setup

```bash
# Install Git
# Ubuntu/Debian
sudo apt update
sudo apt install git

# macOS
brew install git

# Windows
# Download from https://git-scm.com/download/win

# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Verify installation
git --version
```

### 2. Docker Setup

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install docker.io docker-compose
sudo usermod -aG docker $USER
sudo systemctl enable docker
sudo systemctl start docker

# macOS
brew install docker docker-compose
# Or download Docker Desktop from https://www.docker.com/products/docker-desktop

# Windows
# Download Docker Desktop from https://www.docker.com/products/docker-desktop

# Verify installation
docker --version
docker-compose --version
```

### 3. Go Setup

```bash
# Ubuntu/Debian
sudo apt install golang-go

# macOS
brew install go

# Windows
# Download from https://golang.org/dl/

# Set GOPATH (if not using Go modules)
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# Verify installation
go version
```

### 4. Node.js Setup

```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# macOS
brew install node

# Windows
# Download from https://nodejs.org/

# Verify installation
node --version
npm --version
```

### 5. PostgreSQL Setup (Optional)

```bash
# Ubuntu/Debian
sudo apt install postgresql postgresql-contrib
sudo systemctl enable postgresql
sudo systemctl start postgresql

# macOS
brew install postgresql
brew services start postgresql

# Windows
# Download from https://www.postgresql.org/download/windows/

# Create database user
sudo -u postgres createuser --interactive
sudo -u postgres createdb vibrox_dev
```

## Project Setup

### 1. Clone Repository

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/VibuRoshin25/vibrox-stack.git
cd vibrox-stack

# If already cloned, update submodules
git submodule update --init --recursive
```

### 2. Verify Repository Structure

```bash
# Check repository structure
ls -la

# Expected structure:
# vibrox-core/     - Go service (User management)
# vibrox-auth/     - Node.js service (Authentication)
# vibrox-echo/     - Go service (Logging)
# manifests/       - Kubernetes manifests
# docker-compose.yml
# README.md
```

### 3. Environment Configuration

```bash
# Create environment files for each service
cp vibrox-core/.env.example vibrox-core/.env
cp vibrox-auth/.env.example vibrox-auth/.env
cp vibrox-echo/.env.example vibrox-echo/.env

# Edit environment files with your configuration
# See individual service documentation for required variables
```

## Development Workflows

### Option 1: Docker Compose (Recommended)

#### Quick Start

```bash
# Start all services
docker-compose up --build

# Start in detached mode
docker-compose up -d --build

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

#### Individual Service Development

```bash
# Start only database and dependencies
docker-compose up -d db logger

# Run vibrox-core locally
cd vibrox-core
go mod download
go run main.go

# Run vibrox-auth locally
cd vibrox-auth
npm install
npm start

# Run vibrox-echo locally
cd vibrox-echo
go mod download
go run main.go
```

### Option 2: Local Development

#### Database Setup

```bash
# Start PostgreSQL locally
sudo systemctl start postgresql

# Create database and user
sudo -u postgres psql
CREATE DATABASE vibrox_dev;
CREATE USER vibrox_user WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE vibrox_dev TO vibrox_user;
\q

# Or use Docker for database only
docker run -d \
  --name postgres-dev \
  -e POSTGRES_DB=vibrox_dev \
  -e POSTGRES_USER=vibrox_user \
  -e POSTGRES_PASSWORD=password \
  -p 5432:5432 \
  postgres:15
```

#### Service Development

```bash
# vibrox-core (Go)
cd vibrox-core
export DB_HOST=localhost
export DB_USER=vibrox_user
export DB_PASSWORD=password
export DB_NAME=vibrox_dev
export AUTH_HOST=localhost:8000
export LOGGER_HOST=localhost:9000
go run main.go

# vibrox-auth (Node.js)
cd vibrox-auth
export DB_HOST=localhost
export DB_USER=vibrox_user
export DB_PASSWORD=password
export DB_NAME=vibrox_dev
export LOGGER_HOST=localhost:9000
export JWT_SECRET=your-secret-key
npm start

# vibrox-echo (Go)
cd vibrox-echo
export LOG_LEVEL=DEBUG
export LOG_PATH=./logs
export LOG_FORMAT=json
go run main.go
```

## IDE Setup

### VS Code Configuration

#### Extensions

Install the following VS Code extensions:

- **Go** (golang.go)
- **Node.js Extension Pack** (ms-vscode.vscode-node-extension-pack)
- **Docker** (ms-azuretools.vscode-docker)
- **YAML** (redhat.vscode-yaml)
- **GitLens** (eamodio.gitlens)
- **Thunder Client** (rangav.vscode-thunder-client)

#### Workspace Settings

Create `.vscode/settings.json`:

```json
{
  "go.useLanguageServer": true,
  "go.lintTool": "golangci-lint",
  "go.formatTool": "goimports",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  },
  "files.exclude": {
    "**/node_modules": true,
    "**/vendor": true,
    "**/logs": true
  }
}
```

#### Launch Configuration

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch vibrox-core",
      "type": "go",
      "request": "launch",
      "mode": "auto",
      "program": "${workspaceFolder}/vibrox-core/main.go",
      "env": {
        "DB_HOST": "localhost",
        "DB_USER": "vibrox_user",
        "DB_PASSWORD": "password",
        "DB_NAME": "vibrox_dev"
      }
    },
    {
      "name": "Launch vibrox-auth",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/vibrox-auth/src/index.js",
      "env": {
        "DB_HOST": "localhost",
        "DB_USER": "vibrox_user",
        "DB_PASSWORD": "password",
        "DB_NAME": "vibrox_dev"
      }
    }
  ]
}
```

### GoLand/IntelliJ Setup

#### Project Configuration

1. Open the `vibrox-stack` directory as a project
2. Configure Go SDK (File → Project Structure → Project SDK)
3. Enable Go modules
4. Install Go plugins if not already installed

#### Run Configurations

Create run configurations for each service:

1. **vibrox-core**: Go Build configuration
2. **vibrox-auth**: Node.js configuration
3. **vibrox-echo**: Go Build configuration

## Testing Setup

### Unit Tests

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

### Integration Tests

```bash
# Start test environment
docker-compose -f docker-compose.test.yml up -d

# Run integration tests
cd vibrox-core
go test -tags=integration ./...

# Clean up
docker-compose -f docker-compose.test.yml down
```

### End-to-End Tests

```bash
# Start full stack
docker-compose up -d

# Run E2E tests
npm run test:e2e

# Stop services
docker-compose down
```

## Debugging

### Go Debugging

```bash
# Run with debug flags
go run -race main.go

# Use delve debugger
go install github.com/go-delve/delve/cmd/dlv@latest
dlv debug main.go

# Profile with pprof
go run main.go &
curl http://localhost:8080/debug/pprof/profile > profile.out
go tool pprof profile.out
```

### Node.js Debugging

```bash
# Run with debug flags
node --inspect src/index.js

# Use nodemon for development
npm install -g nodemon
nodemon src/index.js

# Profile with clinic
npm install -g clinic
clinic doctor -- node src/index.js
```

### Database Debugging

```bash
# Connect to database
docker-compose exec db psql -U postgres -d postgres

# Or local PostgreSQL
psql -h localhost -U vibrox_user -d vibrox_dev

# Common queries
\dt                    # List tables
\d users              # Describe table
SELECT * FROM users;   # Query data
```

## Performance Monitoring

### Application Metrics

```bash
# Enable metrics endpoints
curl http://localhost:8080/metrics

# Monitor with Prometheus
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

### Resource Monitoring

```bash
# Monitor Docker resources
docker stats

# Monitor system resources
htop
iotop
nethogs
```

## Troubleshooting

### Common Issues

#### 1. Port Conflicts

```bash
# Check if ports are in use
netstat -tulpn | grep :8080
netstat -tulpn | grep :8000
netstat -tulpn | grep :9000
netstat -tulpn | grep :5432

# Kill processes using ports
sudo lsof -ti:8080 | xargs kill -9
```

#### 2. Docker Issues

```bash
# Reset Docker
docker system prune -a
docker volume prune

# Check Docker status
docker info
docker-compose config
```

#### 3. Database Issues

```bash
# Reset database
docker-compose down -v
docker-compose up -d db

# Check database logs
docker-compose logs db

# Reset local PostgreSQL
sudo systemctl restart postgresql
```

#### 4. Go Module Issues

```bash
# Clean module cache
go clean -modcache

# Download dependencies
go mod download

# Verify dependencies
go mod verify
```

#### 5. Node.js Issues

```bash
# Clear npm cache
npm cache clean --force

# Remove node_modules
rm -rf node_modules package-lock.json
npm install

# Check Node.js version
node --version
npm --version
```

### Getting Help

1. **Check logs**: Use `docker-compose logs` to debug issues
2. **Review documentation**: Check service-specific documentation
3. **Search issues**: Look for similar issues in the repository
4. **Ask questions**: Create an issue with detailed information

## Best Practices

### Code Organization

1. **Follow language conventions**: Go fmt, ESLint for Node.js
2. **Use meaningful names**: Clear variable and function names
3. **Add comments**: Explain complex logic and decisions
4. **Keep functions small**: Single responsibility principle

### Git Workflow

1. **Create feature branches**: `git checkout -b feature/your-feature`
2. **Make small commits**: Atomic, focused changes
3. **Write good commit messages**: Clear and descriptive
4. **Test before committing**: Run tests locally
5. **Pull before pushing**: Avoid merge conflicts

### Testing

1. **Write unit tests**: For all business logic
2. **Use integration tests**: For service interactions
3. **Test edge cases**: Error conditions and boundaries
4. **Maintain test data**: Use fixtures and factories

### Performance

1. **Profile regularly**: Identify bottlenecks
2. **Use connection pooling**: For database connections
3. **Implement caching**: For frequently accessed data
4. **Monitor resources**: CPU, memory, disk usage

---

_This development setup guide should be updated when new tools or workflows are added to the project._
