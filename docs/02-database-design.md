# Database Design

## Overview

The SupplyMesh database is designed using PostgreSQL to support a multi-tenant procurement and supplier intelligence platform.

It manages organizations, users, roles, suppliers, procurement requests, approval workflows, purchase orders, invoices, payments, and audit logs.

## ER Diagram

The Entity Relationship Diagram (ERD) illustrates the database tables, their attributes, and relationships.

![SupplyMesh Entity Relationship Diagram](/diagrams/erd.png)

**Interactive Diagram:** [View SupplyMesh ER Diagram on dbdiagram.io](https://dbdiagram.io/d/6afb14b0d4a729077dfeda3e)

## Database Design Considerations

- **Relational Structure:** PostgreSQL ensures structured data management and relationship integrity.
- **Multi-Tenancy:** Organization-based data separation.
- **Role-Based Access Control:** Supports configurable roles and permissions.
- **Data Integrity:** Uses primary keys, foreign keys, and constraints.
- **Scalability:** Designed to support future procurement features.