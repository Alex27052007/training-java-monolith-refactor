# Business Rules (Reverse Engineered)

## Scope and Method

These rules were derived from source code and runtime/config artifacts in this repository. Each rule includes code evidence and a concrete example.

## Rule Catalog

### BR-001: Customer bill totals are calculated from billable hours joined to category rates
- Rule statement: Customer billing totals are computed as the sum of `(hour.hours * category.hourly_rate)` for all hours found by customer ID.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:24`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:38`
- Concrete example: If a customer has 8.50 hours in Development at 150.00 and 4.00 hours in Consulting at 200.00, line totals are 1275.00 and 800.00 and bill total is 2075.00.
- Confidence: Observed

### BR-002: Billing generation fails hard when customer ID is unknown
- Rule statement: Billing service throws a runtime error if the requested customer ID does not exist.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:26`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:28`
- Concrete example: Calling customer bill generation with an ID absent from `customers` raises `RuntimeException("Customer not found")`.
- Confidence: Observed

### BR-003: Monthly service report includes only entries matching requested year and month
- Rule statement: Monthly report in service logic filters billable hours by exact year and month from `date_logged`.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:54`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:63`
- Concrete example: For `(year=2026, month=7)`, a row dated 2026-06-30 is excluded and a row dated 2026-07-01 is included.
- Confidence: Observed

### BR-004: Billable-hour validation requires valid customer/category references
- Rule statement: Validation marks entries invalid if customer ID or category ID cannot be resolved.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:94`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:103`
- Concrete example: A billable-hour payload with non-existent `categoryId` gets `Invalid category ID.` in validation output.
- Confidence: Observed

### BR-005: Billable-hour validation requires hours > 0
- Rule statement: Service validation rejects null, zero, and negative hours.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:111`
- Concrete example: `hours = 0.00` returns `Hours must be greater than zero.`
- Confidence: Observed

### BR-006: Future dates are rejected, weekend dates are warned (not blocked) by service validation
- Rule statement: Service validation rejects future dates and flags weekend dates as warning text.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:117`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:119`
- Concrete example: A Saturday date in the past can pass with warning; a date tomorrow is rejected.
- Confidence: Observed

### BR-007: Hours JSP persists billable hours directly through DAO
- Rule statement: The hours entry UI writes directly to `BillableHourDAO.save` after local form checks.
- Evidence:
  - `src/main/webapp/hours.jsp:59`
  - `src/main/webapp/hours.jsp:60`
- Concrete example: Submitting `hours.jsp` with non-empty required fields calls DAO save without calling `BillingService.validateBillableHour`.
- Confidence: Observed

### BR-008: Hours UI permits zero-hour entry client-side
- Rule statement: HTML validation for hours uses `min="0"`, allowing zero at form level.
- Evidence:
  - `src/main/webapp/hours.jsp:177`
- Concrete example: Browser-side validation accepts `0` as input, while service-layer logic would reject it.
- Confidence: Observed

### BR-009: Users UI creates User objects with constructor arguments in name,email order
- Rule statement: Users page constructs `new User(name, email)` while entity constructor signature is `(email, name)`.
- Evidence:
  - `src/main/webapp/users.jsp:15`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/entity/User.java:10`
- Concrete example: Form input `name=Alice`, `email=alice@example.com` maps to User(email="Alice", name="alice@example.com") before persistence.
- Confidence: Observed

### BR-010: Reports JSP bypasses DAO/service and opens direct JDBC connections
- Rule statement: Reports UI uses direct Derby driver loading and `DriverManager.getConnection` against a hard-coded DB URL.
- Evidence:
  - `src/main/webapp/reports.jsp:20`
  - `src/main/webapp/reports.jsp:136`
  - `src/main/webapp/reports.jsp:137`
- Concrete example: Selecting customer report executes SQL directly in JSP, independent of `BillingService` report methods.
- Confidence: Observed

### BR-011: Reports monthly query uses fixed end-of-month day 31 string
- Rule statement: Monthly report SQL range is built using `startDate=YYYY-MM-01` and `endDate=YYYY-MM-31`.
- Evidence:
  - `src/main/webapp/reports.jsp:235`
  - `src/main/webapp/reports.jsp:236`
  - `src/main/webapp/reports.jsp:245`
- Concrete example: February range is emitted as `YYYY-02-01` to `YYYY-02-31` and then passed to `java.sql.Date.valueOf`.
- Confidence: Observed

### BR-012: Startup initializes schema mode and sample data on application context initialization
- Rule statement: On startup, the app checks Liberty datasource availability, initializes schema when Liberty datasource exists, and then initializes sample data.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/StartupListener.java:14`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/StartupListener.java:18`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/StartupListener.java:26`
- Concrete example: First run with empty `users` table leads to sample users/customers/categories/hours insertion.
- Confidence: Observed

### BR-013: Sample data seeding is skipped if at least one user exists
- Rule statement: Sample-data initializer exits early when `userDAO.findAll()` returns non-empty.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/DataInitializationService.java:26`
- Concrete example: If one user row exists, no sample customers/categories/hours are created by initializer.
- Confidence: Observed

### BR-014: Customer DAO applies defensive validation before insert/update
- Rule statement: Customer save/update operations enforce non-null and non-empty customer name/email in code.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/CustomerDAO.java:14`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/CustomerDAO.java:17`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/CustomerDAO.java:20`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/CustomerDAO.java:83`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/CustomerDAO.java:86`
- Concrete example: Calling `customerDAO.save` with blank email throws `IllegalArgumentException` before SQL execution.
- Confidence: Observed

## Coverage Notes

- Rules above cover service, DAO, startup, and JSP-reporting paths.
- Unresolved precedence conflicts are listed in `open-questions.md`.
