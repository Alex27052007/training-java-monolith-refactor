# Implementation Plan: Target Architecture

**Branch**: `003-target-architecture` | **Date**: 2026-07-08 | **Spec**: specs/003-target-architecture/spec.md

**Input**: Feature specification from `/specs/003-target-architecture/spec.md`

## Summary

The Big Bad Monolith is a Java EE billing application currently running on IBM WebSphere Liberty with an embedded Apache Derby database, JSP presentation layer, and tightly coupled integration points across XML configuration files, startup scripts, and direct JDBC calls from JSP pages. Phase-1 analysis produced 14 business rules (BR-001..BR-014), 13 integration inventory items (INT-001..INT-013), and 8 open questions (OQ-001..OQ-008). This plan establishes the target architecture that resolves all open questions, replaces all legacy integration points, and organises the new system into five cohesive functional modules.

The target is a TypeScript modular monolith running on Node.js 22 LTS inside a Docker OCI container, backed by PostgreSQL 16, served via Fastify 4.x. All 14 business rules are preserved in the new system; the three confirmed bugs identified during phase-1 analysis (BR-007 JSP direct DAO bypass, BR-009 User constructor inversion, BR-011 hardcoded day-31 end date) are corrected as first-class migration gates. Static credentials (INT-009, INT-010) are retired and replaced with env-var–injected secrets. The strangler-fig migration runs across six phases (Phase 0 Foundation through Phase 6 Cutover), with explicit entry/exit criteria for each phase.

Migration follows a module-by-module ownership model: the `identity` module owns user data correction (OQ-003) and seed guard (BR-013); `customer` owns customer CRUD and validation (BR-002, BR-014); `hours` owns all billable-hour logic and enforces service-layer authority (resolves OQ-001); `reporting` owns billing and monthly reports with correct month-end arithmetic (fixes BR-010, BR-011, resolves OQ-004); and `platform` owns startup, health, readiness, schema migration, and observability (BR-012). The architecture-only constraint (FR-012) means no implementation code is produced in this phase; every decision in this document includes a traceability reference to at least one BR-### or phase-1 substitution domain entry.

## Technical Context

**Language/Version**: TypeScript 5.x on Node.js 22 LTS

**Primary Dependencies**: Fastify 4.x, node-postgres (`pg`), pino, `@fastify/jwt`, bcryptjs, prom-client, node-pg-migrate

**Storage**: PostgreSQL 16

**Testing**: Vitest (unit + integration), Supertest (HTTP contract tests)

**Target Platform**: OCI container (Docker), Linux amd64

**Project Type**: Modular monolith web-service

**Performance Goals**: < 200ms p95 for all API endpoints under 100 concurrent users. Billing is not a high-throughput workload per phase-1 analysis; the goal is correctness and migratability, not extreme throughput.

**Constraints**: No code-level implementation in this phase (FR-012). Architecture-only output. All choices must trace to at least one BR-### or substitution-audit domain entry (FR-011, FR-013).

**Scale/Scope**: Single deployable unit, 5 functional modules, 4 entity tables, 13 integration replacement contracts, 6 migration phases.

## Constitution Check

`constitution.md` in this repository is a blank template; no project-specific gates are defined. Proceeding with spec requirements as the governing gate set. All gate requirements cited in the spec (FR-001..FR-013) are evaluated below.

**Gate evaluation:**

| Requirement | Description | Verdict | Notes |
|---|---|---|---|
| FR-001 | Module boundary map with BR ownership | PASS | Five modules defined; each BR assigned to exactly one module owner in this plan and in research.md |
| FR-002 | Substitution audit decisions recorded | PASS | All 8 substitution categories decided in research.md and integration-contracts.md |
| FR-003 | OQ-001..OQ-008 all resolved | PASS | Each OQ resolved in research.md with decision, rationale, and phase traceability |
| FR-004 | Target data model produced | PASS | data-model.md contains CREATE TABLE statements, field mapping, OQ-003 correction, invariant matrix |
| FR-005 | Integration contracts INT-001..INT-013 | PASS | contracts/integration-contracts.md covers all 13 entries |
| FR-006 | API surface contracts per module | PASS | contracts/api-contracts.md covers all 5 modules |
| FR-007 | Observability spec (health, ready, metrics) | PASS | Endpoints defined in api-contracts.md; pino + prom-client in Technical Context |
| FR-008 | Auth pattern decided | PASS | JWT + bcryptjs; env-var secrets; no static credentials |
| FR-009 | Migration phasing with entry/exit criteria | PASS | Six phases with entry/exit criteria in Migration Phases section below |
| FR-010 | Risk register with all OQs | PASS | Risk register in this plan covers OQ-001..OQ-008 |
| FR-011 | Traceability on every decision | PASS | All sections include BR/substitution traceability |
| FR-012 | No code-level implementation | PASS | This plan is architecture-only |
| FR-013 | No scope beyond phase-1/2 motivation | PASS | All new elements trace to BR-### or substitution domain findings |

## Project Structure

### Documentation (this feature)

```text
specs/003-target-architecture/
├── plan.md                        # This file
├── spec.md                        # Feature specification (input)
├── research.md                    # Phase 0 output — OQ resolutions, framework decisions
├── data-model.md                  # Phase 1 output — PostgreSQL schema, migration plan
├── quickstart.md                  # Validation and runbook guide
└── contracts/
    ├── integration-contracts.md   # INT-001..INT-013 replacement decisions
    └── api-contracts.md           # HTTP endpoint contracts per module
```

### Source Code (repository root — target layout)

```text
src/
├── modules/
│   ├── identity/
│   │   ├── identity.routes.ts       # POST /auth/login, user CRUD routes
│   │   ├── identity.service.ts      # User management, JWT issuance, seed guard (BR-013)
│   │   ├── identity.repository.ts   # pg queries for users table
│   │   └── identity.schema.ts       # Fastify JSON schemas for request/response
│   ├── customer/
│   │   ├── customer.routes.ts       # Customer CRUD routes
│   │   ├── customer.service.ts      # BR-002 (not-found), BR-014 (non-null/non-empty)
│   │   ├── customer.repository.ts   # pg queries for customers table
│   │   └── customer.schema.ts
│   ├── hours/
│   │   ├── hours.routes.ts          # BillingCategory + BillableHour routes
│   │   ├── hours.service.ts         # BR-001, BR-004..BR-008; service is authoritative (OQ-001)
│   │   ├── hours.repository.ts      # pg queries for billing_categories + billable_hours
│   │   └── hours.schema.ts
│   ├── reporting/
│   │   ├── reporting.routes.ts      # GET /reports/billing, GET /reports/monthly
│   │   ├── reporting.service.ts     # BR-003; last-day-of-month fix (BR-011/OQ-004)
│   │   ├── reporting.repository.ts  # pg queries; no direct JDBC (fixes BR-010)
│   │   └── reporting.schema.ts
│   └── platform/
│       ├── platform.routes.ts       # GET /health, GET /ready, GET /metrics
│       ├── startup.ts               # BR-012 — schema init + seed guard
│       └── platform.schema.ts
├── db/
│   ├── connection.ts                # pg Pool; DATABASE_URL from env (replaces INT-001..INT-004)
│   └── migrations/                  # node-pg-migrate migration files
├── config.ts                        # Env-var config (PORT, JWT_SECRET, LOG_LEVEL, NODE_ENV, etc.)
├── logger.ts                        # pino instance (JSON to stdout)
└── app.ts                           # Fastify app factory; plugin registration

tests/
├── unit/                            # Vitest unit tests per module service/repository
├── integration/                     # Vitest integration tests with test PG database
└── contract/                        # Supertest HTTP contract tests per module

Dockerfile                           # Multi-stage: build → prod image (Linux amd64)
docker-compose.yml                   # Local dev: postgres:16 + app service
.env.example                         # Documents all required env vars (no secrets)
package.json
tsconfig.json
vitest.config.ts
```

**Structure Decision**: Single-project modular monolith. Five `src/modules/` directories each own their routes, service logic, repository queries, and JSON schemas. Shared infrastructure (DB connection, config, logger) lives in `src/`. No monorepo workspace overhead is justified at this scale (13 integration contracts, 4 entity tables). Traceability: substitution domain — dated single-deployable runtime (Liberty EAR).

## Migration Phases

### Phase 0 — Foundation
**Entry**: Architecture approved (this plan merged).
**Work**: TypeScript scaffold, Dockerfile, PostgreSQL schema via node-pg-migrate, CI pipeline skeleton, env-var config, pino logger, Fastify app factory.
**Exit criteria**:
1. `docker build` succeeds, producing a runnable Linux amd64 image.
2. `GET /health` returns `{"status":"ok"}` HTTP 200.
3. `GET /ready` returns `{"status":"ready","db":"connected"}` HTTP 200 with a real PostgreSQL connection.
4. All four tables created in PG by migration runner on container start.
**Traceability**: BR-012 (startup checks datasource, initialises schema), INT-001..INT-005 (all Derby integration points must not appear in new code).

### Phase 1 — Identity
**Entry**: Phase 0 exit criteria met.
**Work**: User entity migration, `POST /auth/login`, user CRUD routes, OQ-003 data audit and correction, seed guard.
**Exit criteria**:
1. Users can authenticate via JWT; token required for all protected routes.
2. Derby `users` table exported, corrected (email/name columns swapped where values match pattern), imported to PG.
3. Human review of correction diff completed and signed off before import gate.
4. Seed guard implemented: if `users` table is non-empty, seeder exits without inserting (BR-013).
5. No static credentials exist in codebase or config files (resolves OQ-007).
**Traceability**: BR-009 (User constructor inversion), BR-013 (seed guard), OQ-003, OQ-007.

### Phase 2 — Customers
**Entry**: Phase 1 exit criteria met.
**Work**: Customer CRUD routes and service, Derby `customers` table migration.
**Exit criteria**:
1. All customer operations served from PG.
2. Derby `customers` table migrated; integrity verified row-count match.
3. BR-002 enforced: service throws 404 (not RuntimeException) for unknown customer ID.
4. BR-014 enforced: `name` and `email` required, non-empty, at API schema layer.
**Traceability**: BR-002, BR-014.

### Phase 3 — Billing Categories
**Entry**: Phase 2 exit criteria met.
**Work**: BillingCategory CRUD routes and service, Derby `billing_categories` table migration.
**Exit criteria**:
1. Categories served from PG; row-count integrity verified.
2. `hourly_rate` non-negative constraint enforced at DB and service layer.
**Traceability**: BR-001 (hourly_rate feeds bill total), BR-004 (category_id resolution).

### Phase 4 — Billable Hours Entry
**Entry**: Phase 3 exit criteria met; OQ-001 decision recorded (service-layer is authoritative).
**Work**: BillableHour entry routes, mandatory service-layer validation on all writes, dual-store window.
**Exit criteria**:
1. `POST /hours` rejects `hours <= 0` (fixes BR-005, BR-008).
2. `POST /hours` rejects future dates (BR-006).
3. Weekend date returns advisory warning field in response body, not 4xx (BR-006, OQ-005).
4. `POST /hours` validates `customer_id` and `category_id` resolve to existing rows (BR-004).
5. Legacy JSP direct DAO path blocked — Liberty is frozen for hours writes (resolves OQ-001).
6. New writes go to PG only; Derby hours table not yet retired (retained read-only for audit).
**Traceability**: BR-001, BR-004..BR-008, OQ-001, OQ-005.

### Phase 5 — Reporting
**Entry**: Phase 4 exit criteria met.
**Work**: Billing and monthly report endpoints from PG, data reconciliation audit (OQ-002), day-31 fix.
**Exit criteria**:
1. `GET /reports/billing` returns correct totals matching `SUM(hours * hourly_rate)` per customer (BR-001, BR-003).
2. `GET /reports/monthly` uses `date_trunc('month', ...) + interval '1 month' - interval '1 day'` for month-end (fixes BR-011, OQ-004).
3. No direct JDBC connections remain; all queries go through service layer (fixes BR-010).
4. Pre-migration reconciliation script run; if Derby/PG divergence found, reconciled before this phase exits (OQ-002).
5. Report results match service-layer logic exactly; smoke tests green.
**Traceability**: BR-001, BR-003, BR-010, BR-011, OQ-002, OQ-004.

### Phase 6 — Cutover
**Entry**: All Phases 0–5 exit criteria met; full smoke test suite green; Derby data fully migrated to PG.
**Work**: Route all traffic to Node.js container; decommission Liberty + Derby.
**Exit criteria**:
1. Liberty container stopped.
2. Derby data files archived (not deleted) to cold storage for 90-day retention hold.
3. DNS / load-balancer routes updated; Node.js container is sole serving unit.
4. jvm.options and related sed mutation scripts removed from repository (resolves OQ-008).
5. No references to `jdbc:derby`, `EmbeddedDriver`, `jdbc/DefaultDataSource` remain in active codebase.
**Traceability**: INT-001..INT-013 (all retired), OQ-008.

## Risk Register

| ID | Description | Severity | Phase | Decision/Mitigation |
|---|---|---|---|---|
| OQ-001 | Service-layer validation vs JSP direct DAO writes (hours.jsp:60 vs BillingService.java:89) | HIGH | Phase 4 | Service-layer is authoritative. JSP direct-DAO path is a confirmed bug (BR-007). Mitigation: service validation enforced at API layer in Phase 4; legacy JSP path blocked before Phase 5 exits. |
| OQ-002 | Derby path divergence — reports.jsp opens own JDBC connection separate from DAO path (reports.jsp:20 vs server.xml:55) | HIGH | Phase 5 | Mitigation: pre-migration data audit script; if divergence found, run reconciliation before Phase 5 cutover gate. Target uses single DATABASE_URL; one connection pool, no divergence possible. |
| OQ-003 | User constructor argument inversion — likely data corruption in production (users.jsp:15, User.java:10) | HIGH / DATA-LOSS | Phase 1 | Mandatory data audit before Derby→PG user migration. Correction script swaps email/name columns where values match email pattern in wrong column. Human review of diff required before import. Gate: import blocked until human sign-off. |
| OQ-004 | Day-31 end date hardcoded for all months — incorrect results for months with < 31 days (reports.jsp:235,236,245) | MEDIUM | Phase 5 | Replace with `date_trunc('month', ...) + interval '1 month' - interval '1 day'` PostgreSQL expression. Traceability: BR-011. |
| OQ-005 | Weekend warning policy — should warning be blocking, advisory, or suppressed? (BillingService.java:119) | MEDIUM | Phase 4 | Weekend flag returned as advisory `warnings` array field in API response body. Not a 4xx. Preserves current non-blocking service behavior. Traceability: BR-006. |
| OQ-006 | Production startup mode — Liberty-managed JNDI, embedded Derby fallback, or both? (LibertyConnectionManager.java:14,34) | MEDIUM | Phase 0 | Target has single PostgreSQL connection via DATABASE_URL env var. Startup fails hard (process exit 1) if DATABASE_URL unavailable. No fallback. Eliminates ambiguity by design. |
| OQ-007 | Static credentials in production — user1/password and app/app in server.xml:35,36,57,58 | HIGH / SECURITY | Phase 1 | No static credentials in target codebase. Auth credentials injected via JWT_SECRET, ADMIN_SEED_EMAIL, ADMIN_SEED_PASSWORD env vars only. Pre-cutover security gate: scan codebase and config for any hardcoded credential strings. Traceability: INT-009, INT-010. |
| OQ-008 | jvm.options sed mutation in liberty-start.sh:20,21,38,39 — fragile in-place mutation of JVM tuning flags | LOW | Phase 6 | Retired with Liberty. No JVM in target. Node.js runtime tuning is done via NODE_OPTIONS env var if needed. Traceability: INT-013. |

## Complexity Tracking

No constitution violations exist for this feature. Constitution.md is a blank template; no project-specific complexity gates are defined. This section is intentionally minimal.

The only notable complexity decision is the choice of a modular monolith over a microservices split. This choice is justified by scope (4 entity tables, 5 functional modules, single team) and avoids distributed system complexity that would outweigh benefits at this scale. Traceability: substitution domain — cloud-hostile operational assumptions in legacy deployment model.
