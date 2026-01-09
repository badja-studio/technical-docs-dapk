# Containerization - Docker & Docker Compose

Complete guide to Docker containerization for DAPK system.

---

## Docker Overview

**Official Site**: https://www.docker.com
**Docker Hub**: https://hub.docker.com
**License**: Apache License 2.0

Docker is a platform for developing, shipping, and running applications in containers - lightweight, portable, and self-sufficient units.

---

## Why Docker for DAPK?

### Consistency
✅ **Same Environment Everywhere**: Dev, staging, production identical
✅ **"Works on My Machine" Eliminated**: Bundled dependencies
✅ **Reproducible Builds**: Dockerfile as code

### Scalability
✅ **Easy Horizontal Scaling**: Add more containers
✅ **Resource Limits**: Control CPU/memory per container
✅ **Load Balancing**: Nginx distributes across instances

### Deployment
✅ **Single Command Deploy**: `docker-compose up -d`
✅ **Fast Rollback**: Revert to previous image
✅ **Zero Downtime Updates**: Rolling updates

### Isolation
✅ **Service Independence**: Containers don't interfere
✅ **Security Boundaries**: Process isolation
✅ **Easy Debugging**: Container logs separate

---

## DAPK Docker Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Host Machine                            │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Docker Network (bridge)                   │  │
│  │                 172.25.0.0/16                          │  │
│  ├───────────────────────────────────────────────────────┤  │
│  │                                                         │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐            │  │
│  │  │Backend-1 │  │Backend-2 │  │Backend-3 │            │  │
│  │  │FastAPI   │  │FastAPI   │  │FastAPI   │            │  │
│  │  │:8000     │  │:8000     │  │:8000     │            │  │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘            │  │
│  │       │             │             │                    │  │
│  │       └─────────────┴─────────────┘                   │  │
│  │                     │                                  │  │
│  │                ┌────▼──────┐                          │  │
│  │                │PostgreSQL │                          │  │
│  │                │:5432      │                          │  │
│  │                └───────────┘                          │  │
│  │                                                         │  │
│  │  ┌──────────┐       ┌──────────┐                     │  │
│  │  │Frontend  │       │  Nginx   │                     │  │
│  │  │Next.js   │◄──────┤Load      │                     │  │
│  │  │:3000     │       │Balancer  │                     │  │
│  │  └──────────┘       │:80,:443  │                     │  │
│  │                     └──────────┘                      │  │
│  │                                                         │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                               │
│  Docker Volumes:                                             │
│  ├── postgres_data (Database files)                         │
│  ├── backend_uploads (Shared files)                         │
│  ├── nginx_logs (Access & error logs)                       │
│  └── redis_data (Cache - optional)                          │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Docker Compose Configuration

Complete `docker-compose.yml` is at `docker/docker-compose.yml`. See [Docker Setup](../06-deployment/docker-setup.md) for full file.

### Key Concepts

#### Services
```yaml
services:
  postgres:     # Database
  backend-1:    # API instance 1
  backend-2:    # API instance 2
  backend-3:    # API instance 3
  nginx:        # Load balancer
  frontend:     # Next.js app
  redis:        # Cache (optional)
```

#### Networks
```yaml
networks:
  dapk-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.25.0.0/16
```

#### Volumes
```yaml
volumes:
  postgres_data:       # Persistent database
  backend_uploads:     # Shared file storage
  nginx_logs:          # Nginx logs
  redis_data:          # Redis persistence
```

---

## Dockerfile Examples

### Backend (FastAPI)

```dockerfile
# docker/backend/Dockerfile

FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create uploads directory
RUN mkdir -p /app/uploads

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# Run application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "1"]
```

### Frontend (Next.js)

```dockerfile
# docker/frontend/Dockerfile

# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Stage 2: Builder
FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build application
ENV NEXT_TELEMETRY_DISABLED=1
RUN npm run build

# Stage 3: Runner
FROM node:20-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

# Copy necessary files
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000

CMD ["node", "server.js"]
```

### Nginx

```dockerfile
# docker/nginx/Dockerfile

FROM nginx:alpine

# Copy configuration
COPY nginx.conf /etc/nginx/nginx.conf
COPY conf.d/ /etc/nginx/conf.d/

# Create log directory
RUN mkdir -p /var/log/nginx

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

---

## Docker Commands

### Basic Operations

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Check status
docker-compose ps

# Stop all services
docker-compose down

# Stop and remove volumes (⚠️ DESTROYS DATA)
docker-compose down -v
```

### Service Management

```bash
# Restart specific service
docker-compose restart backend-1

# Stop specific service
docker-compose stop frontend

# Start specific service
docker-compose start frontend

# Rebuild and restart
docker-compose up -d --build backend-1
```

### Scaling

```bash
# Scale backend to 5 instances
docker-compose up -d --scale backend=5

# Or add manually in docker-compose.yml
```

### Logs & Debugging

```bash
# View logs for all services
docker-compose logs

# Follow logs for specific service
docker-compose logs -f backend-1

# Last 100 lines
docker-compose logs --tail=100 postgres

# View logs from last hour
docker-compose logs --since 1h

# Execute command in running container
docker-compose exec backend-1 bash
docker-compose exec postgres psql -U dapk_user -d dapk
```

---

## Environment Variables

### .env File

```bash
# Database
DB_PASSWORD=secure_password_change_in_production
DATABASE_URL=postgresql+asyncpg://dapk_user:${DB_PASSWORD}@postgres:5432/dapk

# JWT
JWT_SECRET=generate_with_openssl_rand_hex_32
JWT_ALGORITHM=HS256

# Application
ENVIRONMENT=production
DEBUG=false

# Frontend
NEXT_PUBLIC_API_URL=https://api.dapk.com/v1
```

### Docker Compose Variable Substitution

```yaml
services:
  backend-1:
    environment:
      DATABASE_URL: ${DATABASE_URL}
      JWT_SECRET: ${JWT_SECRET}
      ENVIRONMENT: ${ENVIRONMENT:-development}  # Default value
```

---

## Health Checks

### Backend Health Check

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
  interval: 30s      # Check every 30 seconds
  timeout: 10s       # Timeout after 10 seconds
  retries: 3         # Try 3 times before marking unhealthy
  start_period: 40s  # Grace period on startup
```

### PostgreSQL Health Check

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U dapk_user -d dapk"]
  interval: 10s
  timeout: 5s
  retries: 5
```

### Nginx Health Check

```yaml
healthcheck:
  test: ["CMD", "nginx", "-t"]  # Test configuration
  interval: 30s
  timeout: 10s
  retries: 3
```

---

## Volume Management

### Inspect Volume

```bash
# List volumes
docker volume ls

# Inspect specific volume
docker volume inspect docker_postgres_data

# Find volume location
docker volume inspect docker_postgres_data --format '{{ .Mountpoint }}'
```

### Backup Volume

```bash
# Backup PostgreSQL data
docker run --rm \
  -v docker_postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres-backup.tar.gz /data
```

### Restore Volume

```bash
# Restore PostgreSQL data
docker run --rm \
  -v docker_postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/postgres-backup.tar.gz -C /
```

---

## Resource Limits

### CPU & Memory Limits

```yaml
services:
  backend-1:
    deploy:
      resources:
        limits:
          cpus: '1.0'      # Maximum 1 CPU
          memory: 1G       # Maximum 1GB RAM
        reservations:
          cpus: '0.5'      # Minimum 0.5 CPU
          memory: 512M     # Minimum 512MB RAM
```

### Check Resource Usage

```bash
# Real-time stats
docker stats

# Specific container
docker stats dapk-backend-1
```

---

## Networking

### Inspect Network

```bash
# List networks
docker network ls

# Inspect network
docker network inspect docker_dapk-network

# See connected containers
docker network inspect docker_dapk-network --format '{{ .Containers }}'
```

### Container Communication

```yaml
# Containers on same network can communicate by service name
services:
  backend-1:
    environment:
      DATABASE_URL: postgresql://dapk_user:pass@postgres:5432/dapk
                                              # ^^^^^^^^
                                              # Service name as hostname
```

---

## Production Best Practices

### Security

```yaml
# Don't expose unnecessary ports
services:
  postgres:
    ports:
      - "127.0.0.1:5432:5432"  # Only localhost
    # Or don't expose at all, use Docker network
```

### Restart Policies

```yaml
services:
  backend-1:
    restart: unless-stopped  # Always restart except manual stop
  
  postgres:
    restart: always          # Always restart
```

### Logging

```yaml
services:
  backend-1:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"      # Max log file size
        max-file: "3"        # Keep 3 log files
```

### Read-Only Filesystem

```yaml
services:
  nginx:
    read_only: true          # Read-only root filesystem
    tmpfs:
      - /var/cache/nginx     # Writable temp directory
      - /var/run
```

---

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker-compose logs service-name

# Check if port is already in use
sudo lsof -i :5432

# Check container details
docker inspect dapk-backend-1
```

### Network Issues

```bash
# Recreate network
docker-compose down
docker network prune
docker-compose up -d

# Check DNS resolution
docker-compose exec backend-1 ping postgres
```

### Permission Issues

```bash
# Check volume permissions
docker-compose exec backend-1 ls -la /app/uploads

# Fix ownership
docker-compose exec backend-1 chown -R app:app /app/uploads
```

### Out of Disk Space

```bash
# Clean up unused resources
docker system prune -a

# Remove unused volumes
docker volume prune

# Check disk usage
docker system df
```

---

## Docker Compose Profiles

### Optional Services

```yaml
services:
  redis:
    profiles: ["with-redis"]  # Only start if profile specified
```

```bash
# Start with Redis
docker-compose --profile with-redis up -d

# Start without Redis
docker-compose up -d
```

---

## Multi-Stage Builds

### Benefits
- **Smaller Images**: Only runtime dependencies
- **Security**: Build tools not in production
- **Faster**: Less to download and deploy

### Example

```dockerfile
# Build stage
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
CMD ["node", "server.js"]
```

---

## Development vs Production

### Development

```yaml
# docker-compose.dev.yml
services:
  backend-1:
    volumes:
      - ../backend:/app      # Mount source code
    command: uvicorn app.main:app --reload  # Auto-reload
    environment:
      DEBUG: "true"
```

```bash
# Run development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
```

### Production

```yaml
# docker-compose.prod.yml
services:
  backend-1:
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
```

```bash
# Run production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

## Monitoring

### Container Health

```bash
# Check health status
docker-compose ps

# Inspect health check results
docker inspect --format='{{json .State.Health}}' dapk-backend-1 | jq
```

### Resource Usage

```bash
# Monitor all containers
docker stats

# Export to file
docker stats --no-stream > stats.log
```

---

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Build and Push Docker Images

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build images
        run: |
          docker-compose build
      
      - name: Push to registry
        run: |
          docker-compose push
      
      - name: Deploy
        run: |
          ssh user@server "cd /opt/dapk && docker-compose pull && docker-compose up -d"
```

---

## Related Documentation

- [Docker Compose File](../../docker/docker-compose.yml)
- [Deployment Guide](../06-deployment/deployment-guide.md)
- [Nginx Configuration](../06-deployment/nginx-configuration.md)
- [System Architecture](../01-architecture/system-architecture.md)

