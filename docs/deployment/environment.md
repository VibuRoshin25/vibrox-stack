# Environment Configuration Guide

## Overview

This guide covers environment configuration for the Vibrox Stack across different deployment environments. Proper configuration management is essential for security, performance, and maintainability.

## Environment Types

### Development Environment

- **Purpose**: Local development and testing
- **Security**: Lower security requirements
- **Performance**: Optimized for development speed
- **Data**: Test data, can be reset frequently

### Staging Environment

- **Purpose**: Pre-production testing
- **Security**: Similar to production
- **Performance**: Production-like performance
- **Data**: Anonymized production data

### Production Environment

- **Purpose**: Live user-facing services
- **Security**: Highest security requirements
- **Performance**: Optimized for production load
- **Data**: Real user data, high availability

## Configuration Management

### Environment Variables

#### Database Configuration

```bash
# PostgreSQL Configuration
DB_HOST=localhost          # Database host
DB_PORT=5432              # Database port
DB_USER=postgres          # Database username
DB_PASSWORD=server        # Database password
DB_NAME=postgres          # Database name
DB_SSL_MODE=disable       # SSL mode (disable/require/verify-ca/verify-full)
DB_MAX_CONNECTIONS=20     # Maximum database connections
DB_CONNECTION_TIMEOUT=30s # Connection timeout
DB_IDLE_TIMEOUT=300s      # Idle connection timeout
```

#### Service Configuration

```bash
# vibrox-core Configuration
PORT=8080                 # Service port
HOST=0.0.0.0             # Service host
LOG_LEVEL=INFO           # Logging level (DEBUG/INFO/WARN/ERROR)
AUTH_HOST=auth:8000      # Authentication service host
LOGGER_HOST=logger:9000  # Logging service host

# vibrox-auth Configuration
JWT_SECRET=your-secret-key           # JWT signing secret
JWT_EXPIRES_IN=24h                   # Token expiration time
JWT_REFRESH_EXPIRES_IN=168h          # Refresh token expiration
BCRYPT_ROUNDS=12                     # Password hashing rounds

# vibrox-echo Configuration
LOG_PATH=./logs                      # Log file path
LOG_FORMAT=json                      # Log format (json/text)
LOG_MAX_SIZE=100                     # Max log file size (MB)
LOG_MAX_FILES=10                     # Max number of log files

# vibrox-dns Configuration
DNS_LISTEN_ADDR=:2053                # UDP/TCP listen address
DNS_UPSTREAM=127.0.0.11:53           # Compose embedded resolver (Compose default)
DNS_TIMEOUT=3s                       # Per-query timeout
HEALTH_LISTEN_ADDR=:8053             # HTTP health endpoint
```

#### Security Configuration

```bash
# Security Settings
CORS_ORIGIN=*                        # CORS allowed origins
RATE_LIMIT_WINDOW=15m                # Rate limiting window
RATE_LIMIT_MAX_REQUESTS=100          # Max requests per window
SESSION_SECRET=your-session-secret   # Session secret
ENCRYPTION_KEY=your-encryption-key   # Data encryption key
```

### Configuration Files

#### Docker Compose Environment

```yaml
# docker-compose.yml
version: "3.8"

services:
  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=${DB_USER:-postgres}
      - POSTGRES_PASSWORD=${DB_PASSWORD:-server}
      - POSTGRES_DB=${DB_NAME:-postgres}
    env_file:
      - .env.database

  app:
    build: ./vibrox-core
    environment:
      - DB_HOST=${DB_HOST:-db}
      - DB_USER=${DB_USER:-postgres}
      - DB_PASSWORD=${DB_PASSWORD:-server}
      - DB_NAME=${DB_NAME:-postgres}
      - AUTH_HOST=${AUTH_HOST:-auth:8000}
      - LOGGER_HOST=${LOGGER_HOST:-logger:9000}
    env_file:
      - .env.app
```

#### Kubernetes ConfigMaps

```yaml
# configmap.yml
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
  LOG_FORMAT: json
  CORS_ORIGIN: "*"
  RATE_LIMIT_WINDOW: "15m"
  RATE_LIMIT_MAX_REQUESTS: "100"
```

#### Kubernetes Secrets

```yaml
# secrets.yml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: vibrox
type: Opaque
data:
  DB_PASSWORD: c2VydmVy # base64 encoded
  JWT_SECRET: eW91ci1zZWNyZXQ= # base64 encoded
  SESSION_SECRET: c2Vzc2lvbi1zZWNyZXQ= # base64 encoded
  ENCRYPTION_KEY: ZW5jcnlwdGlvbi1rZXk= # base64 encoded
```

## Environment-Specific Configurations

### Development-Environment

#### .env.development

```bash
# Development Environment Variables
NODE_ENV=development
LOG_LEVEL=DEBUG
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=server
DB_NAME=vibrox_dev
DB_SSL_MODE=disable

# Service Configuration
AUTH_HOST=localhost:8000
LOGGER_HOST=localhost:9000
PORT=8080

# Security (Development)
JWT_SECRET=dev-secret-key
CORS_ORIGIN=*
RATE_LIMIT_MAX_REQUESTS=1000

# Logging
LOG_PATH=./logs
LOG_FORMAT=text
LOG_MAX_SIZE=10
LOG_MAX_FILES=5
```

#### Docker Compose Development

```yaml
# docker-compose.dev.yml
version: "3.8"

services:
  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=server
      - POSTGRES_DB=vibrox_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_dev_data:/var/lib/postgresql/data

  app:
    build:
      context: ./vibrox-core
      dockerfile: Dockerfile.dev
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=DEBUG
    volumes:
      - ./vibrox-core:/app
      - /app/node_modules
    ports:
      - "8080:8080"
    command: ["go", "run", "main.go"]

volumes:
  postgres_dev_data:
```

### Staging-Environment

#### .env.staging

```bash
# Staging Environment Variables
NODE_ENV=staging
LOG_LEVEL=INFO
DB_HOST=postgres-staging
DB_PORT=5432
DB_USER=vibrox_staging
DB_PASSWORD=${DB_PASSWORD}
DB_NAME=vibrox_staging
DB_SSL_MODE=require

# Service Configuration
AUTH_HOST=vibrox-auth-staging:8000
LOGGER_HOST=vibrox-echo-staging:9000
PORT=8080

# Security (Staging)
JWT_SECRET=${JWT_SECRET}
CORS_ORIGIN=https://staging.vibrox.com
RATE_LIMIT_MAX_REQUESTS=500

# Logging
LOG_PATH=/var/log/vibrox
LOG_FORMAT=json
LOG_MAX_SIZE=100
LOG_MAX_FILES=10
```

#### Kubernetes Staging

```yaml
# staging-configmap.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: staging-config
  namespace: vibrox-staging
data:
  NODE_ENV: staging
  LOG_LEVEL: INFO
  DB_HOST: postgres-staging-service
  DB_USER: vibrox_staging
  DB_NAME: vibrox_staging
  AUTH_HOST: vibrox-auth-staging-service:8000
  LOGGER_HOST: vibrox-echo-staging-service:9000
  CORS_ORIGIN: https://staging.vibrox.com
  RATE_LIMIT_MAX_REQUESTS: "500"
```

### Production-Environment

#### .env.production

```bash
# Production Environment Variables
NODE_ENV=production
LOG_LEVEL=WARN
DB_HOST=postgres-production
DB_PORT=5432
DB_USER=vibrox_production
DB_PASSWORD=${DB_PASSWORD}
DB_NAME=vibrox_production
DB_SSL_MODE=verify-full

# Service Configuration
AUTH_HOST=vibrox-auth-production:8000
LOGGER_HOST=vibrox-echo-production:9000
PORT=8080

# Security (Production)
JWT_SECRET=${JWT_SECRET}
CORS_ORIGIN=https://api.vibrox.com
RATE_LIMIT_MAX_REQUESTS=100
SESSION_SECRET=${SESSION_SECRET}
ENCRYPTION_KEY=${ENCRYPTION_KEY}

# Logging
LOG_PATH=/var/log/vibrox
LOG_FORMAT=json
LOG_MAX_SIZE=500
LOG_MAX_FILES=20
```

#### Kubernetes Production

```yaml
# production-configmap.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: production-config
  namespace: vibrox-production
data:
  NODE_ENV: production
  LOG_LEVEL: WARN
  DB_HOST: postgres-production-service
  DB_USER: vibrox_production
  DB_NAME: vibrox_production
  AUTH_HOST: vibrox-auth-production-service:8000
  LOGGER_HOST: vibrox-echo-production-service:9000
  CORS_ORIGIN: https://api.vibrox.com
  RATE_LIMIT_MAX_REQUESTS: "100"
  LOG_MAX_SIZE: "500"
  LOG_MAX_FILES: "20"
```

## Security Best Practices

### Secret Management

#### Environment-Variables

```bash
# Never commit secrets to version control
# Use environment variables for sensitive data
export DB_PASSWORD="your-secure-password"
export JWT_SECRET="your-jwt-secret"
export SESSION_SECRET="your-session-secret"
```

#### Docker Secrets

```yaml
# docker-compose.yml with secrets
version: "3.8"

services:
  app:
    image: vibrox-core:latest
    secrets:
      - db_password
      - jwt_secret
    environment:
      - DB_PASSWORD_FILE=/run/secrets/db_password
      - JWT_SECRET_FILE=/run/secrets/jwt_secret

secrets:
  db_password:
    file: ./secrets/db_password.txt
  jwt_secret:
    file: ./secrets/jwt_secret.txt
```

#### Kubernetes-Secrets

```yaml
# Create secrets
kubectl create secret generic app-secrets \
  --from-literal=DB_PASSWORD=your-secure-password \
  --from-literal=JWT_SECRET=your-jwt-secret \
  --from-literal=SESSION_SECRET=your-session-secret

# Use secrets in deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vibrox-core
spec:
  template:
    spec:
      containers:
      - name: vibrox-core
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: DB_PASSWORD
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: JWT_SECRET
```

### SSL/TLS Configuration

#### Database SSL

```bash
# Development (no SSL)
DB_SSL_MODE=disable

# Staging (require SSL)
DB_SSL_MODE=require

# Production (verify SSL)
DB_SSL_MODE=verify-full
DB_SSL_CERT=/path/to/client-cert.pem
DB_SSL_KEY=/path/to/client-key.pem
DB_SSL_ROOT_CERT=/path/to/ca-cert.pem
```

#### Application SSL

```yaml
# Kubernetes Ingress with SSL
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vibrox-ingress
  annotations:
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

## Configuration Validation

### Environment Validation

```go
// config/validation.go
package config

import (
    "fmt"
    "os"
    "strconv"
)

type Config struct {
    Database DatabaseConfig
    Security SecurityConfig
    Logging  LoggingConfig
}

func (c *Config) Validate() error {
    if err := c.Database.Validate(); err != nil {
        return fmt.Errorf("database config: %w", err)
    }

    if err := c.Security.Validate(); err != nil {
        return fmt.Errorf("security config: %w", err)
    }

    if err := c.Logging.Validate(); err != nil {
        return fmt.Errorf("logging config: %w", err)
    }

    return nil
}

func (c *DatabaseConfig) Validate() error {
    if c.Host == "" {
        return fmt.Errorf("DB_HOST is required")
    }

    if c.Port <= 0 || c.Port > 65535 {
        return fmt.Errorf("invalid DB_PORT: %d", c.Port)
    }

    if c.User == "" {
        return fmt.Errorf("DB_USER is required")
    }

    if c.Password == "" {
        return fmt.Errorf("DB_PASSWORD is required")
    }

    return nil
}
```

### Configuration Testing

```go
// config/config_test.go
package config

import (
    "os"
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestConfigValidation(t *testing.T) {
    // Test valid configuration
    config := &Config{
        Database: DatabaseConfig{
            Host:     "localhost",
            Port:     5432,
            User:     "postgres",
            Password: "password",
            Name:     "vibrox",
        },
    }

    err := config.Validate()
    assert.NoError(t, err)

    // Test invalid configuration
    config.Database.Host = ""
    err = config.Validate()
    assert.Error(t, err)
    assert.Contains(t, err.Error(), "DB_HOST is required")
}
```

## Configuration Management Tools

### External Configuration

#### HashiCorp Vault

```bash
# Install Vault
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install vault

# Configure Vault
vault server -dev

# Store secrets
vault kv put secret/vibrox DB_PASSWORD=secure-password JWT_SECRET=jwt-secret

# Retrieve secrets
vault kv get secret/vibrox
```

#### AWS Secrets Manager

```bash
# Store secret
aws secretsmanager create-secret \
    --name vibrox/database \
    --description "Vibrox database credentials" \
    --secret-string '{"DB_PASSWORD":"secure-password","DB_USER":"vibrox"}'

# Retrieve secret
aws secretsmanager get-secret-value --secret-id vibrox/database
```

### Configuration Monitoring

#### Health Checks

```go
// health/config.go
package health

import (
    "context"
    "database/sql"
    "time"
)

func CheckDatabaseConfig(ctx context.Context, db *sql.DB) error {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()

    return db.PingContext(ctx)
}

func CheckEnvironmentConfig() error {
    required := []string{
        "DB_HOST", "DB_USER", "DB_PASSWORD", "DB_NAME",
        "JWT_SECRET", "AUTH_HOST", "LOGGER_HOST",
    }

    for _, env := range required {
        if os.Getenv(env) == "" {
            return fmt.Errorf("required environment variable %s is not set", env)
        }
    }

    return nil
}
```

## Troubleshooting

### Common Configuration Issues

#### Database Connection Issues

```bash
# Check database connectivity
psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME

# Test connection with SSL
psql "postgresql://$DB_USER:$DB_PASSWORD@$DB_HOST:$DB_PORT/$DB_NAME?sslmode=require"

# Check environment variables
env | grep DB_
```

#### Service Communication Issues

```bash
# Test service connectivity
curl -v http://$AUTH_HOST/health
curl -v http://$LOGGER_HOST/health

# Check DNS resolution
nslookup $AUTH_HOST
nslookup $LOGGER_HOST

# Test gRPC connectivity
grpcurl -plaintext $AUTH_HOST:8000 list
```

#### Configuration-Validation

```bash
# Validate configuration
go run cmd/validate-config/main.go

# Check configuration files
docker-compose config

# Validate Kubernetes manifests
kubectl apply --dry-run=client -f manifests/
```

---

_This environment configuration guide should be updated when new configuration options or deployment environments are added._
