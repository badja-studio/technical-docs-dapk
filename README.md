# DAPK Technical Documentation

Complete technical documentation for **PT. Duta Angkasa Prima Kargo** (DAPK) logistics system.

## 📋 Table of Contents

- [Overview](#overview)
- [System Components](#system-components)
- [Quick Links](#quick-links)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Documentation Structure](#documentation-structure)

---

## Overview

DAPK is a comprehensive logistics management system designed to streamline shipping operations, from shipment creation to final delivery. The system includes three main components:

1. **Core System** - Complete shipment and manifest management
2. **Mobile App** - Courier application for delivery operations
3. **Corporate Website** - Customer portal and public information

**For Stakeholders**: Start with the [Executive Summary](docs/00-overview/executive-summary.md)
**For Developers**: Jump to [Getting Started](docs/07-development/getting-started.md)
**For DevOps**: See [Deployment Guide](docs/06-deployment/deployment-guide.md)

---

## System Components

### 1. Core System
Complete back-office operations for logistics management:

- ✅ **Shipment Management** - Input shipments with auto tariff calculation & AWB generation
- ✅ **Manifest Management** - Outbound & Inbound manifest creation and tracking
- ✅ **Surat Jalan** - Warehouse to transport handover documentation
- ✅ **Cash Register** - Daily transaction reconciliation and packing lists
- ✅ **Delivery Runsheet** - Courier assignment and route management
- ✅ **Tracking System** - Real-time shipment tracking with complete history
- ✅ **Reports** - Uncashregister, Unoutbound, Uninbound, Undelivery monitoring
- ✅ **Master Data** - Management of couriers, tariffs, warehouses, locations

### 2. Mobile App for Couriers
Dedicated mobile application for delivery personnel:

- 📱 **Runsheet Management** - View and manage assigned delivery routes
- 📱 **Proof of Delivery (POD)** - Capture photos and signatures
- 📱 **Real-time Updates** - Update shipment status on-the-go
- 📱 **Profile Management** - Courier account and settings

### 3. Corporate Website
Public-facing website and customer portal:

- 🌐 **Customer Registration** - Self-service account creation
- 🌐 **Dashboard** - Customer shipment history and analytics
- 🌐 **Public Tracking** - Track shipments without login
- 🌐 **Tariff Calculator** - Estimate shipping costs
- 🌐 **Content Pages** - Products, services, news, promotions
- 🌐 **Branch Locator** - Find DAPK locations
- 🌐 **CMS** - Content management for administrators

---

## Quick Links

### 📚 Documentation
- [Executive Summary](docs/00-overview/executive-summary.md) - High-level overview for stakeholders
- [Project Scope](docs/00-overview/project-scope.md) - Objectives and deliverables
- [Glossary](docs/00-overview/glossary.md) - Terms and acronyms

### 🏗️ Architecture
- [System Architecture](docs/01-architecture/system-architecture.md) - Complete system design
- [Scalability Strategy](docs/01-architecture/scalability-strategy.md) - Load balancing and scaling
- [Security Architecture](docs/01-architecture/security-architecture.md) - Authentication and authorization

### 💾 Database
- [Database Schema Overview](docs/03-database/schema-overview.md) - Design principles
- [Entity Relationship Diagram](docs/03-database/entity-relationship-diagram.md) - Complete ERD
- [Schema Definitions (SQL)](docs/03-database/schema-definitions.sql) - Full DDL
- [Soft Delete Strategy](docs/03-database/soft-delete-strategy.md) - Implementation guide

### 🔌 API
- [OpenAPI Specification](docs/04-api/openapi-spec.yaml) - Complete API documentation
- [API Overview](docs/04-api/api-overview.md) - REST conventions
- [Authentication](docs/04-api/authentication.md) - JWT authentication flow
- [Error Handling](docs/04-api/error-handling.md) - Standard error responses

### 🔄 Application Flows
- [Shipment Lifecycle](docs/05-application-flows/shipment-lifecycle.md) - End-to-end flow
- [Flow Diagrams](docs/05-application-flows/flows/) - Detailed process flows

### 🚀 Deployment
- [Deployment Guide](docs/06-deployment/deployment-guide.md) - Step-by-step deployment
- [Docker Setup](docs/06-deployment/docker-setup.md) - Container configuration
- [Nginx Configuration](docs/06-deployment/nginx-configuration.md) - Load balancer setup
- [Scaling Guide](docs/06-deployment/scaling-guide.md) - How to scale the system

### 👨‍💻 Development
- [Getting Started](docs/07-development/getting-started.md) - Local development setup
- [Coding Standards](docs/07-development/coding-standards.md) - Code conventions
- [Testing Strategy](docs/07-development/testing-strategy.md) - Testing approach

---

## Tech Stack

### Backend
- **Language**: Python 3.11+
- **Framework**: FastAPI
- **Database**: PostgreSQL 15+
- **ORM**: SQLAlchemy (async)
- **Authentication**: JWT (python-jose)
- **Validation**: Pydantic

### Frontend
- **Framework**: Next.js 14
- **Language**: TypeScript
- **UI Library**: React 18
- **Styling**: Tailwind CSS (recommended)
- **State Management**: React Context / Zustand

### Infrastructure
- **Containerization**: Docker & Docker Compose
- **Load Balancer**: Nginx
- **Cache**: Redis (optional)
- **File Storage**: Local volumes / S3-compatible

### Development Tools
- **API Testing**: Swagger UI (auto-generated)
- **Database Migration**: Alembic
- **Code Quality**: Black, Pylint, ESLint, Prettier

---

## Getting Started

### Prerequisites
- Docker 24+ and Docker Compose
- Git
- (Optional) Python 3.11+ and Node.js 18+ for local development

### Quick Start

1. **Clone the repository** (when implementation repo is ready)
```bash
git clone https://github.com/your-org/dapk-system.git
cd dapk-system
```

2. **Set up environment variables**
```bash
cp .env.example .env
# Edit .env with your configuration
```

3. **Start all services**
```bash
docker-compose up -d
```

4. **Access the applications**
- Core System: http://localhost:3000
- API Documentation: http://localhost:8000/docs
- PostgreSQL: localhost:5432

For detailed setup instructions, see [Getting Started Guide](docs/07-development/getting-started.md).

---

## Documentation Structure

```
docs/
├── 00-overview/          # High-level documentation for all audiences
├── 01-architecture/      # System architecture and design decisions
├── 02-tech-stack/        # Technology choices and configurations
├── 03-database/          # Database schema and design
│   └── tables/           # Individual table specifications
├── 04-api/               # API documentation
│   └── endpoints/        # Endpoint specifications by module
├── 05-application-flows/ # Business process flows and diagrams
│   └── flows/            # Detailed flow documents
├── 06-deployment/        # Deployment and operations guides
└── 07-development/       # Development guidelines and setup

docker/
├── backend/              # Backend Dockerfile
├── frontend/             # Frontend Dockerfile
├── nginx/                # Nginx configuration
│   └── conf.d/           # Site configurations
└── postgresql/           # PostgreSQL initialization
```

---

## Key Features

### Shipment Management
- Automatic AWB (Air Waybill) number generation
- Dynamic tariff calculation based on weight, distance, service type
- Receipt generation with barcode/QR code
- Real-time status updates
- Complete tracking history

### Manifest Operations
- Outbound manifest creation for origin → destination
- Inbound manifest verification
- Discrepancy reporting
- Seal and signature workflow

### Delivery Operations
- Runsheet generation and assignment
- Mobile app integration for couriers
- GPS tracking (future enhancement)
- Proof of Delivery with photo capture
- Failed delivery management with retry logic

### Financial Operations
- Cash register reconciliation
- Packing list generation
- Discrepancy management
- COD (Cash on Delivery) tracking

### Reporting & Analytics
- Uncashregister report - Delivered but not reconciled
- Unoutbound report - Pending manifest creation
- Uninbound report - In transit too long
- Undelivery report - Not assigned to courier
- Daily summary reports
- Courier performance metrics

---

## System Status Definitions

### Shipment Statuses
- **pending** - Shipment created, awaiting confirmation
- **confirmed** - Verified and ready for manifest
- **manifested_outbound** - Added to outbound manifest
- **in_transit** - En route to destination
- **arrived_hub** - Arrived at destination warehouse
- **manifested_inbound** - Processed at destination
- **ready_for_delivery** - Awaiting courier assignment
- **out_for_delivery** - Courier is delivering
- **delivered** - Successfully delivered
- **failed_delivery** - Delivery attempt failed
- **returned** - Returned to sender

---

## API Base URLs

- **Production**: `https://api.dapk.com/v1`
- **Staging**: `https://staging-api.dapk.com/v1`
- **Local Development**: `http://localhost:8000/v1`

---

## Support & Contact

- **Technical Issues**: See [Troubleshooting Guide](docs/07-development/troubleshooting.md)
- **API Questions**: Check [API Documentation](docs/04-api/openapi-spec.yaml)
- **Architecture Questions**: Review [Architecture Docs](docs/01-architecture/)

---

## Version History

- **v1.0.0** (2026-01-09) - Initial technical documentation release

---

## License

© 2026 PT. Duta Angkasa Prima Kargo. All rights reserved.

---

**Next Steps**:
- Read the [Executive Summary](docs/00-overview/executive-summary.md) for business context
- Review [System Architecture](docs/01-architecture/system-architecture.md) for technical overview
- Check [Database Schema](docs/03-database/schema-overview.md) for data model
- Explore [API Specification](docs/04-api/openapi-spec.yaml) for integration details
