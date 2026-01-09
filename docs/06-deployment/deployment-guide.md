# Deployment Guide - DAPK Logistics System

Step-by-step guide to deploy DAPK system using Docker and Docker Compose.

---

## Prerequisites

### Required Software
- **Docker**: Version 24.0+ ([Install Docker](https://docs.docker.com/get-docker/))
- **Docker Compose**: Version 2.0+ (included with Docker Desktop)
- **Git**: For cloning repository
- **Text Editor**: For editing configuration files

### System Requirements

**Minimum (Development)**:
- CPU: 4 cores
- RAM: 8 GB
- Storage: 50 GB available
- Network: Stable internet connection

**Recommended (Production)**:
- CPU: 8+ cores
- RAM: 16+ GB
- Storage: 200+ GB SSD
- Network: High-bandwidth, low-latency

---

## Quick Start (Development)

### 1. Clone Repository

```bash
git clone https://github.com/your-org/dapk-system.git
cd dapk-system
```

### 2. Create Environment File

```bash
cp .env.example .env
```

Edit `.env` with your settings:

```env
# Database
DB_PASSWORD=your_secure_password_here

# JWT Authentication
JWT_SECRET=your-secret-key-at-least-32-characters-long

# Environment
ENVIRONMENT=development
DEBUG=true

# Frontend
NEXT_PUBLIC_API_URL=http://localhost:8000/v1

# Redis (optional)
REDIS_PASSWORD=your_redis_password
```

### 3. Start Services

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Check status
docker-compose ps
```

### 4. Initialize Database

```bash
# Database will be initialized automatically from init.sql
# Or run migrations manually:
docker-compose exec backend-1 alembic upgrade head
```

### 5. Access Applications

- **Frontend**: http://localhost:3000
- **API Documentation**: http://localhost:8000/docs
- **API**: http://localhost:8000/v1
- **PostgreSQL**: localhost:5432

### 6. Create Admin User

```bash
docker-compose exec backend-1 python -m app.scripts.create_admin \
  --email admin@dapk.com \
  --password SecurePassword123 \
  --full-name "System Administrator"
```

---

## Production Deployment

### Pre-Deployment Checklist

- [ ] Server provisioned (cloud or on-premise)
- [ ] Domain names configured (api.dapk.com, dapk.com)
- [ ] SSL certificates obtained (Let's Encrypt or commercial)
- [ ] Firewall configured (ports 80, 443 open)
- [ ] Database backup strategy in place
- [ ] Monitoring tools ready
- [ ] Code deployed to production branch

---

### Step 1: Server Setup

#### Update System

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git ufw
```

#### Configure Firewall

```bash
# Enable firewall
sudo ufw enable

# Allow SSH
sudo ufw allow 22/tcp

# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Check status
sudo ufw status
```

#### Install Docker

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Enable Docker to start on boot
sudo systemctl enable docker
sudo systemctl start docker

# Verify installation
docker --version
docker-compose --version
```

---

### Step 2: Clone and Configure

```bash
# Create application directory
sudo mkdir -p /opt/dapk
sudo chown $USER:$USER /opt/dapk
cd /opt/dapk

# Clone repository
git clone https://github.com/your-org/dapk-system.git .
git checkout production  # Or specific version tag

# Create environment file
cp .env.example .env
nano .env  # Edit with production values
```

#### Production Environment Variables

```env
# Database
DB_PASSWORD=<generate-strong-password-32-chars>
DATABASE_URL=postgresql+asyncpg://dapk_user:${DB_PASSWORD}@postgres:5432/dapk

# JWT (Generate with: openssl rand -hex 32)
JWT_SECRET=<generate-with-openssl-rand-hex-32>
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=1440

# Application
ENVIRONMENT=production
DEBUG=false

# CORS
ALLOWED_ORIGINS=https://dapk.com,https://www.dapk.com,https://api.dapk.com

# File Upload
UPLOAD_DIR=/app/uploads
MAX_UPLOAD_SIZE_MB=10

# Frontend
NEXT_PUBLIC_API_URL=https://api.dapk.com/v1
NODE_ENV=production

# Redis
REDIS_PASSWORD=<generate-strong-password>

# Email (for notifications, optional)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=noreply@dapk.com
SMTP_PASSWORD=<your-email-password>
SMTP_FROM=DAPK System <noreply@dapk.com>
```

**Security Notes**:
- Never commit `.env` to version control
- Use strong, randomly generated passwords
- Store secrets in environment variables or secret manager
- Rotate secrets regularly

---

### Step 3: SSL Certificates

#### Option A: Let's Encrypt (Free, Automated)

```bash
# Install Certbot
sudo apt install -y certbot

# Stop Nginx temporarily
docker-compose stop nginx

# Obtain certificates
sudo certbot certonly --standalone \
  -d api.dapk.com \
  -d dapk.com \
  -d www.dapk.com \
  --email admin@dapk.com \
  --agree-tos \
  --non-interactive

# Certificates will be in /etc/letsencrypt/live/

# Link certificates to docker volume
sudo mkdir -p docker/nginx/ssl
sudo ln -s /etc/letsencrypt/live/api.dapk.com docker/nginx/ssl/api
sudo ln -s /etc/letsencrypt/live/dapk.com docker/nginx/ssl/frontend

# Auto-renewal (cron job)
sudo crontab -e
# Add: 0 0 1 * * certbot renew --quiet && docker-compose restart nginx
```

#### Option B: Commercial SSL Certificate

1. Purchase certificate from CA
2. Generate CSR and private key
3. Receive certificate files (.crt, .key, .ca-bundle)
4. Place in `docker/nginx/ssl/`
5. Update nginx configuration with file paths

---

### Step 4: Database Preparation

#### Copy Schema to Init Script

```bash
cp docs/03-database/schema-definitions.sql docker/postgresql/init.sql
```

#### (Optional) Import Existing Data

```bash
# If migrating from existing system:
docker-compose up -d postgres
docker-compose exec postgres psql -U dapk_user -d dapk < backup.sql
```

---

### Step 5: Build and Start Services

#### Production Docker Compose

Create `docker-compose.prod.yml`:

```yaml
version: '3.8'

# Override for production
services:
  backend-1:
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
  
  backend-2:
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
  
  backend-3:
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
  
  nginx:
    restart: always
  
  frontend:
    restart: always
  
  postgres:
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "5"
```

#### Build Images

```bash
cd docker
docker-compose -f docker-compose.yml -f docker-compose.prod.yml build

# Or build individually
docker-compose build backend-1
docker-compose build frontend
docker-compose build nginx
```

#### Start Services

```bash
# Start in detached mode
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# View logs
docker-compose logs -f

# Check all services are running
docker-compose ps
```

Expected output:
```
NAME                COMMAND                  SERVICE      STATUS    PORTS
dapk-backend-1      "uvicorn app.main:..."   backend-1    Up        8000/tcp
dapk-backend-2      "uvicorn app.main:..."   backend-2    Up        8000/tcp
dapk-backend-3      "uvicorn app.main:..."   backend-3    Up        8000/tcp
dapk-frontend       "node server.js"         frontend     Up        3000/tcp
dapk-nginx          "nginx -g 'daemon ..."   nginx        Up        0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp
dapk-postgres       "docker-entrypoint..."   postgres     Up        0.0.0.0:5432->5432/tcp
```

---

### Step 6: Database Migrations

```bash
# Run initial migration
docker-compose exec backend-1 alembic upgrade head

# Check current version
docker-compose exec backend-1 alembic current
```

---

### Step 7: Create Admin User

```bash
docker-compose exec backend-1 python -m app.scripts.create_admin \
  --email admin@dapk.com \
  --password <secure-password> \
  --full-name "System Administrator" \
  --role admin
```

---

### Step 8: Verify Deployment

#### Health Checks

```bash
# Check API health
curl https://api.dapk.com/health

# Should return:
# {"status":"healthy","database":"connected","timestamp":"2026-01-09T..."}

# Check frontend
curl https://dapk.com

# Should return HTML content
```

#### Test API Authentication

```bash
# Login
curl -X POST https://api.dapk.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@dapk.com","password":"<password>"}'

# Should return JWT token
```

#### Access Applications

- **Public Website**: https://dapk.com
- **API Documentation**: https://api.dapk.com/docs
- **Admin Panel**: https://dapk.com/admin (if implemented)

---

### Step 9: Configure Monitoring

#### System Monitoring

```bash
# Install monitoring tools (optional)
docker-compose -f docker-compose.yml -f docker-compose.monitoring.yml up -d

# Includes: Prometheus, Grafana, Node Exporter
```

#### Log Monitoring

```bash
# View real-time logs
docker-compose logs -f backend-1

# View all backend logs
docker-compose logs backend-1 backend-2 backend-3

# View nginx access logs
docker-compose exec nginx tail -f /var/log/nginx/access.log

# View nginx error logs
docker-compose exec nginx tail -f /var/log/nginx/error.log
```

---

### Step 10: Backup Configuration

#### Database Backup Script

Create `/opt/dapk/scripts/backup-database.sh`:

```bash
#!/bin/bash
BACKUP_DIR="/opt/dapk/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/dapk_backup_$TIMESTAMP.sql"

mkdir -p $BACKUP_DIR

docker-compose exec -T postgres pg_dump -U dapk_user dapk > $BACKUP_FILE

gzip $BACKUP_FILE

# Keep only last 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_FILE.gz"
```

#### Schedule Backups (Cron)

```bash
chmod +x /opt/dapk/scripts/backup-database.sh

# Add to crontab
crontab -e

# Daily backup at 2 AM
0 2 * * * /opt/dapk/scripts/backup-database.sh >> /var/log/dapk-backup.log 2>&1
```

---

## Scaling Backend Instances

### Add More Backend Instances

#### Edit docker-compose.yml

```yaml
# Add backend-4
backend-4:
  build:
    context: ../backend
    dockerfile: ../docker/backend/Dockerfile
  container_name: dapk-backend-4
  environment:
    INSTANCE_ID: backend-4
    # ... same as other backends
  depends_on:
    postgres:
      condition: service_healthy
  networks:
    - dapk-network
  volumes:
    - backend_uploads:/app/uploads
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
    interval: 30s
  restart: unless-stopped
```

#### Update Nginx Configuration

Edit `docker/nginx/conf.d/backend.conf`:

```nginx
upstream backend_api {
    least_conn;
    
    server backend-1:8000 max_fails=3 fail_timeout=30s;
    server backend-2:8000 max_fails=3 fail_timeout=30s;
    server backend-3:8000 max_fails=3 fail_timeout=30s;
    server backend-4:8000 max_fails=3 fail_timeout=30s;  # NEW
    
    keepalive 32;
}
```

#### Deploy New Instance

```bash
docker-compose up -d backend-4
docker-compose restart nginx
```

---

## Updating the System

### Rolling Update (Zero Downtime)

```bash
# Pull latest code
cd /opt/dapk
git pull origin production

# Rebuild images
docker-compose build

# Update one backend at a time
docker-compose up -d --no-deps backend-1
sleep 30  # Wait for health check
docker-compose up -d --no-deps backend-2
sleep 30
docker-compose up -d --no-deps backend-3

# Update frontend
docker-compose up -d --no-deps frontend

# Update nginx (brief downtime)
docker-compose up -d --no-deps nginx
```

### Database Migrations

```bash
# Run migrations before updating backends
docker-compose exec backend-1 alembic upgrade head

# Then proceed with rolling update
```

---

## Troubleshooting

### Service Won't Start

```bash
# Check logs
docker-compose logs <service-name>

# Check container status
docker-compose ps

# Restart service
docker-compose restart <service-name>

# Rebuild and restart
docker-compose up -d --build <service-name>
```

### Database Connection Errors

```bash
# Check PostgreSQL is running
docker-compose ps postgres

# Check database logs
docker-compose logs postgres

# Test connection from backend
docker-compose exec backend-1 python -c \
  "from app.core.database import engine; print(engine.url)"

# Connect to PostgreSQL directly
docker-compose exec postgres psql -U dapk_user -d dapk
```

### High CPU/Memory Usage

```bash
# Check resource usage
docker stats

# Restart services
docker-compose restart

# Adjust Docker resource limits (docker-compose.yml)
services:
  backend-1:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
```

### SSL Certificate Issues

```bash
# Renew Let's Encrypt certificate
sudo certbot renew

# Restart nginx
docker-compose restart nginx

# Check certificate expiry
openssl x509 -in /etc/letsencrypt/live/api.dapk.com/fullchain.pem -noout -dates
```

---

## Maintenance

### Regular Tasks

**Daily**:
- Check application logs for errors
- Monitor disk space usage
- Verify backups completed successfully

**Weekly**:
- Review system metrics (CPU, memory, disk I/O)
- Check database performance (slow queries)
- Update security patches

**Monthly**:
- Test backup restoration
- Review and archive old logs
- Update dependencies (security updates)
- Rotate JWT secrets (if required)

---

## Related Documentation

- [Docker Setup Details](docker-setup.md)
- [Nginx Configuration](nginx-configuration.md)
- [Scaling Guide](scaling-guide.md)
- [Environment Variables Reference](environment-variables.md)
- [Monitoring Setup](monitoring.md)
