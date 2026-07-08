# Data Model: Legacy Monolith Rediscovery

## 1) Domain Entities Observed in the Monolith

### User
- Fields:
  - `id: BIGINT` (PK, identity)
  - `email: VARCHAR(255)` (NOT NULL, UNIQUE)
  - `name: VARCHAR(255)` (NOT NULL)
- Source of truth:
  - Schema creation in DAO connection bootstrap (`ConnectionManager.java:32`).
  - Java entity in `entity/User.java` (`User.java:5`, `User.java:10`).
- Invariants:
  - Email uniqueness enforced at DB level.
  - Non-null email/name enforced at DB level.
  - No explicit defensive validation in `UserDAO.save` beyond database failure handling.

### Customer
- Fields:
  - `id: BIGINT` (PK, identity)
  - `name: VARCHAR(255)` (NOT NULL)
  - `email: VARCHAR(255)` (NOT NULL)
  - `address: VARCHAR(500)` (nullable)
  - `created_at: TIMESTAMP` (NOT NULL)
- Source of truth:
  - Schema creation in DAO connection bootstrap (`ConnectionManager.java:42`).
  - Java entity in `entity/Customer.java`.
- Invariants:
  - Non-null name/email at DB level.
  - Defensive checks in `CustomerDAO.save/update` require non-null/non-empty name/email.
  - `created_at` defaulted in code when missing.

### BillingCategory
- Fields:
  - `id: BIGINT` (PK, identity)
  - `name: VARCHAR(255)` (NOT NULL)
  - `description: VARCHAR(500)` (nullable)
  - `hourly_rate: DECIMAL(10,2)` (NOT NULL)
- Source of truth:
  - Schema creation + entity class (`ConnectionManager.java:54`).
- Invariants:
  - Non-null rate enforced in schema.
  - Category object null-check on save.
  - Positive/zero rate is not constrained at DB level; UI forms use client-side `min=0` but backend does not enforce strictly.

### BillableHour
- Fields:
  - `id: BIGINT` (PK, identity)
  - `customer_id: BIGINT` (FK -> customers.id, NOT NULL)
  - `user_id: BIGINT` (FK -> users.id, NOT NULL)
  - `category_id: BIGINT` (FK -> billing_categories.id, NOT NULL)
  - `hours: DECIMAL(8,2)` (NOT NULL)
  - `note: VARCHAR(1000)` (nullable)
  - `date_logged: DATE` (NOT NULL)
  - `created_at: TIMESTAMP` (NOT NULL)
- Source of truth:
  - Schema creation + entity class (`ConnectionManager.java:65`).
- Invariants:
  - FK integrity enforced at DB level.
  - Non-null critical fields enforced at DB level.
  - Service-level validation requires `hours > 0`, `date_logged <= today`, and flags weekend entries as warning.
  - JSP logging path currently saves directly through DAO and does not call service validation.

## 2) Relationships

### Customer (1) -> (N) BillableHour
- Evidence: `ConnectionManager.java:75` and reporting join `reports.jsp:243`.
- Optionality:
  - A billable hour must have one customer.
  - A customer can exist without any billable hours.

### User (1) -> (N) BillableHour
- Evidence: `ConnectionManager.java:76` and reporting join `reports.jsp:154`.
- Optionality:
  - A billable hour must have one user.
  - A user can exist without any logged hours.

### BillingCategory (1) -> (N) BillableHour
- Evidence: `ConnectionManager.java:77` and reporting join `reports.jsp:155`.
- Optionality:
  - A billable hour must have one category.
  - A category can exist without any usage.

## 3) Derived/Computed Data Concepts

### Bill Line Amount
- Formula: `line_amount = hours * hourly_rate`.
- Used in billing and reports calculations.
- Evidence: `BillingService.java:38`, `reports.jsp:151`.

### Customer Bill Totals
- Aggregations:
  - `totalHours = SUM(hours)`
  - `totalAmount = SUM(hours * category.hourly_rate)`
- Evidence: `BillingService.java:45`, `BillingService.java:39`, `reports.jsp:151`.

### Monthly Totals
- Service path uses date component comparison.
- Reporting JSP path uses explicit start/end date bounds.
- Evidence: `BillingService.java:63`, `reports.jsp:235`, `reports.jsp:236`, `reports.jsp:245`.

## 4) Invariant Matrix (Enforcement Source)

- `users.email unique`: DB Constraint
- `users.email/name not null`: DB Constraint
- `customers.name/email not empty`: Defensive Code + DB Constraint
- `customers.created_at present`: DB Constraint + Defensive Code defaulting
- `billing_categories.hourly_rate present`: DB Constraint
- `billable_hours FK integrity`: DB Constraint
- `billable_hours.hours > 0`: Defensive Code (service path)
- `billable_hours.date_logged not future`: Defensive Code (service path)
- `billable_hours.weekend logging`: Defensive Code warning (non-blocking)

Evidence references for matrix:
- `ConnectionManager.java:32`, `ConnectionManager.java:42`, `ConnectionManager.java:54`, `ConnectionManager.java:65`
- `CustomerDAO.java:17`, `CustomerDAO.java:20`
- `BillingService.java:111`, `BillingService.java:117`, `BillingService.java:119`
- `hours.jsp:177`, `hours.jsp:60` (UI/DAO path contrast)

## 5) Feature Artifact Entities (for rediscovery package)

The rediscovery output itself uses structured documentation entities:
- `BusinessRuleRecord`
- `DataModelFact`
- `IntegrationEntry`
- `OpenQuestion`
- `EvidenceReference`

These are defined by specification requirements and contract artifacts, not persisted in the monolith database.
