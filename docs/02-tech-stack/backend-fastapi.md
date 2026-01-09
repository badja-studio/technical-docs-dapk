# Backend Technology - FastAPI (Python)

Complete guide to FastAPI backend implementation for DAPK system.

---

## FastAPI Overview

**Official Site**: https://fastapi.tiangolo.com
**GitHub**: https://github.com/tiangolo/fastapi (60,000+ stars)
**License**: MIT

FastAPI is a modern, fast (high-performance) web framework for building APIs with Python 3.11+ based on standard Python type hints.

---

## Why FastAPI for DAPK?

### Performance
- **One of the fastest Python frameworks** (on par with NodeJS and Go)
- Async/await support natively
- Can handle 10,000+ concurrent requests
- Perfect for high-throughput logistics operations

### Automatic API Documentation
- **Swagger UI** automatically generated at `/docs`
- **ReDoc** alternative at `/redoc`
- OpenAPI 3.0 standard
- No manual documentation needed

### Developer Experience
- Type hints for autocomplete & validation
- Minimal boilerplate code
- Excellent error messages
- Fast to develop, fast to run

### Production Ready
- Used by Microsoft, Uber, Netflix
- Battle-tested in production
- Security best practices built-in
- Excellent documentation

---

## Project Structure

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                     # FastAPI app initialization
│   │
│   ├── api/                        # API endpoints
│   │   ├── __init__.py
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── router.py           # Main router
│   │       ├── dependencies.py     # Shared dependencies
│   │       └── endpoints/
│   │           ├── __init__.py
│   │           ├── auth.py         # Authentication endpoints
│   │           ├── shipments.py    # Shipment CRUD
│   │           ├── manifests.py    # Manifest operations
│   │           ├── runsheets.py    # Runsheet management
│   │           ├── tracking.py     # Tracking endpoints
│   │           ├── reports.py      # Report generation
│   │           └── master_data.py  # Master data CRUD
│   │
│   ├── core/                       # Core configuration
│   │   ├── __init__.py
│   │   ├── config.py               # Settings from environment
│   │   ├── security.py             # JWT, password hashing
│   │   └── database.py             # DB connection, session
│   │
│   ├── models/                     # SQLAlchemy ORM models
│   │   ├── __init__.py
│   │   ├── base.py                 # Base model with timestamps
│   │   ├── user.py                 # User model
│   │   ├── shipment.py             # Shipment model
│   │   ├── manifest.py             # Manifest models
│   │   ├── runsheet.py             # Runsheet model
│   │   └── ...                     # Other models
│   │
│   ├── schemas/                    # Pydantic schemas
│   │   ├── __init__.py
│   │   ├── user.py                 # UserCreate, UserResponse
│   │   ├── shipment.py             # ShipmentCreate, ShipmentResponse
│   │   ├── manifest.py             # Manifest schemas
│   │   └── ...                     # Other schemas
│   │
│   ├── services/                   # Business logic layer
│   │   ├── __init__.py
│   │   ├── shipment_service.py     # Shipment business logic
│   │   ├── manifest_service.py     # Manifest logic
│   │   ├── tariff_service.py       # Tariff calculation
│   │   ├── awb_service.py          # AWB generation
│   │   └── pdf_service.py          # PDF generation
│   │
│   ├── repositories/               # Data access layer
│   │   ├── __init__.py
│   │   ├── base.py                 # Base repository
│   │   ├── shipment_repository.py  # Shipment data access
│   │   ├── user_repository.py      # User data access
│   │   └── ...                     # Other repositories
│   │
│   ├── utils/                      # Helper functions
│   │   ├── __init__.py
│   │   ├── validators.py           # Custom validators
│   │   ├── formatters.py           # Data formatters
│   │   └── generators.py           # ID generators, etc.
│   │
│   └── scripts/                    # Management scripts
│       ├── __init__.py
│       ├── create_admin.py         # Create admin user
│       └── seed_data.py            # Seed test data
│
├── tests/                          # Test suite
│   ├── __init__.py
│   ├── conftest.py                 # Pytest configuration
│   ├── test_api/
│   │   ├── test_auth.py
│   │   ├── test_shipments.py
│   │   └── ...
│   └── test_services/
│       ├── test_shipment_service.py
│       └── ...
│
├── alembic/                        # Database migrations
│   ├── versions/
│   ├── env.py
│   └── script.py.mako
│
├── alembic.ini                     # Alembic configuration
├── requirements.txt                # Production dependencies
├── requirements-dev.txt            # Development dependencies
├── .env.example                    # Environment variables template
├── pytest.ini                      # Pytest configuration
└── Dockerfile                      # Docker image definition
```

---

## Core Dependencies

```txt
# requirements.txt

# Web Framework
fastapi==0.104.0
uvicorn[standard]==0.24.0      # ASGI server with websocket support

# Database
sqlalchemy==2.0.23             # ORM with async support
asyncpg==0.29.0                # PostgreSQL async driver
alembic==1.12.0                # Database migrations

# Validation & Serialization
pydantic==2.5.0                # Data validation
pydantic-settings==2.1.0       # Settings management
email-validator==2.1.0         # Email validation

# Authentication & Security
python-jose[cryptography]==3.3.0  # JWT tokens
passlib[bcrypt]==1.7.4         # Password hashing
python-multipart==0.0.6        # Form data & file uploads

# Document Generation
reportlab==4.0.7               # PDF generation
pillow==10.1.0                 # Image processing

# Utilities
python-dateutil==2.8.2         # Date manipulation
pytz==2023.3                   # Timezone support

# Redis (Optional)
redis==5.0.1                   # Redis client
hiredis==2.2.3                 # Fast Redis parser
```

---

## Configuration Management

### app/core/config.py

```python
from pydantic_settings import BaseSettings
from typing import List

class Settings(BaseSettings):
    """Application settings loaded from environment variables"""
    
    # Application
    PROJECT_NAME: str = "DAPK API"
    VERSION: str = "1.0.0"
    API_V1_PREFIX: str = "/v1"
    DEBUG: bool = False
    ENVIRONMENT: str = "development"
    
    # Database
    DATABASE_URL: str
    DB_POOL_SIZE: int = 20
    DB_MAX_OVERFLOW: int = 10
    
    # JWT
    JWT_SECRET: str
    JWT_ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 1440  # 24 hours
    REFRESH_TOKEN_EXPIRE_DAYS: int = 30
    
    # CORS
    ALLOWED_ORIGINS: List[str] = ["http://localhost:3000"]
    
    # File Upload
    UPLOAD_DIR: str = "/app/uploads"
    MAX_UPLOAD_SIZE_MB: int = 10
    
    # Redis (Optional)
    REDIS_HOST: str = "redis"
    REDIS_PORT: int = 6379
    REDIS_PASSWORD: str = ""
    REDIS_DB: int = 0
    
    # Email (Optional)
    SMTP_HOST: str = ""
    SMTP_PORT: int = 587
    SMTP_USER: str = ""
    SMTP_PASSWORD: str = ""
    
    class Config:
        env_file = ".env"
        case_sensitive = True

# Global settings instance
settings = Settings()
```

---

## Database Connection

### app/core/database.py

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.ext.asyncio import async_sessionmaker
from sqlalchemy.orm import declarative_base
from app.core.config import settings

# Create async engine
engine = create_async_engine(
    settings.DATABASE_URL,
    pool_size=settings.DB_POOL_SIZE,
    max_overflow=settings.DB_MAX_OVERFLOW,
    pool_pre_ping=True,  # Verify connection before use
    pool_recycle=3600,   # Recycle connections every hour
    echo=settings.DEBUG  # Log SQL queries in debug mode
)

# Session factory
AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
    autocommit=False,
    autoflush=False
)

# Base class for models
Base = declarative_base()

# Dependency for getting DB session
async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        try:
            yield session
        finally:
            await session.close()
```

---

## Authentication & Security

### app/core/security.py

```python
from datetime import datetime, timedelta
from typing import Optional
from jose import jwt, JWTError
from passlib.context import CryptContext
from app.core.config import settings

# Password hashing
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    """Hash a password using bcrypt"""
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """Verify password against hash"""
    return pwd_context.verify(plain_password, hashed_password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    """Create JWT access token"""
    to_encode = data.copy()
    
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(
            minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES
        )
    
    to_encode.update({"exp": expire, "type": "access"})
    
    encoded_jwt = jwt.encode(
        to_encode,
        settings.JWT_SECRET,
        algorithm=settings.JWT_ALGORITHM
    )
    
    return encoded_jwt

def decode_access_token(token: str) -> dict:
    """Decode and verify JWT token"""
    try:
        payload = jwt.decode(
            token,
            settings.JWT_SECRET,
            algorithms=[settings.JWT_ALGORITHM]
        )
        return payload
    except JWTError:
        return None
```

---

## Base Model (with Soft Delete)

### app/models/base.py

```python
from sqlalchemy import Column, DateTime, func
from sqlalchemy.dialects.postgresql import UUID
from app.core.database import Base
import uuid

class BaseModel(Base):
    """Base model with common fields"""
    __abstract__ = True
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    created_at = Column(DateTime(timezone=True), nullable=False, server_default=func.now())
    updated_at = Column(DateTime(timezone=True), nullable=False, server_default=func.now(), onupdate=func.now())
    deleted_at = Column(DateTime(timezone=True), nullable=True)
    
    def soft_delete(self):
        """Mark record as deleted"""
        self.deleted_at = func.now()
    
    def restore(self):
        """Restore soft-deleted record"""
        self.deleted_at = None
```

---

## Repository Pattern

### app/repositories/base.py

```python
from typing import Generic, TypeVar, Type, Optional, List
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, and_
from app.models.base import BaseModel

ModelType = TypeVar("ModelType", bound=BaseModel)

class BaseRepository(Generic[ModelType]):
    """Base repository with common CRUD operations"""
    
    def __init__(self, model: Type[ModelType], db: AsyncSession):
        self.model = model
        self.db = db
    
    async def get(self, id: str) -> Optional[ModelType]:
        """Get single record by ID (active only)"""
        stmt = select(self.model).where(
            and_(
                self.model.id == id,
                self.model.deleted_at.is_(None)
            )
        )
        result = await self.db.execute(stmt)
        return result.scalar_one_or_none()
    
    async def get_all(self, skip: int = 0, limit: int = 100) -> List[ModelType]:
        """Get all active records with pagination"""
        stmt = select(self.model).where(
            self.model.deleted_at.is_(None)
        ).offset(skip).limit(limit)
        
        result = await self.db.execute(stmt)
        return result.scalars().all()
    
    async def create(self, obj_in: dict) -> ModelType:
        """Create new record"""
        db_obj = self.model(**obj_in)
        self.db.add(db_obj)
        await self.db.commit()
        await self.db.refresh(db_obj)
        return db_obj
    
    async def update(self, id: str, obj_in: dict) -> Optional[ModelType]:
        """Update existing record"""
        db_obj = await self.get(id)
        if not db_obj:
            return None
        
        for field, value in obj_in.items():
            setattr(db_obj, field, value)
        
        await self.db.commit()
        await self.db.refresh(db_obj)
        return db_obj
    
    async def soft_delete(self, id: str) -> bool:
        """Soft delete record"""
        db_obj = await self.get(id)
        if not db_obj:
            return False
        
        db_obj.soft_delete()
        await self.db.commit()
        return True
```

---

## API Endpoint Example

### app/api/v1/endpoints/shipments.py

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from typing import List

from app.core.database import get_db
from app.schemas.shipment import ShipmentCreate, ShipmentResponse
from app.services.shipment_service import ShipmentService
from app.api.v1.dependencies import get_current_user
from app.models.user import User

router = APIRouter(prefix="/shipments", tags=["Shipments"])

@router.post("/", response_model=ShipmentResponse, status_code=status.HTTP_201_CREATED)
async def create_shipment(
    shipment_in: ShipmentCreate,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """Create new shipment with automatic AWB generation and tariff calculation"""
    
    service = ShipmentService(db)
    shipment = await service.create_shipment(shipment_in, current_user.id)
    
    return shipment

@router.get("/", response_model=List[ShipmentResponse])
async def list_shipments(
    skip: int = 0,
    limit: int = 50,
    status: Optional[str] = None,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """List shipments with optional filters"""
    
    service = ShipmentService(db)
    shipments = await service.list_shipments(skip, limit, status)
    
    return shipments

@router.get("/{shipment_id}", response_model=ShipmentResponse)
async def get_shipment(
    shipment_id: str,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """Get single shipment by ID"""
    
    service = ShipmentService(db)
    shipment = await service.get_shipment(shipment_id)
    
    if not shipment:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Shipment not found"
        )
    
    return shipment
```

---

## Application Initialization

### app/main.py

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.core.config import settings
from app.api.v1.router import api_router

# Create FastAPI app
app = FastAPI(
    title=settings.PROJECT_NAME,
    version=settings.VERSION,
    openapi_url=f"{settings.API_V1_PREFIX}/openapi.json",
    docs_url="/docs",
    redoc_url="/redoc"
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include API router
app.include_router(api_router, prefix=settings.API_V1_PREFIX)

# Health check endpoint
@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "version": settings.VERSION,
        "environment": settings.ENVIRONMENT
    }

# Startup event
@app.on_event("startup")
async def startup_event():
    print(f"🚀 {settings.PROJECT_NAME} started!")
    print(f"📖 Docs: http://localhost:8000/docs")

# Shutdown event
@app.on_event("shutdown")
async def shutdown_event():
    print("👋 Shutting down...")
```

---

## Running the Application

### Development

```bash
# Install dependencies
pip install -r requirements.txt

# Run with auto-reload
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Production

```bash
# Run with Gunicorn + Uvicorn workers
gunicorn app.main:app \
  --workers 4 \
  --worker-class uvicorn.workers.UvicornWorker \
  --bind 0.0.0.0:8000 \
  --access-logfile - \
  --error-logfile -
```

### Docker

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## Testing

### pytest Configuration

```python
# tests/conftest.py
import pytest
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from app.core.database import Base
from app.main import app

@pytest.fixture
async def db_session():
    """Create test database session"""
    engine = create_async_engine("postgresql+asyncpg://test_user:test_pass@localhost/test_db")
    
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    async with AsyncSession(engine) as session:
        yield session
    
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
```

### Run Tests

```bash
# Run all tests
pytest

# With coverage
pytest --cov=app tests/

# Run specific test file
pytest tests/test_api/test_shipments.py

# Run with verbose output
pytest -v
```

---

## Performance Tips

1. **Use async/await**: All I/O operations should be async
2. **Connection pooling**: Configure appropriate pool_size
3. **Lazy loading**: Use `lazy="selectin"` for relationships
4. **Indexing**: Add indexes on foreign keys and filters
5. **Caching**: Use Redis for frequently accessed data
6. **Pagination**: Always limit query results
7. **Background tasks**: Use FastAPI background tasks for non-critical operations

---

## Related Documentation

- [Tech Stack Overview](overview.md)
- [Database Setup](database-postgresql.md)
- [API Specification](../04-api/openapi-spec.yaml)
- [System Architecture](../01-architecture/system-architecture.md)

