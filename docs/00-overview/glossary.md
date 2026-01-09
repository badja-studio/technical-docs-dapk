# Glossary - DAPK Terms & Acronyms

Quick reference for terms used throughout the DAPK system documentation.

---

## Business Terms

### AWB (Air Waybill) Number
Unique tracking number assigned to each shipment. Format: `DAPK-YYYYMMDD-NNNNN`
- Example: `DAPK-20260109-00001`
- Used for tracking and identification throughout shipment lifecycle

### Shipment
A package or item being transported from origin to destination. Also called "kiriman" in Indonesian.

### Manifest
A document listing multiple shipments grouped together for transport.
- **Outbound Manifest**: Shipments leaving origin warehouse
- **Inbound Manifest**: Shipments arriving at destination warehouse

### Runsheet
A delivery route assignment listing shipments for a courier to deliver. Also called "delivery runsheet" or "surat jalan kurir".

### Surat Jalan
Handover document from warehouse to transport/midmile carrier. Records vehicle, driver, and manifest information with signatures.

### Cash Register
Daily reconciliation document showing all COD (Cash on Delivery) amounts collected by courier. Also called "packing list".

### POD (Proof of Delivery)
Evidence that shipment was delivered, including:
- Photo of recipient with package
- Recipient signature
- Recipient name and relationship
- Delivery timestamp

### COD (Cash on Delivery)
Payment method where recipient pays upon delivery rather than sender paying upfront.

### Origin
Starting location/warehouse where shipment begins journey.

### Destination
Target location/warehouse where shipment will be delivered.

### Hub
Main warehouse or distribution center that processes shipments.

### Branch
DAPK office location that accepts shipments and handles deliveries.

### Commodity
Type of goods being shipped (e.g., "Elektronik", "Dokumen", "Pakaian").

### Service Type
Shipping speed option:
- **Regular**: Standard delivery time
- **Express**: Faster delivery, higher cost

### Tariff
Shipping rate/price calculated based on weight, distance, and service type.

---

## Shipment Statuses

### pending
Shipment created but not yet confirmed. Awaiting verification.

### confirmed
Shipment verified and ready to be added to manifest.

### manifested_outbound
Shipment added to outbound manifest for transport.

### in_transit
Shipment departed from origin, traveling to destination.

### arrived_hub
Shipment arrived at destination warehouse/hub.

### manifested_inbound
Shipment processed at destination, ready for delivery assignment.

### ready_for_delivery
Shipment at destination warehouse, awaiting courier assignment.

### out_for_delivery
Courier has shipment and is attempting delivery.

### delivered
Shipment successfully delivered to recipient with POD captured.

### failed_delivery
Delivery attempt unsuccessful (recipient unavailable, wrong address, etc.).

### returned
Shipment being returned to sender after failed delivery attempts.

---

## System Modules

### Core System
Main back-office application for warehouse staff and administrators. Handles shipments, manifests, reports, and master data.

### Mobile App
Courier application for Android/iOS devices. Used for runsheet management and POD capture.

### Corporate Website
Public-facing website with customer portal, tracking, and company information.

---

## Technical Terms

### API (Application Programming Interface)
Interface that allows different software systems to communicate. The DAPK API enables mobile app and website to interact with core system.

### REST (Representational State Transfer)
Architecture style for APIs using HTTP methods (GET, POST, PUT, DELETE).

### JWT (JSON Web Token)
Security token format used for authentication. Users receive JWT after login, include it in subsequent requests.

### Endpoint
Specific URL in API that performs an action. Example: `/v1/shipments` to list shipments.

### UUID (Universally Unique Identifier)
128-bit unique identifier used as primary key for database records.
- Example: `550e8400-e29b-41d4-a716-446655440000`
- Advantages: No sequential ID leakage, distributed system friendly

### Soft Delete
Data deletion method that marks records as deleted (`deleted_at` timestamp) rather than removing them. Allows data recovery and maintains referential integrity.

### Load Balancer
System (Nginx) that distributes incoming requests across multiple backend servers for better performance and reliability.

### Horizontal Scaling
Adding more servers to handle increased load, rather than upgrading existing server (vertical scaling).

### Docker
Containerization platform that packages applications with dependencies for consistent deployment.

### Docker Compose
Tool for defining and running multi-container Docker applications using YAML configuration.

---

## Database Terms

### Schema
Structure and organization of database tables, columns, and relationships.

### Table
Database entity that stores records (rows) of related data.

### Primary Key
Unique identifier for each record in a table (we use UUID).

### Foreign Key
Column that references primary key of another table, establishing relationships.

### Index
Database structure that improves query performance on specific columns.

### Migration
Versioned changes to database schema, allowing controlled updates.

### ORM (Object-Relational Mapping)
Tool (SQLAlchemy) that maps database tables to Python classes.

### Transaction
Group of database operations that succeed or fail together (ACID properties).

---

## Report Types

### Uncashregister Report
Lists shipments with status 'delivered' but not yet reconciled in cash register. Indicates courier hasn't submitted cash.

### Unoutbound Report
Lists shipments in 'confirmed' status for extended period without being added to outbound manifest. Flags potential forgotten shipments.

### Uninbound Report
Lists shipments in 'in_transit' status too long without inbound manifest creation. Flags shipments delayed or lost in transit.

### Undelivery Runsheet Report
Lists shipments 'ready_for_delivery' for extended period without runsheet assignment. Indicates courier assignment backlog.

---

## User Roles

### Admin
Full system access, manages users, configures system settings.

### Supervisor
Creates manifests and runsheets, assigns work to staff and couriers.

### Warehouse Staff
Creates shipments, processes arrivals, generates receipts.

### Finance
Performs cash register verification and reconciliation.

### Courier
Mobile app user who delivers shipments and captures POD.

### Customer
Website user who tracks shipments and views history.

---

## API Response Terms

### 200 OK
Request successful, data returned.

### 201 Created
New resource created successfully (e.g., shipment created).

### 400 Bad Request
Request invalid, usually validation error.

### 401 Unauthorized
Authentication required or token invalid.

### 403 Forbidden
User doesn't have permission for action.

### 404 Not Found
Requested resource doesn't exist.

### 500 Internal Server Error
Server error, something went wrong processing request.

---

## Infrastructure Terms

### Nginx
Web server and load balancer that distributes traffic to backend instances.

### PostgreSQL
Relational database management system storing all DAPK data.

### FastAPI
Modern Python web framework for building APIs.

### Next.js
React framework for building frontend applications with server-side rendering.

### Redis
In-memory data store used for caching frequently accessed data (optional).

---

## Acronyms Quick Reference

- **API**: Application Programming Interface
- **AWB**: Air Waybill (tracking number)
- **COD**: Cash on Delivery
- **CMS**: Content Management System
- **CRUD**: Create, Read, Update, Delete
- **DDL**: Data Definition Language (SQL)
- **ERD**: Entity Relationship Diagram
- **HTTP**: Hypertext Transfer Protocol
- **HTTPS**: HTTP Secure (encrypted)
- **JWT**: JSON Web Token
- **ORM**: Object-Relational Mapping
- **POD**: Proof of Delivery
- **REST**: Representational State Transfer
- **SSL**: Secure Sockets Layer
- **TLS**: Transport Layer Security
- **UI**: User Interface
- **UX**: User Experience
- **UUID**: Universally Unique Identifier

---

## Indonesian to English

Common Indonesian terms used in DAPK context:

| Indonesian | English | Notes |
|------------|---------|-------|
| Kiriman | Shipment | Package being transported |
| Pengirim | Sender | Person sending package |
| Penerima | Receiver/Recipient | Person receiving package |
| Kurir | Courier | Delivery person |
| Gudang | Warehouse | Storage facility |
| Cabang | Branch | Office location |
| Tarif | Tariff/Rate | Shipping price |
| Resi | Receipt/AWB | Tracking number document |
| Manifest | Manifest | Shipment grouping document |
| Surat Jalan | Delivery Order | Handover document |
| Packing List | Packing List | Cash register document |
| Berat | Weight | Shipment weight |
| Barang | Goods/Commodity | Item type |
| Tujuan | Destination | Target location |
| Asal | Origin | Starting location |

---

## Common Workflows

### Shipment Creation Flow
Input details → Calculate tariff → Generate AWB → Print receipt → Status: pending

### Manifest Flow
Select shipments → Create manifest → Seal → Status: manifested

### Delivery Flow
Create runsheet → Assign courier → Out for delivery → Deliver → Capture POD → Status: delivered

### Cash Reconciliation Flow
End of day → Courier submits cash → Finance verifies → Close register → Complete

---

## Date/Time Formats

### ISO 8601
Standard date-time format used in API: `YYYY-MM-DDTHH:MM:SSZ`
- Example: `2026-01-09T14:30:00Z`

### Display Format
User-friendly format: `DD/MM/YYYY HH:MM`
- Example: `09/01/2026 14:30`

---

## Document Conventions

### File Paths
- Use absolute paths from repository root
- Example: `/Users/mac/Documents/code/technical-docs-dapk/docs/...`

### Code Blocks
```python
# Python code examples
```

```sql
-- SQL examples
```

```yaml
# YAML configuration
```

### Emphasis
- **Bold**: Important terms, actions
- *Italic*: Emphasis, notes
- `Code`: Technical terms, commands, code references

---

**Related Documentation**:
- [Executive Summary](executive-summary.md)
- [System Architecture](../01-architecture/system-architecture.md)
- [API Overview](../04-api/api-overview.md)
