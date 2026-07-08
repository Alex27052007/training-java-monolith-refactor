# Data Model: Target Architecture

**Phase**: 1 — Design
**Date**: 2026-07-08
**Feature**: 003-target-architecture

## Overview

The target data model replaces the Apache Derby schema with a PostgreSQL 16 schema. The four legacy entities (User, Customer, BillingCategory, BillableHour) are preserved with corrections for the confirmed bugs identified in phase-1 analysis. The most significant change is the addition of `password_hash` to the `users` table (replacing the Liberty basic user registry) and the correction of the `email`/`name` column inversion bug (BR-009, OQ-003).

All tables use `BIGINT GENERATED ALWAYS AS IDENTITY` as the primary key strategy, replacing Derby's auto-increment to align with PostgreSQL idioms. `created_at` columns default to `now()` so application code does not need to supply them. Foreign key constraints are enforced at the database level. Check constraints enforce domain invariants that are independently enforced at the application layer (see Invariant Matrix below).

---

## Target PostgreSQL Schema

```sql
-- ============================================================
-- Table: users
-- Owns: identity module
-- BR coverage: BR-009 (field order corrected), BR-013 (seed guard in app)
-- ============================================================
CREATE TABLE users (
    id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email         VARCHAR(255) NOT NULL,
    name          VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at    TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
    CONSTRAINT uq_users_email UNIQUE (email),
    CONSTRAINT chk_users_email_nonempty CHECK (length(trim(email)) > 0),
    CONSTRAINT chk_users_name_nonempty  CHECK (length(trim(name))  > 0)
);

-- ============================================================
-- Table: customers
-- Owns: customer module
-- BR coverage: BR-002 (not-found error), BR-014 (non-null/non-empty)
-- ============================================================
CREATE TABLE customers (
    id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       VARCHAR(255)  NOT NULL,
    email      VARCHAR(255)  NOT NULL,
    address    VARCHAR(500),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
    CONSTRAINT chk_customers_name_nonempty  CHECK (length(trim(name))  > 0),
    CONSTRAINT chk_customers_email_nonempty CHECK (length(trim(email)) > 0)
);

-- ============================================================
-- Table: billing_categories
-- Owns: hours module (category sub-resource)
-- BR coverage: BR-001 (hourly_rate feeds bill total), BR-004 (category resolution)
-- ============================================================
CREATE TABLE billing_categories (
    id           BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name         VARCHAR(255)    NOT NULL,
    description  VARCHAR(500),
    hourly_rate  DECIMAL(10, 2)  NOT NULL,
    created_at   TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
    CONSTRAINT uq_billing_categories_name UNIQUE (name),
    CONSTRAINT chk_billing_categories_name_nonempty CHECK (length(trim(name)) > 0),
    CONSTRAINT chk_billing_categories_rate_positive CHECK (hourly_rate >= 0)
);

-- ============================================================
-- Table: billable_hours
-- Owns: hours module
-- BR coverage: BR-001 (hours*rate), BR-004 (FK resolution), BR-005 (hours > 0),
--              BR-006 (date not future; weekend advisory), BR-007 (no bypass)
-- ============================================================
CREATE TABLE billable_hours (
    id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT         NOT NULL REFERENCES customers(id)          ON DELETE RESTRICT,
    user_id     BIGINT         NOT NULL REFERENCES users(id)              ON DELETE RESTRICT,
    category_id BIGINT         NOT NULL REFERENCES billing_categories(id) ON DELETE RESTRICT,
    hours       DECIMAL(8, 2)  NOT NULL,
    note        VARCHAR(1000),
    date_logged DATE           NOT NULL,
    created_at  TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
    CONSTRAINT chk_billable_hours_positive  CHECK (hours > 0),
    CONSTRAINT chk_billable_hours_not_future CHECK (date_logged <= CURRENT_DATE)
);

-- ============================================================
-- Indexes (query-pattern driven)
-- ============================================================
CREATE INDEX idx_billable_hours_customer_id ON billable_hours(customer_id);
CREATE INDEX idx_billable_hours_date_logged  ON billable_hours(date_logged);
CREATE INDEX idx_billable_hours_user_id      ON billable_hours(user_id);
```

---

## Field-by-Field Mapping: Legacy → Target

### users

| Legacy Field | Legacy Type | Target Field | Target Type | Change Notes |
|---|---|---|---|---|
| `id` | BIGINT PK | `id` | BIGINT GENERATED ALWAYS AS IDENTITY PK | PG identity column |
| `email` | VARCHAR(255) NOT NULL UNIQUE | `email` | VARCHAR(255) NOT NULL UNIQUE | **OQ-003: legacy rows likely contain name values here — see correction below** |
| `name` | VARCHAR(255) NOT NULL | `name` | VARCHAR(255) NOT NULL | **OQ-003: legacy rows likely contain email values here** |
| *(not present)* | — | `password_hash` | VARCHAR(255) NOT NULL | New: bcrypt hash replaces Liberty user registry (INT-009) |
| *(not present)* | — | `created_at` | TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now() | New: added for audit trail |

### customers

| Legacy Field | Legacy Type | Target Field | Target Type | Change Notes |
|---|---|---|---|---|
| `id` | BIGINT PK | `id` | BIGINT GENERATED ALWAYS AS IDENTITY PK | PG identity column |
| `name` | VARCHAR(255) NOT NULL | `name` | VARCHAR(255) NOT NULL | No change; CHECK constraint added |
| `email` | VARCHAR(255) NOT NULL | `email` | VARCHAR(255) NOT NULL | No change; CHECK constraint added |
| `address` | VARCHAR(500) | `address` | VARCHAR(500) | No change; nullable preserved |
| `created_at` | TIMESTAMP NOT NULL | `created_at` | TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now() | Timezone-aware; DEFAULT added |

### billing_categories

| Legacy Field | Legacy Type | Target Field | Target Type | Change Notes |
|---|---|---|---|---|
| `id` | BIGINT PK | `id` | BIGINT GENERATED ALWAYS AS IDENTITY PK | PG identity column |
| `name` | VARCHAR(255) NOT NULL | `name` | VARCHAR(255) NOT NULL | UNIQUE constraint added |
| `description` | VARCHAR(500) | `description` | VARCHAR(500) | No change; nullable preserved |
| `hourly_rate` | DECIMAL(10,2) NOT NULL | `hourly_rate` | DECIMAL(10,2) NOT NULL | CHECK `>= 0` added |
| *(not present)* | — | `created_at` | TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now() | New: audit trail |

### billable_hours

| Legacy Field | Legacy Type | Target Field | Target Type | Change Notes |
|---|---|---|---|---|
| `id` | BIGINT PK | `id` | BIGINT GENERATED ALWAYS AS IDENTITY PK | PG identity column |
| `customer_id` | BIGINT NOT NULL FK | `customer_id` | BIGINT NOT NULL FK REFERENCES customers(id) ON DELETE RESTRICT | RESTRICT prevents orphan billing data |
| `user_id` | BIGINT NOT NULL FK | `user_id` | BIGINT NOT NULL FK REFERENCES users(id) ON DELETE RESTRICT | RESTRICT added |
| `category_id` | BIGINT NOT NULL FK | `category_id` | BIGINT NOT NULL FK REFERENCES billing_categories(id) ON DELETE RESTRICT | RESTRICT added |
| `hours` | DECIMAL(8,2) NOT NULL | `hours` | DECIMAL(8,2) NOT NULL | CHECK `hours > 0` added (BR-005) |
| `note` | VARCHAR(1000) | `note` | VARCHAR(1000) | No change; nullable |
| `date_logged` | DATE NOT NULL | `date_logged` | DATE NOT NULL | CHECK `<= CURRENT_DATE` added (BR-006) |
| `created_at` | TIMESTAMP NOT NULL | `created_at` | TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now() | Timezone-aware; DEFAULT added |

---

## OQ-003 Correction: User Email/Name Inversion

### Problem Description
`users.jsp:15` calls `new User(name, email)` but `User.java:10` constructor signature is `(email, name)`. Every User record created through the JSP UI has the `email` column populated with the person's name, and the `name` column populated with the person's email address. The UNIQUE constraint on `email` was therefore enforcing uniqueness on names, not email addresses.

### Detection Query (run against Derby export)
```sql
-- Identify rows where the 'email' column does NOT contain an @ character
-- (i.e., it is probably a name value, not an email address)
SELECT id, email AS suspected_name, name AS suspected_email
FROM   users
WHERE  email NOT LIKE '%@%'
   OR  name  LIKE '%@%';
```

### Correction Migration Query (run against Derby export CSV before PG import)
```sql
-- Swap email and name columns for rows where they appear inverted
-- Run against the Derby export staging table before importing to PG
UPDATE users_staging
SET    email = name,
       name  = email
WHERE  email NOT LIKE '%@%'
   OR  name  LIKE '%@%';
```

### Migration Gate
1. Export Derby `users` to `users_staging` table.
2. Run detection query. Record count of affected rows.
3. Run correction query on `users_staging`.
4. Human reviewer inspects a sample of corrected rows and verifies that `email` column now contains email-format values and `name` column contains human-readable name values.
5. Reviewer signs off in the migration log (logged as a SQL comment or separate audit record).
6. Only after sign-off: import `users_staging` into PG `users` table (with `password_hash` set to a temporary bcrypt hash of a random string; users must reset password via `ADMIN_SEED_PASSWORD` flow).

### Traceability
BR-009

---

## Entity Relationships

```
users
  │
  │  1:N (user_id)
  ▼
billable_hours ◄──── 1:N (customer_id) ──── customers
      │
      │  N:1 (category_id)
      ▼
billing_categories
```

**Narrative:**
- A `user` logs zero or more `billable_hours` entries.
- A `customer` is billed for zero or more `billable_hours` entries.
- A `billing_category` (with its `hourly_rate`) classifies zero or more `billable_hours` entries.
- `billable_hours` is the central join table connecting users, customers, and categories.
- Deleting a `user`, `customer`, or `billing_category` is restricted (`ON DELETE RESTRICT`) if they have associated billable_hours rows, preventing orphaned billing data.

---

## Invariant Matrix

| Invariant | Description | DB Constraint | App Layer | Both |
|---|---|---|---|---|
| `users.email` unique | Each email address belongs to one user account | ✓ `UNIQUE` constraint | ✓ Service returns 409 on duplicate | Both |
| `users.email` non-empty | Email cannot be blank string | ✓ `CHECK trim > 0` | ✓ JSON Schema `minLength: 1` | Both |
| `users.name` non-empty | Name cannot be blank string | ✓ `CHECK trim > 0` | ✓ JSON Schema `minLength: 1` | Both |
| `customers.name` non-empty | Customer name cannot be blank (BR-014) | ✓ `CHECK trim > 0` | ✓ JSON Schema `minLength: 1` | Both |
| `customers.email` non-empty | Customer email cannot be blank (BR-014) | ✓ `CHECK trim > 0` | ✓ JSON Schema `minLength: 1` | Both |
| `billing_categories.hourly_rate >= 0` | Rate cannot be negative | ✓ `CHECK hourly_rate >= 0` | ✓ JSON Schema `minimum: 0` | Both |
| `billing_categories.name` unique | Category names must be distinct | ✓ `UNIQUE` constraint | ✓ Service returns 409 on duplicate | Both |
| `billable_hours.hours > 0` | Hours must be strictly positive (BR-005) | ✓ `CHECK hours > 0` | ✓ JSON Schema `exclusiveMinimum: 0` | Both |
| `billable_hours.date_logged <= today` | Cannot log hours for future dates (BR-006) | ✓ `CHECK date_logged <= CURRENT_DATE` | ✓ Service validates before insert | Both |
| `billable_hours.customer_id` resolves | Customer must exist (BR-004) | ✓ FK `REFERENCES customers(id)` | ✓ Service pre-validates and returns 422 | Both |
| `billable_hours.user_id` resolves | User must exist (BR-004) | ✓ FK `REFERENCES users(id)` | ✓ Service pre-validates and returns 422 | Both |
| `billable_hours.category_id` resolves | Category must exist (BR-004) | ✓ FK `REFERENCES billing_categories(id)` | ✓ Service pre-validates and returns 422 | Both |
| Weekend advisory (BR-006) | Weekend date produces warning, not error | ✗ No DB constraint (non-blocking) | ✓ Service returns `warnings` array in response | App only |
| Seed guard (BR-013) | Seeder skips if users table non-empty | ✗ Not a constraint | ✓ Startup service checks `COUNT(*)` before seeding | App only |

---

## Migration Sequence

### Step 1: Export Legacy Derby Data
```
1. Stop the Liberty server (or freeze writes via maintenance mode).
2. Run Derby `ij` tool to export each table to CSV:
   CONNECT 'jdbc:derby:./data/bigbadmonolith';
   CALL SYSCS_UTIL.SYSCS_EXPORT_TABLE(null,'USERS','users_export.csv',null,null,null);
   CALL SYSCS_UTIL.SYSCS_EXPORT_TABLE(null,'CUSTOMERS','customers_export.csv',null,null,null);
   CALL SYSCS_UTIL.SYSCS_EXPORT_TABLE(null,'BILLING_CATEGORIES','categories_export.csv',null,null,null);
   CALL SYSCS_UTIL.SYSCS_EXPORT_TABLE(null,'BILLABLE_HOURS','hours_export.csv',null,null,null);
3. Record row counts for each export: users=N, customers=N, billing_categories=N, billable_hours=N.
```

### Step 2: Audit OQ-003 User Inversion
```
4. Load users_export.csv into a staging table in PostgreSQL (or local SQLite).
5. Run OQ-003 detection query (see OQ-003 section above).
6. Record count of inverted rows. If count > 0, run correction query.
7. Human reviewer inspects diff and signs off in migration log.
```

### Step 3: Import to PostgreSQL (in dependency order)
```
8.  Import billing_categories first (no FK dependencies).
9.  Import customers (no FK dependencies).
10. Import users (corrected; set temporary password_hash for all rows).
11. Import billable_hours last (all FKs must resolve).
12. Verify row counts match export counts from Step 1.
```

### Step 4: Validate
```
13. Run integrity check: SELECT COUNT(*) FROM billable_hours
    LEFT JOIN customers ON billable_hours.customer_id = customers.id
    WHERE customers.id IS NULL;
    Expected: 0 rows.
14. Run same check for user_id and category_id.
15. Run sample billing report query against PG and compare totals to legacy Derby output.
16. Gate: Phase 1 (for users), Phase 2 (for customers), Phase 3 (for categories), Phase 4 (for hours).
```

---

## Seed Data Specification

Governed by BR-012 (startup checks datasource, initialises schema, then seeds sample data) and BR-013 (seeder exits early if `users` table non-empty).

### Seed Guard Logic
```
On startup:
1. Run pending migrations (node-pg-migrate).
2. Check: SELECT COUNT(*) FROM users > 0?
   - YES: skip all seeding (BR-013). Log: "Seed skipped: users table is non-empty."
   - NO: proceed to seed.
3. If ADMIN_SEED_EMAIL and ADMIN_SEED_PASSWORD env vars are set:
   - Insert one admin user with bcrypt-hashed password.
4. If NODE_ENV = 'development' and seed flag set:
   - Insert sample customers, billing categories, and billable hours for local dev testing.
```

### Sample Seed Records (development only)

**users** (1 admin, after OQ-007 credential requirement):
- email: from `ADMIN_SEED_EMAIL` env var
- name: "Admin User"
- password_hash: bcrypt(ADMIN_SEED_PASSWORD, cost=12)

**customers** (2 sample records):
- Acme Corp, acme@example.com
- TechStart Ltd, tech@example.com

**billing_categories** (3 sample records):
- Development, hourly_rate: 150.00
- Consulting, hourly_rate: 200.00
- Support, hourly_rate: 100.00

**billable_hours** (sample entries referencing seeded users/customers/categories):
- Entries covering the current month for OQ-004 month-end testing
- Entries that fall on weekends to exercise BR-006 advisory behavior
