# TechZone Architecture

## Overview

TechZone is a web-based inventory and sales management system built using PHP and Oracle Database.

The browser communicates with the PHP application through Apache. PHP handles authentication, authorization, validation, business logic, transaction processing, and database access through Oracle OCI8.

The preserved v1.0 release was developed and regression-tested in a Windows/XAMPP environment. A separate OCI demonstration deployment now validates the clean TechZone schema and core application workflows against Oracle Autonomous Database 26ai.

## Application Architecture

```text
Web Browser
    │ HTTP / HTTPS
    ▼
Apache / PHP 8
    │ OCI8
    ▼
Oracle Database
```

## Application Layers

### Presentation Layer

HTML5, CSS3, and JavaScript provide the operator interface for the dashboard, inventory, orders, customers, suppliers, authentication, and transaction processing.

### Application Layer

PHP 8 handles authentication, session management, role-based authorization, CSRF validation, input validation, inventory operations, sales processing, search, dashboard calculations, auditing, logging, and protected error handling.

### Database Access

TechZone communicates with Oracle through PHP OCI8. Database operations use Oracle bind variables rather than constructing SQL statements directly from untrusted user input.

## Core Database Model

```text
CATEGORIES
     │
     ▼
 PRODUCTS ◄──────── SUPPLIERS
     │
     ▼
ORDER_ITEMS
     │
     ▼
  ORDERS
     │
     ▼
 CUSTOMERS

USERS
AUDIT_LOGS
LOGIN_ATTEMPTS
```

## Transaction Processing

Order creation is handled as a database transaction:

```text
Validate customer
        │
Validate products
        │
Lock required inventory rows
        │
Validate available stock
        │
Create order and order items
        │
Deduct inventory
        │
Calculate final total
        │
COMMIT
```

If a required operation fails, the transaction is rolled back. This prevents partially completed sales from leaving inventory in an inconsistent state.

## Stock Protection

Inventory-sensitive operations use database row locking where required during order processing. Stock is validated inside the transaction before the final update and commit.

## Customer Sales Model

TechZone supports two transaction types.

**Walk-in Sale** uses the protected `Walk-in Customer` system record for counter transactions where customer registration is unnecessary.

**Registered Customer Sale** associates the transaction with a specific customer record and preserves that customer's transaction history.

## Historical Data Protection

TechZone applies safe-deletion rules. Products referenced by order items, customers referenced by orders, and suppliers referenced by products are protected from normal permanent deletion. The Walk-in Customer system record receives additional protection.

## Authentication & Authorization

Passwords are stored as secure PHP password hashes. Successful authentication regenerates the session identifier and the application implements session inactivity expiration.

Database-backed login-attempt tracking provides temporary rate limiting after repeated failed authentication attempts.

The v1.0 application supports `ADMIN`, `STAFF`, and `VIEWER` roles. Authorization is enforced server-side rather than treating interface visibility as the security boundary.

The isolated cloud demo adds a `DEMO` role. It permits read access and controlled order creation while blocking administrative product, customer, and supplier modification.

## CSRF & Validation

State-changing forms use CSRF tokens. TechZone also performs reusable server-side validation for required fields, numeric values, positive quantities, identifiers, and optional email addresses. Database constraints provide an additional integrity layer.

## Audit & Logging

`AUDIT_LOGS` records important operational and security events such as authentication, logout, product/customer/supplier changes, deletion operations, and order creation.

TechZone separates application, security, and database logging. Deployment logs are stored outside the publicly accessible application tree. Sensitive credentials are not intentionally written to audit or application logs.

## Dashboard Architecture

Sales-performance calculations support `TODAY`, `7 DAYS`, `30 DAYS`, and `ALL TIME`. The selected period affects sales metrics such as revenue, sales count, units sold, average sale, category revenue, and best-selling products. Inventory-state metrics remain independent from the selected sales period.

## Search Architecture

Products, orders, customers, and suppliers support server-side Oracle search using bind variables. Searchable information includes product name, category, supplier, order ID, customer name, phone, email, and supplier location.

## Application Security Layers

```text
Authentication
      │
Login Rate Limiting
      │
Session Security
      │
Authorization
      │
CSRF Protection
      │
Input Validation
      │
Oracle Bind Variables
      │
Database Constraints
      │
Transaction Control
      │
Audit & Logging
```

---

## OCI Cloud Demonstration Architecture

The demonstration environment is isolated from the preserved v1.0 release and from the original local Oracle schema.

```text
┌───────────────────────────────┐
│        Client Browser         │
└───────────────┬───────────────┘
                │ HTTP
                ▼
┌───────────────────────────────┐
│ Oracle Cloud Infrastructure   │
│ Ampere A1 Compute             │
│ Oracle Linux 9                │
│                               │
│ Apache HTTP Server            │
│ PHP-FPM / PHP 8               │
│ OCI8                          │
│ TechZone Demo Application     │
└───────────────┬───────────────┘
                │ TCPS :1522
                ▼
┌───────────────────────────────┐
│ Oracle Autonomous Database    │
│ 26ai                          │
│                               │
│ Dedicated TechZone demo       │
│ application schema            │
└───────────────────────────────┘
```

The database is not installed on the public web VM. The application communicates with Autonomous Database through Oracle Instant Client and OCI8 over TCPS.

### Cloud Database Isolation

The cloud environment uses a dedicated TechZone schema rather than the original local development schema. The clean database installer creates the application tables, sequences, constraints, view, and required system records. A separate demo dataset populates fictional operational data.

Database network access is restricted to the application environment rather than exposing an Oracle listener as a public inbound service.

### Demo Authorization Boundary

The cloud deployment uses a restricted application-level `DEMO` role. The role can explore the operational interface and execute the controlled sales workflow, while normal administrative product/customer/supplier modification remains unavailable.

This provides a usable client demonstration without granting full administrative access to the public demo account.

### Demo Reset Model

The cloud environment includes an administrator-operated reset process outside the publicly mapped web directory. It clears demonstration transaction/business data in dependency-safe order, preserves required system records and the demo login, reloads the baseline dataset, and verifies the resulting state.

Sequence values are not required to return to their original numbers; relational integrity and baseline business data are the reset requirements.

## Cloud Host Security

The OCI demonstration host applies multiple infrastructure and web-server controls:

- Oracle Linux 9 security updates
- SELinux in enforcing mode
- firewalld enabled
- SSH public-key authentication
- SSH password authentication disabled
- Only required public web/administration listeners exposed
- Unused RPC service disabled
- Apache detailed version exposure reduced
- HTTP TRACE disabled
- Unused CGI route blocked
- Sensitive application directories not publicly mapped
- PHP version exposure disabled
- PHP file uploads disabled for the current demo workload
- Application logs stored outside the web application tree
- Automatic log rotation
- Autonomous Database access over TCPS
- Restricted database network access

The deployment intentionally does not claim HTTPS at this stage. Trusted TLS is planned as a separate domain-based deployment step; HSTS and HTTPS-only behavior should only be enabled after trusted TLS is operational.

## Oracle 26ai Validation Scope

The clean TechZone schema installs successfully on Oracle Autonomous Database 26ai. The demonstration dataset also loads successfully.

Validated application behavior in the cloud environment includes:

- PHP/OCI8 database connectivity
- Authentication
- Dashboard queries and analytics
- Inventory/customer/supplier/order read workflows
- Demo-role authorization
- Order creation
- Stock deduction
- Transaction commit behavior
- Order detail retrieval
- Demo reset process
- Service operation after host reboot

This is substantial compatibility validation of the schema and core workflow. It is not presented as exhaustive certification of every ADMIN/STAFF write operation on Oracle 26ai because the public demo intentionally restricts those operations.

## Technology Stack

| Component | Stable v1.0 Environment | OCI Demo Environment |
| --- | --- | --- |
| Backend | PHP 8.2 | PHP 8.0 / PHP-FPM |
| Database | Oracle Database 10g XE | Oracle Autonomous Database 26ai |
| Database Interface | OCI8 | OCI8 |
| Oracle Client | Local Oracle client | Oracle Instant Client |
| Web Server | Apache / XAMPP | Apache on Oracle Linux 9 |
| Frontend | HTML5 / CSS3 / JavaScript | HTML5 / CSS3 / JavaScript |
| Host | Windows | OCI Ampere A1 / Oracle Linux 9 |
| Authentication | PHP Sessions | PHP Sessions |
| Database Model | Relational | Relational |

## Current Release

**TechZone v1.0 — Inventory Operations & Sales Management System**

The v1.0 architecture remains the preserved stable baseline. The OCI demonstration deployment is maintained separately so cloud validation and future deployment work do not alter the frozen release.
