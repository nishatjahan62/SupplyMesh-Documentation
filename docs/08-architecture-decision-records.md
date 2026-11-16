# Architecture Decision Records (ADR)

## 1. Overview

This document records the major technical decisions made during the design of SupplyMesh.

Each decision considers the product requirements, available alternatives, advantages, disadvantages, and trade-offs.

The purpose is to keep the architecture consistent, maintainable, and scalable.

---

## ADR-001: PostgreSQL Instead of MongoDB

**Status:** Accepted

### Problem
SupplyMesh manages highly related data such as organizations, suppliers, purchase requests, approvals, purchase orders, invoices, and payments.

The database must maintain strong relationships and transaction consistency.

### Available Alternatives
- PostgreSQL
- MySQL
- MongoDB

### Selected Solution
PostgreSQL as the primary database.

### Reason for Selection
SupplyMesh requires relational relationships, foreign keys, complex queries, and reliable financial transactions.

PostgreSQL supports these requirements directly.

### Advantages
- Strong relational data integrity.
- ACID transactions.
- Foreign keys and constraints.
- Advanced indexing and query capabilities.
- Row-Level Security support.

### Disadvantages
- Requires proper schema design.
- Schema changes need database migrations.

### Trade-offs
A structured relational database requires more planning than a flexible document database, but provides better consistency for procurement and financial operations.

---

## ADR-002: Drizzle ORM Instead of Prisma

**Status:** Accepted

### Problem
The backend needs a database access solution that supports TypeScript, PostgreSQL, migrations, and complex relational queries.

### Available Alternatives
- Drizzle ORM
- Prisma
- Raw SQL

### Selected Solution
Drizzle ORM.

### Reason for Selection
Drizzle provides a TypeScript-based query interface while keeping queries relatively close to SQL.

This is useful for SupplyMesh because its procurement operations involve relational queries and transactions.

### Advantages
- Type-safe database queries.
- SQL-like query structure.
- PostgreSQL support.
- Lightweight ORM approach.
- Control over complex queries.

### Disadvantages
- Requires more manual query and schema management.
- Some operations may need additional SQL knowledge.

### Trade-offs
Drizzle provides more direct SQL control, while Prisma offers a more abstract developer experience. The project prioritizes SQL transparency and query control.

---

## ADR-003: Modular Monolith Instead of Microservices

**Status:** Accepted

### Problem
SupplyMesh contains several connected business modules, including supplier management, procurement, approvals, and finance.

The architecture must remain manageable during initial development.

### Available Alternatives
- Modular Monolith
- Microservices
- Service-Oriented Architecture (SOA)

### Selected Solution
Modular Monolithic Architecture.

### Reason for Selection
The modules share business data and frequently communicate with each other.

A modular monolith keeps deployment simple while maintaining clear module boundaries.

### Advantages
- Easier development and deployment.
- Simpler communication between modules.
- Easier transaction management.
- Lower infrastructure complexity.

### Disadvantages
- Modules share the same deployment lifecycle.
- Poor module boundaries can create tight coupling.
- Independent scaling is limited.

### Trade-offs
The project accepts shared deployment in exchange for simpler development and operations. Individual modules can be extracted into services when justified by future requirements.

---

## ADR-004: Redis for Caching and Background Jobs

**Status:** Accepted

### Problem
SupplyMesh needs efficient access to frequently requested data and reliable processing of time-consuming tasks such as email notifications and report generation.

### Available Alternatives
- PostgreSQL only
- Redis
- Other dedicated caching and queue systems

### Selected Solution
Redis for caching and job queue coordination.

### Reason for Selection
Redis supports fast temporary data access and can be used with BullMQ for background job processing.

### Advantages
- Reduces repeated database queries.
- Supports background job queues.
- Supports job retries and delayed execution.
- Helps improve application responsiveness.

### Disadvantages
- Requires additional infrastructure.
- Cache invalidation needs careful handling.
- Queue failures require monitoring and recovery.

### Trade-offs
Redis introduces infrastructure complexity, but it helps separate time-consuming tasks from normal API requests.

PostgreSQL remains the source of truth for business data. Redis is not used as the permanent storage for procurement records.

---

## ADR-005: Shared Database and Schema for Multi-Tenancy

**Status:** Accepted

### Problem
Multiple organizations will use SupplyMesh. Their data must remain isolated while the platform stays manageable.

### Available Alternatives
- Separate database per organization.
- Separate schema per organization.
- Shared database and shared schema.

### Selected Solution
Shared PostgreSQL database and shared schema with `organization_id`.

### Reason for Selection
The shared-schema approach reduces infrastructure and maintenance complexity while supporting multiple organizations.

### Advantages
- Lower infrastructure cost.
- Easier database management.
- Centralized migrations.
- Efficient resource utilization.

### Disadvantages
- Requires strict tenant-level access checks.
- Queries must always apply organization filters.
- A tenant isolation mistake could expose another organization's data.

### Trade-offs
The project accepts shared infrastructure in exchange for easier operations. Backend authorization, tenant-aware queries, composite foreign keys, and PostgreSQL Row-Level Security where appropriate will strengthen isolation.

---

## ADR-006: Database Tables for Approval Workflows

**Status:** Accepted

### Problem
SupplyMesh needs configurable multi-level approval workflows based on organizational rules, purchase amounts, and approver roles.

### Available Alternatives
- Store workflows as JSON.
- Store workflows in relational tables.
- Hardcode approval rules in backend logic.

### Selected Solution
Use relational tables such as:

- `approval_workflows`
- `approval_rules`
- `approval_steps`
- `purchase_request_approvals`

### Reason for Selection
Approval workflows require structured relationships, searchable rules, and a history of individual approval decisions.

Relational tables make these records easier to validate and manage.

### Advantages
- Flexible approval configuration.
- Clear approval history.
- Easier reporting.
- Better relational integrity.

### Disadvantages
- Requires multiple related tables.
- Complex workflows need careful service logic.

### Trade-offs
The design requires more tables than a JSON-based approach, but provides better data consistency and reporting capabilities.

Workflow versions or snapshots should be maintained so that changes to future approval rules do not alter historical decisions.

---

## ADR-007: Database Transactions for Invoice and Purchase Order Consistency

**Status:** Accepted

### Problem
Purchase orders, invoices, and payments are financially related. Partial database updates could create inconsistent records.

### Available Alternatives
- Independent database updates.
- Database transactions.
- Distributed transaction management.

### Selected Solution
Use PostgreSQL database transactions for related business updates.

### Reason for Selection
Operations such as creating a purchase order with its items or recording a payment and updating an invoice must remain consistent.

### Advantages
- Atomic database operations.
- Prevents incomplete updates.
- Maintains financial data consistency.
- Supports rollback after failures.

### Disadvantages
- Long transactions can reduce performance.
- Concurrent updates require proper handling.

### Trade-offs
Transactions add some database overhead, but consistency is more important for procurement and financial records.

External payment processing must be handled separately using idempotency keys and reconciliation because an external payment cannot be rolled back by a PostgreSQL transaction.

---

## ADR-008: BullMQ for Background Processing

**Status:** Accepted

### Problem
Email delivery, document processing, and report generation can take time and should not block normal API requests.

### Available Alternatives
- Process everything inside API requests.
- Use Redis with BullMQ.
- Use a managed cloud queue service.

### Selected Solution
BullMQ with Redis and separate background workers.

### Reason for Selection
BullMQ supports asynchronous processing, retry policies, delayed jobs, and job tracking.

### Advantages
- Keeps API responses faster.
- Supports retry handling.
- Separates background tasks from API operations.
- Allows workers to scale independently.

### Disadvantages
- Requires worker management.
- Failed jobs need monitoring.
- Duplicate job execution must be considered.

### Trade-offs
The queue introduces another infrastructure component, but improves reliability and responsiveness for time-consuming tasks.

Jobs should be designed to be idempotent to avoid duplicate business operations.

---

## ADR-009: Object Storage Instead of Database File Storage

**Status:** Accepted

### Problem
SupplyMesh needs to store supplier documents, invoices, and procurement attachments.

Storing large files directly in database records can increase database storage and backup requirements.

### Available Alternatives
- Store files inside PostgreSQL.
- Use local server storage.
- Use S3-compatible object storage.

### Selected Solution
S3-compatible object storage for file contents and PostgreSQL for document metadata.

### Reason for Selection
Object storage is designed for managing files independently from relational business data.

### Advantages
- Suitable for large files.
- Separates file storage from database storage.
- Supports private access and signed URLs.
- Allows independent storage scaling.

### Disadvantages
- Requires an external storage service.
- Upload and database operations need failure handling.
- Storage permissions must be managed carefully.

### Trade-offs
The architecture introduces a separate storage dependency, but reduces database storage overhead and supports future document growth.

---

## 10. Summary of Technical Decisions

| ADR | Decision | Selected Solution |
|---|---|---|
| ADR-001 | Primary Database | PostgreSQL |
| ADR-002 | ORM | Drizzle ORM |
| ADR-003 | Application Architecture | Modular Monolith |
| ADR-004 | Caching | Redis |
| ADR-005 | Multi-Tenancy | Shared Database and Schema |
| ADR-006 | Approval Workflows | Relational Tables |
| ADR-007 | Data Consistency | PostgreSQL Transactions |
| ADR-008 | Background Processing | BullMQ with Redis |
| ADR-009 | File Storage | S3-Compatible Object Storage |

## 11. Conclusion

These decisions are based on SupplyMesh's business requirements, including multi-tenant data isolation, procurement workflows, approval management, financial consistency, and future scalability.

The selected technologies provide a practical foundation for building and maintaining the initial platform without introducing unnecessary architectural complexity.