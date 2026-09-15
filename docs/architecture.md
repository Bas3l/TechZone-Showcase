# TechZone Architecture

## Overview

TechZone is a web-based inventory and sales management system built using PHP and Oracle Database.

The browser communicates with the PHP application through Apache. PHP handles authentication, authorization, validation, business logic, transaction processing, and database access through Oracle OCI8.

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

TechZone supports `ADMIN`, `STAFF`, and `VIEWER` roles. Authorization is enforced server-side rather than treating interface visibility as the security boundary.

## CSRF & Validation

State-changing forms use CSRF tokens. TechZone also performs reusable server-side validation for required fields, numeric values, positive quantities, identifiers, and optional email addresses. Database constraints provide an additional integrity layer.

## Audit & Logging

`AUDIT_LOGS` records important operational and security events such as authentication, logout, product/customer/supplier changes, deletion operations, and order creation.

TechZone separates application, security, and database logging. Production deployments can place these logs outside the publicly accessible web directory. Sensitive credentials are not intentionally written to audit or application logs.

## Dashboard Architecture

Sales-performance calculations support `TODAY`, `7 DAYS`, `30 DAYS`, and `ALL TIME`. The selected period affects sales metrics such as revenue, sales count, units sold, average sale, category revenue, and best-selling products. Inventory-state metrics remain independent from the selected sales period.

## Search Architecture

Products, orders, customers, and suppliers support server-side Oracle search using bind variables. Searchable information includes product name, category, supplier, order ID, customer name, phone, email, and supplier location.

## Security Layers

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

## Technology Stack

| Component | Technology |
| --- | --- |
| Backend | PHP 8.2 |
| Database | Oracle Database |
| Database Interface | OCI8 |
| Web Server | Apache |
| Development Environment | XAMPP / Windows |
| Frontend | HTML5 / CSS3 / JavaScript |
| Authentication | PHP Sessions |
| Database Model | Relational |

## Current Release

**TechZone v1.0 — Inventory Operations & Sales Management System**

The v1.0 architecture represents the stable baseline of TechZone. Future development can evolve independently while the v1.0 release remains preserved.
