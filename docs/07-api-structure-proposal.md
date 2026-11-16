# SupplyMesh API Structure Proposal

## 1. Overview

This document proposes the REST API structure for the SupplyMesh Global Procurement & Supplier Intelligence Platform.

The API will connect the Next.js frontend with the NestJS backend and provide access to organization management, supplier management, procurement, approval workflows, purchase orders, invoices, and payments.

### 1.1 Technology Stack

| Component | Technology |
|---|---|
| Backend Framework | NestJS |
| API Style | REST API |
| API Format | JSON |
| Database | PostgreSQL |
| ORM | Drizzle ORM |
| Authentication | JWT |
| Authorization | Role-Based Access Control (RBAC) |
| Cache | Redis |
| Background Jobs | BullMQ |
| API Versioning | URL-based versioning |

## 2. API Design Principles

The API will follow these principles:

- **RESTful Design:** Use standard HTTP methods and resource-based URLs.
- **Versioning:** Prefix endpoints with `/api/v1`.
- **JSON Communication:** Use JSON for request and response bodies.
- **Authentication:** Protect private endpoints using JWT authentication.
- **Authorization:** Enforce role and permission checks.
- **Multi-Tenant Isolation:** Ensure users can access only authorized organization data.
- **Validation:** Validate incoming requests using DTOs and validation rules.
- **Consistent Responses:** Follow a consistent response and error structure.
- **Pagination:** Use pagination for endpoints returning large collections.

## 3. Base URL and API Versioning

### Base URL

```text
/api/v1
```

Example:

```text
GET /api/v1/suppliers
```

### Versioning Strategy

SupplyMesh will use URL-based API versioning.

- `/api/v1` represents the first stable API version.
- Breaking changes will be introduced through a new version, such as `/api/v2`.
- Non-breaking improvements can be added to the existing version.

## 4. Authentication and Authorization

### 4.1 Authentication

JWT-based authentication will be used to identify authenticated users.

Private API requests must include an access token in the Authorization header.

Example:

```http
Authorization: Bearer <access_token>
```

Authentication endpoints:

| Method | Endpoint | Description |
|---|---|---|
| POST | `/auth/register` | Register a user |
| POST | `/auth/login` | Authenticate a user |
| POST | `/auth/refresh` | Refresh an access token |
| POST | `/auth/logout` | Log out the current session |
| GET | `/auth/me` | Get the authenticated user's profile |

### 4.2 Authorization

SupplyMesh will use Role-Based Access Control (RBAC).

Each authenticated user must have an active membership in the selected organization before accessing its protected resources.

Authorization checks will verify:

1. User authentication.
2. Organization membership.
3. Assigned roles.
4. Required permissions.
5. Resource ownership and organization scope.

Example:

A user with supplier-management permission can manage suppliers, while a user without that permission cannot perform supplier-management operations.

## 5. Multi-Tenant API Handling

SupplyMesh uses a shared database and shared schema multi-tenant architecture.

The API must enforce organization-level data isolation.

### 5.1 Organization Context

Organization context can be provided through a request header:

```http
X-Organization-Id: <organization_id>
```

The backend must verify that the authenticated user belongs to the requested organization.

The organization ID provided by the client must never be trusted without validation.

### 5.2 Tenant Isolation Rules

- Every organization-scoped query must include the active organization scope.
- Users must have an active membership in the requested organization.
- Users cannot access another organization's suppliers, requests, purchase orders, or invoices.
- Resource IDs must be validated against the active organization.
- Organization context must be established by trusted backend logic before accessing tenant data.

## 6. API Module Structure

The API will be organized into independent NestJS modules based on business responsibilities.

### 6.1 Authentication Module

| Method | Endpoint | Description |
|---|---|---|
| POST | `/auth/register` | Register a new user |
| POST | `/auth/login` | Log in |
| POST | `/auth/refresh` | Refresh access token |
| POST | `/auth/logout` | Log out |
| GET | `/auth/me` | Get current user |

### 6.2 Organization Module

| Method | Endpoint | Description |
|---|---|---|
| POST | `/organizations` | Create an organization |
| GET | `/organizations` | Get organizations available to the user |
| GET | `/organizations/:organizationId` | Get organization details |
| PATCH | `/organizations/:organizationId` | Update organization details |
| GET | `/organizations/:organizationId/members` | List organization members |
| POST | `/organizations/:organizationId/invitations` | Invite a user |
| PATCH | `/organizations/:organizationId/members/:membershipId` | Update membership information |

### 6.3 Department Module

| Method | Endpoint | Description |
|---|---|---|
| POST | `/departments` | Create a department |
| GET | `/departments` | List departments |
| GET | `/departments/:departmentId` | Get department details |
| PATCH | `/departments/:departmentId` | Update a department |
| DELETE | `/departments/:departmentId` | Delete a department |

### 6.4 User and RBAC Module

| Method | Endpoint | Description |
|---|---|---|
| GET | `/users` | List users available to the organization |
| GET | `/users/:userId` | Get user details |
| PATCH | `/users/:userId` | Update user information |
| GET | `/roles` | List organization roles |
| POST | `/roles` | Create a role |
| PATCH | `/roles/:roleId` | Update a role |
| DELETE | `/roles/:roleId` | Delete a role |
| GET | `/permissions` | List available permissions |
| PUT | `/memberships/:membershipId/roles` | Assign roles to a membership |

### 6.5 Supplier Management Module

| Method | Endpoint | Description |
|---|---|---|
| POST | `/suppliers` | Create a supplier |
| GET | `/suppliers` | List suppliers |
| GET | `/suppliers/:supplierId` | Get supplier details |
| PATCH | `/suppliers/:supplierId` | Update supplier information |
| DELETE | `/suppliers/:supplierId` | Remove a supplier |
| POST | `/suppliers/:supplierId/documents` | Upload supplier documents |
| GET | `/suppliers/:supplierId/documents` | List supplier documents |
| PATCH | `/suppliers/:supplierId/status` | Update supplier verification status |
| GET | `/suppliers/:supplierId/performance` | Get supplier performance records |

### 6.6 Procurement Request Module

| Method | Endpoint | Description |
|---|---|---|
| POST | `/purchase-requests` | Create a purchase request |
| GET | `/purchase-requests` | List purchase requests |
| GET | `/purchase-requests/:requestId` | Get request details |
| PATCH | `/purchase-requests/:requestId` | Update a draft request |
| POST | `/purchase-requests/:requestId/submit` | Submit a request for approval |
| POST | `/purchase-requests/:requestId/cancel` | Cancel a request where permitted |
| POST | `/purchase-requests/:requestId/items` | Add request items |
| PATCH | `/purchase-requests/:requestId/items/:itemId` | Update a request item |
| DELETE | `/purchase-requests/:requestId/items/:itemId` | Remove a request item |

### 6.7 Approval Workflow Module

| Method | Endpoint | Description |
|---|---|---|
| POST | `/approval-workflows` | Create an approval workflow |
| GET | `/approval-workflows` | List approval workflows |
| GET | `/approval-workflows/:workflowId` | Get workflow details |
| PATCH | `/approval-workflows/:workflowId` | Update a workflow |
| POST | `/approval-workflows/:workflowId/steps` | Add an approval step |
| PATCH | `/approval-workflows/:workflowId/steps/:stepId` | Update an approval step |
| GET | `/purchase-requests/:requestId/approvals` | Get request approval history |
| POST | `/purchase-requests/:requestId/approvals/:approvalId/approve` | Approve a request |
| POST | `/purchase-requests/:requestId/approvals/:approvalId/reject` | Reject a request |

### 6.8 Purchase Order Module

| Method | Endpoint | Description |
|---|---|---|
| POST | `/purchase-orders` | Create a purchase order |
| GET | `/purchase-orders` | List purchase orders |
| GET | `/purchase-orders/:purchaseOrderId` | Get purchase order details |
| PATCH | `/purchase-orders/:purchaseOrderId` | Update an eligible purchase order |
| PATCH | `/purchase-orders/:purchaseOrderId/status` | Update purchase order status |
| GET | `/purchase-orders/:purchaseOrderId/items` | Get purchase order items |

### 6.9 Invoice Module

| Method | Endpoint | Description |
|---|---|---|
| POST | `/invoices` | Submit an invoice |
| GET | `/invoices` | List invoices |
| GET | `/invoices/:invoiceId` | Get invoice details |
| PATCH | `/invoices/:invoiceId` | Update an eligible invoice |
| POST | `/invoices/:invoiceId/verify` | Verify an invoice |
| POST | `/invoices/:invoiceId/reject` | Reject an invoice |

### 6.10 Payment Module

| Method | Endpoint | Description |
|---|---|---|
| POST | `/invoices/:invoiceId/payments` | Record a payment |
| GET | `/invoices/:invoiceId/payments` | Get payments for an invoice |
| GET | `/payments` | List payments |
| GET | `/payments/:paymentId` | Get payment details |
| PATCH | `/payments/:paymentId/status` | Update payment status |

### 6.11 Audit Log Module

| Method | Endpoint | Description |
|---|---|---|
| GET | `/audit-logs` | List organization audit logs |
| GET | `/audit-logs/:logId` | Get audit log details |

Audit logs should be read-only through the regular API. The backend generates audit records when relevant actions occur.

## 7. Request and Response Structure

The API will use a consistent JSON response format.

### 7.1 Successful Response

Example:

```json
{
  "success": true,
  "message": "Supplier created successfully",
  "data": {
    "id": "supplier-uuid",
    "companyName": "Global Trading Ltd",
    "status": "PENDING"
  }
}
```

### 7.2 Error Response

Example:

```json
{
  "success": false,
  "message": "You do not have permission to perform this action",
  "error": {
    "code": "FORBIDDEN",
    "details": null
  }
}
```

### 7.3 HTTP Status Codes

| Status Code | Meaning |
|---|---|
| 200 | Request successful |
| 201 | Resource created successfully |
| 204 | Request successful with no response body |
| 400 | Invalid request |
| 401 | Authentication required |
| 403 | Permission denied |
| 404 | Resource not found |
| 409 | Resource conflict |
| 422 | Validation error |
| 429 | Too many requests |
| 500 | Internal server error |

## 8. Pagination, Filtering, and Sorting

Collection endpoints should support pagination and filtering where appropriate.

### 8.1 Pagination

Example:

```http
GET /api/v1/suppliers?page=1&limit=20
```

Example response:

```json
{
  "success": true,
  "message": "Suppliers retrieved successfully",
  "data": [
    {
      "id": "supplier-uuid",
      "companyName": "Global Trading Ltd"
    }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

### 8.2 Filtering

Example:

```http
GET /api/v1/suppliers?status=APPROVED&category=Electronics
```

### 8.3 Sorting

Example:

```http
GET /api/v1/purchase-requests?sortBy=createdAt&sortOrder=desc
```

The backend must validate pagination, filter, and sorting parameters to prevent invalid queries.

## 9. Validation and Error Handling

### 9.1 Request Validation

NestJS DTOs and validation rules will validate incoming request data.

Examples:

- Required fields must not be empty.
- Email addresses must have a valid format.
- Monetary values must not be negative.
- Quantities must be greater than zero.
- Resource IDs must have valid formats.
- Status values must match supported values.
- Uploaded files must follow allowed file types and size limits.

### 9.2 Centralized Error Handling

A centralized exception-handling mechanism will ensure consistent error responses.

The API should handle:

- Validation errors.
- Authentication errors.
- Authorization errors.
- Missing resources.
- Duplicate records.
- Database constraint violations.
- Unexpected server errors.

Internal stack traces and sensitive information must not be exposed in production responses.

## 10. API Security

The following security measures will be applied:

- JWT-based authentication for protected routes.
- RBAC permission checks.
- Organization membership validation.
- Tenant-aware database queries.
- DTO-based request validation.
- Rate limiting for sensitive endpoints.
- Secure password hashing.
- Safe file upload validation.
- HTTPS for production communication.
- Audit logging for important business operations.
- Protection against unauthorized resource access.

## 11. NestJS Module Organization

The backend will follow a modular architecture.

Proposed structure:

```text
src/
├── modules/
│   ├── auth/
│   ├── organizations/
│   ├── users/
│   ├── memberships/
│   ├── departments/
│   ├── roles/
│   ├── permissions/
│   ├── suppliers/
│   ├── procurement/
│   ├── approvals/
│   ├── purchase-orders/
│   ├── invoices/
│   ├── payments/
│   └── audit-logs/
│
├── common/
│   ├── decorators/
│   ├── guards/
│   ├── interceptors/
│   ├── filters/
│   ├── pipes/
│   └── dto/
│
├── database/
├── config/
└── main.ts
```

Each module will contain its own controllers, services, DTOs, and related business logic.

### Responsibilities

| Layer | Responsibility |
|---|---|
| Controller | Handles HTTP requests and responses |
| Service | Contains business logic |
| DTO | Defines and validates request data |
| Guard | Handles authentication and authorization |
| Repository / Data Access | Handles database operations through Drizzle ORM |
| Interceptor | Handles cross-cutting request/response behavior |
| Exception Filter | Handles API errors consistently |

## 12. API Documentation

The API will use Swagger / OpenAPI documentation.

Swagger will provide:

- Available API endpoints.
- HTTP methods.
- Request parameters.
- Request body schemas.
- Response schemas.
- Authentication requirements.
- Error response examples.

The documentation will help frontend developers understand and integrate with backend APIs.

## 13. Future API Improvements

The following improvements may be introduced when the platform grows:

- WebSocket or Server-Sent Events for real-time notifications.
- Advanced search and filtering for supplier intelligence.
- API usage analytics and monitoring.
- Webhook support for external integrations.
- Additional API versions for major changes.
- Bulk import and export endpoints.
- External procurement and accounting integrations.

## 14. Conclusion

The proposed API structure provides a modular and scalable foundation for SupplyMesh. RESTful endpoints, JWT authentication, RBAC, organization-level data isolation, consistent responses, and centralized validation will help maintain a secure and maintainable backend.

All endpoints in this document are proposed API contracts and should be finalized during implementation.