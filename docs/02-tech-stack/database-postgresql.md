# Database Technology - PostgreSQL

Complete guide to PostgreSQL database implementation for DAPK system.

---

## PostgreSQL Overview

**Official Site**: https://www.postgresql.org
**Version**: 15+ (Latest stable)
**License**: PostgreSQL License (permissive, BSD-like)

PostgreSQL is a powerful, open-source object-relational database system with over 30 years of active development.

---

## Why PostgreSQL for DAPK?

### Reliability & Data Integrity
✅ **ACID Compliant**: Guaranteed data integrity
✅ **Proven Track Record**: 30+ years in production
✅ **Zero Data Loss**: With proper configuration
✅ **Referential Integrity**: Foreign keys enforced

### Advanced Features
✅ **UUID Support**: Native gen_random_uuid()
✅ **JSONB**: Store flexible data efficiently
✅ **Full-Text Search**: Built-in text search
✅ **Array Types**: Store arrays natively
✅ **Partitioning**: For large tables
✅ **Extensions**: Extensible architecture

### Performance
✅ **Query Optimizer**: Advanced query planner
✅ **Indexing**: Multiple index types (B-tree, Hash, GiST, GIN)
✅ **Parallel Query**: Multi-core query execution
✅ **Connection Pooling**: Efficient connections

### Scalability
✅ **Read Replicas**: Scale read operations
✅ **Partitioning**: Horizontal data splitting
✅ **Foreign Data Wrappers**: Connect to other databases
✅ **Logical Replication**: Real-time data sync

---

## Installation

### Docker (Recommended)

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: dapk
      POSTGRES_USER: dapk_user
      POSTGRES_PASSWORD: secure_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
```

### Ubuntu/Debian

```bash
# Add PostgreSQL repository
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

# Install PostgreSQL 15
sudo apt update
sudo apt install -y postgresql-15 postgresql-contrib-15

# Start service
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

---

## Configuration

### postgresql.conf (Important Settings)

```ini
# Connection Settings
max_connections = 100                    # Maximum concurrent connections
shared_buffers = 256MB                   # RAM for caching (25% of system RAM)
effective_cache_size = 1GB               # OS cache size estimate

# Write Ahead Log (WAL)
wal_level = replica                      # For replication
max_wal_size = 1GB
min_wal_size = 80MB

# Query Planning
random_page_cost = 1.1                   # For SSD (default 4.0)
effective_io_concurrency = 200           # For SSD

# Logging
log_min_duration_statement = 1000        # Log slow queries (>1s)
log_line_prefix = '%t [%p]: [%l-1] '
log_timezone = 'UTC'

# Autovacuum
autovacuum = on
autovacuum_max_workers = 3
```

### pg_hba.conf (Authentication)

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     peer
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
host    dapk            dapk_user       172.25.0.0/16           scram-sha-256  # Docker network
```

---

## Database Setup

### Create Database & User

```sql
-- Connect as postgres superuser
sudo -u postgres psql

-- Create user
CREATE USER dapk_user WITH PASSWORD 'secure_password_here';

-- Create database
CREATE DATABASE dapk OWNER dapk_user;

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE dapk TO dapk_user;

-- Connect to database
\c dapk

-- Enable extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";     -- UUID generation
CREATE EXTENSION IF NOT EXISTS "pg_trgm";       -- Text search optimization

-- Grant schema permissions
GRANT ALL ON SCHEMA public TO dapk_user;
GRANT ALL ON ALL TABLES IN SCHEMA public TO dapk_user;
GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO dapk_user;
```

---

## Schema Implementation

### Import Schema

```bash
# Import schema from documentation
psql -U dapk_user -d dapk -f docs/03-database/schema-definitions.sql

# Or via Docker
cat docs/03-database/schema-definitions.sql | \
  docker-compose exec -T postgres psql -U dapk_user -d dapk
```

### Verify Tables

```sql
-- List all tables
\dt

-- Describe specific table
\d shipments

-- Check table sizes
SELECT 
    table_name,
    pg_size_pretty(pg_total_relation_size(quote_ident(table_name))) as size
FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY pg_total_relation_size(quote_ident(table_name)) DESC;
```

---

## Index Strategy

### Automatic Indexes
- Primary keys (UUID): B-tree index automatically created
- Unique constraints: B-tree index automatically created

### Manual Indexes (Already in Schema)

```sql
-- Foreign keys (for joins)
CREATE INDEX idx_shipments_origin ON shipments(origin_warehouse_id);
CREATE INDEX idx_shipments_destination ON shipments(destination_warehouse_id);

-- Status queries (with soft delete)
CREATE INDEX idx_shipments_status ON shipments(status, deleted_at);

-- Date range queries
CREATE INDEX idx_shipments_dates ON shipments(created_at, deleted_at);

-- Composite index for common filters
CREATE INDEX idx_shipments_warehouse_status 
ON shipments(origin_warehouse_id, status, deleted_at);

-- Text search (if needed)
CREATE INDEX idx_shipments_awb_trgm ON shipments USING gin(awb_number gin_trgm_ops);
```

### Index Monitoring

```sql
-- Check index usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Find unused indexes
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
AND indexname NOT LIKE '%_pkey';
```

---

## Performance Optimization

### Connection Pooling

**Application Side** (SQLAlchemy):
```python
engine = create_async_engine(
    DATABASE_URL,
    pool_size=20,           # Connections per instance
    max_overflow=10,        # Additional connections when busy
    pool_pre_ping=True,     # Verify connection before use
    pool_recycle=3600       # Recycle connections every hour
)
```

**Database Side** (PgBouncer - Optional):
```ini
[databases]
dapk = host=localhost dbname=dapk

[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
```

### Query Optimization

```sql
-- Use EXPLAIN ANALYZE to understand query plans
EXPLAIN ANALYZE
SELECT * FROM shipments 
WHERE status = 'pending' 
AND deleted_at IS NULL
LIMIT 50;

-- Materialized view for expensive reports
CREATE MATERIALIZED VIEW daily_shipment_stats AS
SELECT 
    DATE(created_at) as date,
    origin_warehouse_id,
    COUNT(*) as total_shipments,
    SUM(total_amount) as total_amount
FROM shipments
WHERE deleted_at IS NULL
GROUP BY DATE(created_at), origin_warehouse_id;

-- Refresh periodically (e.g., cron job)
REFRESH MATERIALIZED VIEW daily_shipment_stats;
```

### Partitioning (For Large Tables)

```sql
-- Partition shipments table by year
CREATE TABLE shipments_2026 PARTITION OF shipments
    FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');

CREATE TABLE shipments_2027 PARTITION OF shipments
    FOR VALUES FROM ('2027-01-01') TO ('2028-01-01');

-- Automatic partition creation (future enhancement)
```

---

## Backup & Recovery

### Automated Backup Script

```bash
#!/bin/bash
# backup-database.sh

BACKUP_DIR="/opt/dapk/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/dapk_backup_$TIMESTAMP.sql"

mkdir -p $BACKUP_DIR

# Backup database
pg_dump -U dapk_user -h localhost dapk > $BACKUP_FILE

# Compress
gzip $BACKUP_FILE

# Keep only last 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_FILE.gz"
```

### Restore from Backup

```bash
# Decompress
gunzip dapk_backup_20260109_020000.sql.gz

# Restore
psql -U dapk_user -d dapk < dapk_backup_20260109_020000.sql
```

### Point-in-Time Recovery (PITR)

```ini
# postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /opt/dapk/wal_archive/%f'
```

---

## Monitoring

### Important Metrics

```sql
-- Database size
SELECT pg_size_pretty(pg_database_size('dapk'));

-- Table sizes
SELECT 
    relname as table_name,
    pg_size_pretty(pg_total_relation_size(relid)) as total_size,
    pg_size_pretty(pg_relation_size(relid)) as table_size,
    pg_size_pretty(pg_total_relation_size(relid) - pg_relation_size(relid)) as index_size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC;

-- Active connections
SELECT count(*) FROM pg_stat_activity;

-- Long running queries
SELECT 
    pid,
    now() - query_start as duration,
    query,
    state
FROM pg_stat_activity
WHERE state != 'idle'
AND query_start < now() - interval '5 minutes'
ORDER BY duration DESC;

-- Cache hit ratio (should be >90%)
SELECT 
    sum(heap_blks_read) as heap_read,
    sum(heap_blks_hit) as heap_hit,
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;
```

---

## Maintenance

### Regular Vacuum

```sql
-- Analyze tables (update statistics)
ANALYZE;

-- Vacuum (reclaim space)
VACUUM;

-- Vacuum full (requires exclusive lock, more thorough)
VACUUM FULL;

-- Automatic vacuum is enabled by default
-- Check autovacuum status
SELECT * FROM pg_stat_user_tables WHERE schemaname = 'public';
```

### Reindexing

```sql
-- Rebuild all indexes in database
REINDEX DATABASE dapk;

-- Rebuild specific table indexes
REINDEX TABLE shipments;

-- Rebuild concurrently (no downtime)
REINDEX INDEX CONCURRENTLY idx_shipments_status;
```

---

## Security Best Practices

### User Permissions

```sql
-- Create read-only user for reports
CREATE USER reports_user WITH PASSWORD 'read_only_pass';
GRANT CONNECT ON DATABASE dapk TO reports_user;
GRANT USAGE ON SCHEMA public TO reports_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO reports_user;

-- Revoke unnecessary permissions
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
```

### SSL/TLS Connection

```ini
# postgresql.conf
ssl = on
ssl_cert_file = '/path/to/server.crt'
ssl_key_file = '/path/to/server.key'
```

### Audit Logging

```ini
# postgresql.conf
log_statement = 'ddl'                    # Log all DDL
log_connections = on
log_disconnections = on
log_duration = on
```

---

## Common Operations

### Reset Sequence

```sql
-- If manual ID insertion breaks sequence
SELECT setval('users_id_seq', (SELECT MAX(id) FROM users));
```

### Find Duplicate Records

```sql
SELECT awb_number, COUNT(*)
FROM shipments
WHERE deleted_at IS NULL
GROUP BY awb_number
HAVING COUNT(*) > 1;
```

### Table Locking

```sql
-- Exclusive lock (blocks all other access)
LOCK TABLE shipments IN EXCLUSIVE MODE;

-- Share lock (allows reads, blocks writes)
LOCK TABLE shipments IN SHARE MODE;
```

---

## Extensions

### Useful Extensions

```sql
-- UUID generation
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Text search optimization
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- PostGIS (for geographic data - future)
CREATE EXTENSION IF NOT EXISTS postgis;

-- pg_stat_statements (query performance)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

---

## Related Documentation

- [Database Schema SQL](../03-database/schema-definitions.sql)
- [Entity Relationship Diagram](../03-database/entity-relationship-diagram.md)
- [Soft Delete Strategy](../03-database/soft-delete-strategy.md)
- [System Architecture](../01-architecture/system-architecture.md)

