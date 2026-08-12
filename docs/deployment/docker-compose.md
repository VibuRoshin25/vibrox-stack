# Docker Compose Setup Guide

## Overview

This guide covers setting up and running the Vibrox Stack locally using Docker Compose for development and testing purposes.

## Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- Git with submodules
- At least 4GB RAM available
- 10GB free disk space

## Quick Start

### 1. Clone the Repository

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/VibuRoshin25/vibrox-stack.git
cd vibrox-stack

# If already cloned, update submodules
git submodule update --init --recursive
```

### 2. Start All Services

```bash
# Start all services in detached mode
docker-compose up -d --build

# Or start with logs visible
docker-compose up --build
```

### 3. Verify Services

```bash
# Check service status
docker-compose ps

# View logs
docker-compose logs -f

# Test endpoints
curl http://localhost:8080/health
```

## Service Configuration

### Docker Compose File Structure

```yaml
# docker-compose.yml
version: "3.8"

services:
  # PostgreSQL Database
  db:
    image: postgres:15
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=server
      - POSTGRES_DB=postgres
    networks:
      - app-network
    volumes:
      - vibrox-vol:/var/lib/postgresql/data

  # Logging Service
  logger:
    build: ./vibrox-echo
    ports:
      - "9000:9000"
    networks:
      - app-network
    volumes:
      - ./vibrox-echo/logs:/app/logs

  # Authentication Service
  auth:
    build: ./vibrox-auth
    ports:
      - "8000:8000"
    environment:
      - DB_HOST=db
      - DB_USER=postgres
      - DB_PASSWORD=server
      - DB_NAME=postgres
      - LOGGER_HOST=logger:9000
    depends_on:
      - db
      - logger
    networks:
      - app-network

  # Main Application Service
  app:
    build: ./vibrox-core
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
      - DB_USER=postgres
      - DB_PASSWORD=server
      - DB_NAME=postgres
      - AUTH_HOST=auth:8000
      - LOGGER_HOST=logger:9000
    depends_on:
      - db
      - logger
      - auth
    networks:
      - app-network

networks:
  app-network:

volumes:
  vibrox-vol:
```

## Service Details

### Service Endpoints

| Service         | Port         | Description            | Health Check        |
| --------------- | ------------ | ---------------------- | ------------------- |
| **vibrox-core** | 8080         | User management API    | `GET /health`       |
| **vibrox-auth** | 8000         | Authentication service | gRPC health check   |
| **vibrox-echo** | 9000         | Logging service        | gRPC health check   |
| **vibrox-dns**  | 2053 UDP/TCP | Forwarding DNS         | `GET :8053/healthz` |
| **PostgreSQL**  | 5432         | Database               | `pg_isready`        |

### Environment Variables

#### Database Configuration

```bash
POSTGRES_USER=postgres
POSTGRES_PASSWORD=server
POSTGRES_DB=postgres
```

#### Service-Configuration

```bash
# vibrox-core
DB_HOST=db
DB_USER=postgres
DB_PASSWORD=server
DB_NAME=postgres
AUTH_HOST=auth:8000
LOGGER_HOST=logger:9000

# vibrox-auth
DB_HOST=db
DB_USER=postgres
DB_PASSWORD=server
DB_NAME=postgres
LOGGER_HOST=logger:9000

# vibrox-dns
DNS_UPSTREAM=1.1.1.1:53
DNS_TIMEOUT=3s
```

The DNS ports are bound to `127.0.0.1` so development does not expose an open
resolver. Test it with `dig @127.0.0.1 -p 2053 example.com`.

Every stack container uses `172.30.0.53` (`vibrox-dns`) as its default
nameserver. The address is reserved through the Compose network's IPAM config.
By default, `vibrox-dns` forwards to Docker's embedded resolver at
`127.0.0.11:53`; this retains both public DNS and Compose service discovery.

## Development Workflow

### Starting Development

```bash
# Start all services
docker-compose up -d

# View logs for specific service
docker-compose logs -f app

# Rebuild and restart specific service
docker-compose up -d --build app
```

### Making Code Changes

```bash
# After making changes to vibrox-core
docker-compose up -d --build app

# After making changes to vibrox-auth
docker-compose up -d --build auth

# After making changes to vibrox-echo
docker-compose up -d --build logger
```

### Database Operations

```bash
# Connect to database
docker-compose exec db psql -U postgres -d postgres

# Run database migrations
docker-compose exec app ./migrate

# Backup database
docker-compose exec db pg_dump -U postgres postgres > backup.sql

# Restore database
docker-compose exec -T db psql -U postgres postgres < backup.sql
```

## Volume Management

### Persistent Data

```bash
# List volumes
docker volume ls

# Inspect volume
docker volume inspect vibrox-stack_vibrox-vol

# Backup volume data
docker run --rm -v vibrox-stack_vibrox-vol:/data -v $(pwd):/backup alpine tar czf /backup/db-backup.tar.gz -C /data .

# Restore volume data
docker run --rm -v vibrox-stack_vibrox-vol:/data -v $(pwd):/backup alpine tar xzf /backup/db-backup.tar.gz -C /data
```

### Log Files

```bash
# View application logs
docker-compose logs -f app

# View database logs
docker-compose logs -f db

# Access log files directly
ls -la ./vibrox-echo/logs/
```

## Network Configuration

### Service Discovery

Services communicate using Docker Compose service names:

- `db` - PostgreSQL database
- `auth` - Authentication service
- `logger` - Logging service
- `app` - Main application

### Network Inspection

```bash
# List networks
docker network ls

# Inspect network
docker network inspect vibrox-stack_app-network

# Connect to network from host
docker run --rm --network vibrox-stack_app-network alpine ping db
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

# Stop conflicting services
sudo systemctl stop postgresql  # If PostgreSQL is running locally
```

#### 2. Service Startup Issues

```bash
# Check service status
docker-compose ps

# View detailed logs
docker-compose logs app
docker-compose logs auth
docker-compose logs logger
docker-compose logs db

# Restart specific service
docker-compose restart app
```

#### 3. Database Connection Issues

```bash
# Test database connectivity
docker-compose exec app ping db
docker-compose exec app nc -zv db 5432

# Check database logs
docker-compose logs db

# Reset database
docker-compose down -v
docker-compose up -d db
```

#### 4. Build Issues

```bash
# Clean build cache
docker-compose build --no-cache

# Remove all containers and images
docker-compose down --rmi all
docker system prune -a

# Rebuild from scratch
docker-compose up --build
```

### Performance Optimization

#### Resource Limits

```yaml
# Add to docker-compose.yml for production-like testing
services:
  app:
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "0.5"
        reservations:
          memory: 256M
          cpus: "0.25"
```

#### Health Checks

```yaml
# Add health checks to services
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

## Testing

### API Testing

```bash
# Test health endpoint
curl http://localhost:8080/health

# Test user endpoints
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@example.com"}'

# Test authentication
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"password"}'
```

### Load Testing

```bash
# Install Apache Bench
sudo apt install apache2-utils

# Run load test
ab -n 1000 -c 10 http://localhost:8080/health
```

## Monitoring

### Service Monitoring

```bash
# Monitor resource usage
docker stats

# Monitor logs in real-time
docker-compose logs -f --tail=100

# Check service health
docker-compose ps
```

### Database Monitoring

```bash
# Connect to database
docker-compose exec db psql -U postgres -d postgres

# Check active connections
SELECT * FROM pg_stat_activity;

# Check database size
SELECT pg_size_pretty(pg_database_size('postgres'));
```

## Cleanup

### Development Cleanup

```bash
# Stop all services
docker-compose down

# Remove volumes (WARNING: This deletes all data)
docker-compose down -v

# Remove all containers and networks
docker-compose down --remove-orphans

# Clean up Docker system
docker system prune -f
```

### Complete Reset

```bash
# Stop and remove everything
docker-compose down -v --rmi all

# Remove all Docker resources
docker system prune -a -f

# Rebuild from scratch
docker-compose up --build
```

## Best Practices

### Development-Workflow

1. **Always use `--build` flag** when making code changes
2. **Check logs first** when troubleshooting issues
3. **Use health checks** to ensure services are ready
4. **Backup data regularly** before major changes
5. **Test in isolation** before running full stack

### Configuration Management

1. **Use environment files** for different configurations
2. **Never commit secrets** to version control
3. **Use Docker secrets** for sensitive data in production
4. **Version control** your docker-compose.yml file

### Performance Tips

1. **Use volume mounts** for development (not bind mounts)
2. **Limit resource usage** to prevent system overload
3. **Use multi-stage builds** for smaller images
4. **Optimize Dockerfile** for faster builds

---

_This guide should be updated when Docker Compose configuration changes or new services are added to the stack._
