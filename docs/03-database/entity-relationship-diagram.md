# Entity Relationship Diagram - DAPK Database

Complete database schema visualization showing all entities and their relationships.

---

## Complete ERD

```mermaid
erDiagram
    %% Master Data Entities
    USERS ||--o{ SHIPMENTS : creates
    USERS ||--o{ COURIERS : "links to"
    USERS ||--o{ CUSTOMERS : "links to"
    USERS ||--o{ AUDIT_LOGS : performs
    
    WAREHOUSES ||--o{ SHIPMENTS : "origin"
    WAREHOUSES ||--o{ SHIPMENTS : "destination"
    WAREHOUSES ||--o{ SHIPMENTS : "current location"
    WAREHOUSES ||--o{ COURIERS : "assigned to"
    WAREHOUSES ||--o{ MANIFESTS_OUTBOUND : "origin"
    WAREHOUSES ||--o{ MANIFESTS_OUTBOUND : "destination"
    WAREHOUSES ||--o{ MANIFESTS_INBOUND : "destination"
    WAREHOUSES ||--o{ DELIVERY_RUNSHEETS : "operates from"
    WAREHOUSES ||--o{ CASH_REGISTERS : "belongs to"
    WAREHOUSES ||--o{ SURAT_JALAN : "originates from"
    
    COURIERS ||--o{ DELIVERY_RUNSHEETS : "assigned to"
    COURIERS ||--o{ CASH_REGISTERS : "submits"
    COURIERS ||--o{ PROOF_OF_DELIVERY : "delivers"
    COURIERS ||--o{ FAILED_DELIVERY_ATTEMPTS : "attempts"
    
    CUSTOMERS ||--o{ SHIPMENTS : "owns"
    
    TARIFFS ||--o{ SHIPMENTS : "applies to"
    
    %% Transactional Relationships
    SHIPMENTS ||--o{ TRACKING_EVENTS : "has history"
    SHIPMENTS ||--o| PROOF_OF_DELIVERY : "has proof"
    SHIPMENTS ||--o{ FAILED_DELIVERY_ATTEMPTS : "has attempts"
    
    MANIFESTS_OUTBOUND ||--o{ SHIPMENTS : contains
    MANIFESTS_OUTBOUND ||--o| MANIFESTS_INBOUND : "becomes"
    MANIFESTS_OUTBOUND ||--o| SURAT_JALAN : "documented by"
    
    MANIFESTS_INBOUND ||--o{ SHIPMENTS : "receives"
    
    DELIVERY_RUNSHEETS ||--o{ SHIPMENTS : includes
    DELIVERY_RUNSHEETS ||--o{ PROOF_OF_DELIVERY : "results in"
    DELIVERY_RUNSHEETS ||--o{ FAILED_DELIVERY_ATTEMPTS : "records"
    
    CASH_REGISTERS ||--o{ SHIPMENTS : reconciles
    
    %% Entity Definitions
    USERS {
        uuid id PK
        varchar email UK
        varchar password_hash
        varchar full_name
        varchar role
        boolean is_active
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    
    WAREHOUSES {
        uuid id PK
        varchar code UK
        varchar name
        text address
        varchar city
        boolean is_hub
        boolean is_active
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    
    COURIERS {
        uuid id PK
        uuid user_id FK
        varchar employee_id UK
        varchar full_name
        varchar phone
        uuid warehouse_id FK
        varchar vehicle_type
        boolean is_active
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    
    CUSTOMERS {
        uuid id PK
        uuid user_id FK
        varchar customer_code UK
        varchar full_name
        varchar phone
        varchar email
        varchar customer_type
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    
    TARIFFS {
        uuid id PK
        uuid origin_warehouse_id FK
        uuid destination_warehouse_id FK
        varchar service_type
        decimal min_weight_kg
        decimal max_weight_kg
        decimal base_rate
        decimal per_kg_rate
        boolean is_active
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    
    SHIPMENTS {
        uuid id PK
        varchar awb_number UK
        varchar sender_name
        varchar receiver_name
        uuid origin_warehouse_id FK
        uuid destination_warehouse_id FK
        decimal weight_kg
        varchar commodity
        varchar service_type
        decimal total_amount
        varchar status
        uuid customer_id FK
        uuid manifest_outbound_id FK
        uuid manifest_inbound_id FK
        uuid delivery_runsheet_id FK
        uuid cash_register_id FK
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    
    MANIFESTS_OUTBOUND {
        uuid id PK
        varchar manifest_number UK
        uuid origin_warehouse_id FK
        uuid destination_warehouse_id FK
        date departure_date
        integer total_shipments
        varchar status
        uuid sealed_by FK
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    
    MANIFESTS_INBOUND {
        uuid id PK
        varchar manifest_number UK
        uuid manifest_outbound_id FK
        uuid destination_warehouse_id FK
        date arrival_date
        integer expected_shipments
        integer received_shipments
        varchar status
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    
    SURAT_JALAN {
        uuid id PK
        varchar sj_number UK
        uuid warehouse_id FK
        uuid manifest_outbound_id FK
        varchar vehicle_number
        varchar driver_name
        timestamp departure_datetime
        varchar status
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    
    DELIVERY_RUNSHEETS {
        uuid id PK
        varchar runsheet_number UK
        uuid warehouse_id FK
        uuid courier_id FK
        date delivery_date
        integer total_shipments
        integer delivered_count
        integer failed_count
        varchar status
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    
    PROOF_OF_DELIVERY {
        uuid id PK
        uuid shipment_id FK
        uuid delivery_runsheet_id FK
        timestamp delivered_at
        uuid delivered_by FK
        varchar recipient_name
        text photo_url
        text signature_url
        timestamp created_at
    }
    
    FAILED_DELIVERY_ATTEMPTS {
        uuid id PK
        uuid shipment_id FK
        uuid delivery_runsheet_id FK
        timestamp attempted_at
        uuid attempted_by FK
        varchar failure_reason
        text failure_notes
        boolean will_retry
        timestamp created_at
    }
    
    CASH_REGISTERS {
        uuid id PK
        varchar register_number UK
        uuid warehouse_id FK
        uuid courier_id FK
        date register_date
        integer total_shipments
        decimal total_amount
        decimal cash_submitted
        varchar status
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }
    
    TRACKING_EVENTS {
        uuid id PK
        uuid shipment_id FK
        varchar event_type
        varchar status
        uuid location_id FK
        varchar location_name
        text description
        timestamp event_timestamp
        uuid created_by FK
        jsonb metadata
        timestamp created_at
    }
    
    AUDIT_LOGS {
        uuid id PK
        uuid user_id FK
        varchar action
        varchar entity_type
        uuid entity_id
        jsonb old_values
        jsonb new_values
        timestamp created_at
    }
```

---

## Key Relationships Explained

### Master Data
- **Users** can be linked to **Couriers** (for mobile app access) or **Customers** (for portal access)
- **Warehouses** serve as origin, destination, and current location for shipments
- **Tariffs** define pricing between warehouse pairs

### Shipment Flow
1. **Shipment** created at origin warehouse
2. Added to **Manifests_Outbound** for transport
3. **Surat_Jalan** documents handover to driver
4. **Manifests_Inbound** records arrival at destination
5. **Delivery_Runsheet** assigns to courier
6. **Proof_of_Delivery** captures successful delivery
7. **Cash_Register** reconciles COD payments

### Audit Trail
- **Tracking_Events** provides immutable history for each shipment
- **Audit_Logs** records all system actions for compliance

---

## Cardinality Notes

- **One-to-Many**: Most relationships (one warehouse has many shipments)
- **One-to-One**: Shipment to POD (one shipment has one proof of delivery)
- **Optional**: Many foreign keys nullable (shipment may not have manifest yet)
- **Immutable**: Tracking events and POD cannot be updated or deleted

---

**Related Documentation**:
- [Schema Definitions SQL](schema-definitions.sql)
- [Database Overview](schema-overview.md)
- [Soft Delete Strategy](soft-delete-strategy.md)
