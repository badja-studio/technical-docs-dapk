# Getting Started - Local Development Setup

Quick guide to set up DAPK system on your local machine for development.

---

## Prerequisites

- **Docker Desktop** installed and running
- **Git** for version control
- **Code Editor** (VS Code recommended)
- **Postman** or similar for API testing (optional)

---

## Quick Setup (5 Minutes)

### 1. Clone Repository

```bash
git clone https://github.com/your-org/dapk-system.git
cd dapk-system
```

### 2. Configure Environment

```bash
# Copy example environment file
cp .env.example .env

# Edit .env if needed (defaults work for local dev)
# Default password: changeme (change in production!)
```

### 3. Start All Services

```bash
cd docker
docker-compose up -d
```

This starts:
- PostgreSQL database (port 5432)
- 3 FastAPI backend instances
- Next.js frontend (port 3000)
- Nginx load balancer (ports 80, 443)
- Redis cache (optional, port 6379)

### 4. Wait for Services

```bash
# Watch logs until all services are healthy
docker-compose logs -f

# Or check status
docker-compose ps
```

All services should show "Up" and "healthy" status.

### 5. Initialize Database

Database will auto-initialize from `docker/postgresql/init.sql`.

To verify:
```bash
docker-compose exec postgres psql -U dapk_user -d dapk -c "\dt"
```

Should list 17+ tables.

### 6. Create Admin User

```bash
docker-compose exec backend-1 python -m app.scripts.create_admin \
  --email admin@dapk.test \
  --password admin123 \
  --full-name "Admin User"
```

---

## Access Applications

- **Frontend**: http://localhost:3000
- **API Docs**: http://localhost:8000/docs (Swagger UI)
- **API Base**: http://localhost:8000/v1
- **Database**: localhost:5432 (user: dapk_user, password: changeme, database: dapk)

---

## Test the API

### Using Swagger UI (Easiest)

1. Open http://localhost:8000/docs
2. Click "Authorize" button
3. Login first: Try out `POST /v1/auth/login`
   ```json
   {
     "email": "admin@dapk.test",
     "password": "admin123"
   }
   ```
4. Copy `access_token` from response
5. Click "Authorize" again, paste token
6. Now you can try other endpoints!

### Using cURL

```bash
# Login
TOKEN=$(curl -X POST http://localhost:8000/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@dapk.test","password":"admin123"}' \
  | jq -r '.access_token')

# Create a shipment
curl -X POST http://localhost:8000/v1/shipments \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "sender_name": "John Doe",
    "sender_phone": "+6281234567890",
    "sender_address": "Jl. Sudirman No. 123",
    "origin_warehouse_id": "uuid-from-db",
    "receiver_name": "Jane Smith",
    "receiver_phone": "+6289876543210",
    "receiver_address": "Jl. Gatot Subroto No. 456",
    "destination_warehouse_id": "uuid-from-db",
    "weight_kg": 5.5,
    "commodity": "Electronics",
    "service_type": "express"
  }'
```

---

## Development Workflow

### Making Code Changes

#### Backend (Python/FastAPI)

```bash
# Backend code is in backend/ directory
# Edit files in backend/app/

# Restart backend to see changes
docker-compose restart backend-1 backend-2 backend-3

# Or rebuild if dependencies changed
docker-compose up -d --build backend-1
```

#### Frontend (Next.js)

```bash
# Frontend code is in frontend/ directory
# Hot reload is enabled by default

# If needed, restart
docker-compose restart frontend
```

### Database Changes

```bash
# Connect to PostgreSQL
docker-compose exec postgres psql -U dapk_user -d dapk

# Run SQL queries
dapk=# SELECT * FROM users;
dapk=# \dt  -- List tables
dapk=# \d shipments  -- Describe table
dapk=# \q  -- Quit
```

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f backend-1

# Last 100 lines
docker-compose logs --tail=100 backend-1
```

---

## Common Tasks

### Reset Database

```bash
# Stop all services
docker-compose down

# Remove database volume
docker volume rm docker_postgres_data

# Start again (will reinitialize)
docker-compose up -d
```

### Run Database Migrations

```bash
# Create new migration
docker-compose exec backend-1 alembic revision -m "Add new table"

# Apply migrations
docker-compose exec backend-1 alembic upgrade head

# Rollback
docker-compose exec backend-1 alembic downgrade -1
```

### Run Tests

```bash
# Backend tests
docker-compose exec backend-1 pytest

# With coverage
docker-compose exec backend-1 pytest --cov=app tests/

# Frontend tests
docker-compose exec frontend npm test
```

---

## Troubleshooting

### Port Already in Use

```bash
# Check what's using the port
sudo lsof -i :5432  # or :3000, :8000, etc.

# Kill the process or change port in docker-compose.yml
```

### Services Won't Start

```bash
# Check Docker is running
docker ps

# Rebuild images
docker-compose build

# Start with verbose output
docker-compose up
```

### Database Connection Failed

```bash
# Check PostgreSQL logs
docker-compose logs postgres

# Verify credentials in .env match docker-compose.yml
# Default: dapk_user / changeme / dapk
```

### Backend Import Errors

```bash
# Rebuild backend with fresh dependencies
docker-compose build backend-1
docker-compose up -d backend-1
```

---

## IDE Setup

### VS Code

Install extensions:
- Python (ms-python.python)
- Pylance (ms-python.vscode-pylance)
- Docker (ms-azuretools.vscode-docker)
- Thunder Client (rangav.vscode-thunder-client) - API testing
- Mermaid Preview (for viewing diagrams)

### Database Client

- **DBeaver** (free, cross-platform)
- **TablePlus** (macOS)
- **pgAdmin** (web-based)

Connection details:
- Host: localhost
- Port: 5432
- Database: dapk
- User: dapk_user
- Password: changeme

---

## Next Steps

1. ✅ System running locally
2. [ ] Explore [API Documentation](http://localhost:8000/docs)
3. [ ] Review [Database Schema](../03-database/schema-definitions.sql)
4. [ ] Read [System Architecture](../01-architecture/system-architecture.md)
5. [ ] Study [Coding Standards](coding-standards.md) (to be created)
6. [ ] Try creating test data via API
7. [ ] Build a simple feature

---

## Additional Resources

- [API Specification](../04-api/openapi-spec.yaml) - Import to Postman
- [Database ERD](../03-database/entity-relationship-diagram.md) - Visual schema
- [Shipment Lifecycle](../05-application-flows/shipment-lifecycle.md) - How it works
- [Deployment Guide](../06-deployment/deployment-guide.md) - For production

---

## Need Help?

- Check [Troubleshooting](#troubleshooting) section above
- Review Docker logs: `docker-compose logs -f`
- Verify all services healthy: `docker-compose ps`
- Consult [Architecture Docs](../01-architecture/)

