# Research: Target Architecture

**Phase**: 0 — Outline & Research
**Date**: 2026-07-08
**Feature**: 003-target-architecture

## Purpose

This document records all Phase 0 research findings for the target architecture of the Big Bad Monolith refactor. It resolves all eight open questions (OQ-001..OQ-008) identified during phase-1 analysis, documents framework and tooling selection decisions, and provides the rationale and traceability required by FR-003 and FR-011. Every decision here directly informs the implementation plan phases and the contracts defined in `contracts/`.

---

## Research Item 1: HTTP Framework — Fastify vs Express

### Decision
Fastify 4.x

### Rationale
Fastify provides built-in JSON Schema validation for request bodies, query strings, and route parameters via its schema compiler. This directly reduces boilerplate needed to enforce BR-004 (reject unresolvable customer_id/category_id), BR-005 (reject hours <= 0), and BR-006 (reject future dates) — all three validations can be expressed declaratively as JSON Schema constraints on the route rather than as imperative guard clauses in the service layer. Fastify also offers a TypeScript-first design with first-party types, superior performance to Express under Node.js (relevant for the sub-200ms p95 goal), and active maintenance by the Fastify core team with regular LTS releases. Its plugin ecosystem (`@fastify/jwt`, `@fastify/cors`, pino integration) covers all cross-cutting requirements identified in the spec.

### Alternatives Considered
**Express 4.x**: Most widely adopted Node.js framework. Rejected because: (1) no built-in schema validation — would require manually wiring `ajv` or `zod`, adding integration surface; (2) callback-based origins require middleware boilerplate that TypeScript typing handles awkwardly; (3) no native async/await error propagation without `express-async-errors` patch; (4) pino integration requires `pino-http` shim with looser typing. Express adds complexity without providing capability benefits relevant to this use case.

**NestJS**: Enterprise-grade Node.js framework with decorators, IoC container, and module system. Rejected because: (1) heavy abstraction layer (Angular-style DI, decorators, reflection metadata) that exceeds the complexity justified by 4 entity tables and 5 modules; (2) introduces a second "monolith framework" layer on top of the chosen runtime, making it harder for future contributors to understand the data flow without framework knowledge; (3) NestJS module system partially duplicates the `src/modules/` boundary design already planned; (4) slower cold-start than Fastify which matters for container readiness checks. NestJS is appropriate for large-team enterprise apps; overkill here.

**Hono**: Modern, lightweight, edge-first framework. Rejected because: (1) ecosystem maturity for JWT + bcrypt integration is younger than Fastify's; (2) `@fastify/jwt` and `@fastify/cors` have more production miles and security audits; (3) team familiarity and documentation depth are lower. Not inherently wrong, but Fastify has a clearer advantage for this migration profile.

### Traceability
- Substitution domain: JSP presentation layer → replace-with-library (Express/Fastify route handlers + TypeScript)
- BR-004, BR-005, BR-006 (validation enforcement at route layer)
- INT-007 (`/big-bad-monolith` context root → replaced by Fastify route prefix or reverse-proxy path)

---

## Research Item 2: OQ-001 Resolution — Validation Pathway Authority

### Decision
Service-layer validation is authoritative for all BillableHour writes. The JSP direct DAO write path (hours.jsp:59,60) is a confirmed bug and must be blocked before Phase 5 exits.

### Rationale
OQ-001 asks: when hours.jsp persists directly via BillableHourDAO.save (BR-007) bypassing BillingService validation (BillingService.java:89), which path is authoritative? Evidence from phase-1 analysis shows that:
1. BillingService contains the canonical validation logic (BR-004: unresolvable IDs rejected; BR-005: hours <= 0 rejected; BR-006: future dates rejected).
2. hours.jsp also has `min="0"` in the HTML form (BR-008), meaning zero hours can pass client-side validation and reach the DAO directly — a double bypass.
3. The service layer is where business invariants live in every other module. The JSP path has no equivalent validation.

The decision is that service-layer validation defines the correct behaviour. The JSP direct-DAO path is a defect, not an intentional alternate write path. Resolution: in the target, ALL BillableHour writes go through `hours.service.ts`; the API layer (Fastify route + JSON Schema) enforces `hours > 0` as a schema constraint before the request reaches the service.

### Phase Impact
Phase 4 (Billable Hours Entry): API layer enforces `hours > 0`; service layer validates date, customer_id, category_id resolution; Liberty JSP path blocked before Phase 5 opens.

### Traceability
BR-004, BR-005, BR-006, BR-007, BR-008

---

## Research Item 3: OQ-002 Resolution — Derby Path Divergence

### Decision
A pre-migration data audit script is required before Phase 5 (Reporting) exit gate. If divergence between the Derby path used by reports.jsp and the Derby path used by the DAO/service layer is found, a one-time reconciliation script runs before Phase 5 cutover.

### Rationale
OQ-002 asks: do reports.jsp and the DAO/service layer use the same physical Derby database? Evidence: reports.jsp:20 opens `jdbc:derby:./data/bigbadmonolith;create=true` using a relative path from the JSP process working directory. The DAO/service layer uses either the Liberty JNDI datasource (INT-002, server.xml:53,55) or the embedded fallback (INT-011), both pointing to `${server.output.dir}/data/bigbadmonolith`. If the JSP process CWD differs from `${server.output.dir}`, these resolve to different physical directories, meaning reports could be reading a stale or divergent copy of the database. This is a data integrity risk.

The target architecture eliminates the problem structurally: there is one `DATABASE_URL` environment variable, one `pg.Pool`, and one PostgreSQL database. The migration audit script (described in quickstart.md) exports and compares row counts and checksums from both Derby paths. If divergence is found, reconciliation (preferring the DAO/service path as authoritative, consistent with OQ-001 decision) runs before Phase 5 reports are validated.

### Phase Impact
Phase 5 pre-condition: audit script result reviewed; reconciliation executed if needed.

### Traceability
BR-003, BR-010, INT-001, INT-002, INT-004, INT-011

---

## Research Item 4: OQ-003 Resolution — User Constructor Inversion

### Decision
The User constructor argument inversion (users.jsp:15 calls `new User(name, email)` but User.java:10 constructor signature is `(email, name)`) is a confirmed data-corruption bug. Mandatory data audit and correction is a Phase 1 gate before Derby→PG user migration.

### Rationale
Every User record created via the JSP UI has email stored in the `name` column and name stored in the `email` column. Because the legacy schema has `email VARCHAR(255) NOT NULL UNIQUE`, uniqueness is being enforced on the name value, not the email. This is a data-loss class bug. In the target PostgreSQL schema both columns are correctly typed and correctly validated.

Migration procedure:
1. Export Derby `users` table to CSV.
2. Run audit query: for each row, check whether the `email` column value matches a valid email regex. If it does not match, the columns are inverted.
3. Correction script: swap `email` and `name` column values for affected rows.
4. Human review: reviewer sees before/after diff and must sign off before import proceeds.
5. Import corrected CSV into PG `users` table.
6. Gate: Phase 1 does not close until human sign-off is recorded in the migration log.

### Phase Impact
Phase 1 (Identity): mandatory gate before user data import.

### Traceability
BR-009

---

## Research Item 5: OQ-004 Resolution — Month-End Date Logic

### Decision
Replace hardcoded day-31 end date with `date_trunc('month', target_month) + interval '1 month' - interval '1 day'` PostgreSQL expression.

### Rationale
OQ-004 asks: what is the expected behavior when a monthly report is run for a month with fewer than 31 days (e.g., February, April, June, September, November)? The legacy SQL hardcodes `YYYY-MM-31` (reports.jsp:235,236,245), which either queries a non-existent date (in databases that reject it) or wraps to the next month (in databases that coerce). Derby's behavior in this case silently produces incorrect report bounds — a query for February ends on 2024-02-31 which Derby treats as 2024-03-02, meaning February reports include 2 days of March data and March reports overlap with those same days.

The PostgreSQL expression `date_trunc('month', make_date(year, month, 1)) + interval '1 month' - interval '1 day'` correctly computes the last calendar day of any month, including leap years, without hardcoding. This is the canonical PostgreSQL idiom and is used in the target `reporting.repository.ts`.

### Phase Impact
Phase 5 (Reporting): all monthly report queries use the corrected expression.

### Traceability
BR-003, BR-011

---

## Research Item 6: OQ-005 Resolution — Weekend Warning Policy

### Decision
Weekend date flag is returned as an advisory `warnings` array in the API response body. It is NOT a 4xx rejection.

### Rationale
OQ-005 asks whether the weekend warning (BillingService.java:117,119) should be blocking, advisory, or ignored. The legacy service currently raises a warning but does not block the write — the billable-hour entry proceeds regardless. This is the correct behaviour for a billing system: weekends can be legitimately billable (on-call, emergency work), and a hard block would prevent legitimate entries. However, silently accepting weekend entries without notification is also undesirable, as it may indicate data-entry errors.

The target preserves the non-blocking behavior. The Fastify route returns HTTP 201 (created) with a `warnings` array in the response JSON: `{"id":"...","warnings":["date falls on a weekend"]}`. This allows frontend consumers to surface the warning to users without requiring a separate validation call. The decision is documented and traceable so it can be revisited if policy changes.

### Phase Impact
Phase 4 (Billable Hours Entry): `POST /hours` response schema includes optional `warnings` array.

### Traceability
BR-006

---

## Research Item 7: OQ-006 Resolution — Production Startup Mode

### Decision
The target has a single PostgreSQL connection via `DATABASE_URL` env var. Startup fails hard (process exit 1) if `DATABASE_URL` is unavailable. No fallback connection mode.

### Rationale
OQ-006 asks which startup mode was used in production: Liberty JNDI-managed, embedded Derby fallback (INT-011), or both. The legacy code (LibertyConnectionManager.java:14,34 and StartupListener.java:21) has a fallback where if the JNDI datasource init fails, it falls back to an embedded Derby connection. This means the application can silently start using a different database than the operator intended, potentially causing data divergence (related to OQ-002).

The target architecture eliminates this risk by design: if `DATABASE_URL` is not set or the PostgreSQL connection cannot be established during the startup readiness check, the process exits with code 1 and the container fails liveness/readiness probes. This is the 12-factor app (https://12factor.net/backing-services) pattern: backing services are attached resources configured via env vars; a missing resource is an error, not a graceful fallback.

### Phase Impact
Phase 0 (Foundation): `GET /ready` returns 503 if DB unreachable; startup logs emit `DATABASE_URL missing` error and exit.

### Traceability
BR-012, INT-002, INT-011, substitution domain — cloud-hostile operational assumptions

---

## Research Item 8: OQ-007 Resolution — Static Credentials

### Decision
No static credentials exist in the target codebase or configuration files. All secrets are injected via environment variables. A pre-cutover security gate scans the codebase for any hardcoded credential strings.

### Rationale
OQ-007 asks: are the default credentials `user1/password` (Liberty basic user registry, server.xml:35,36) and `app/app` (Derby JDBC credentials, server.xml:57,58) active beyond local development? The legacy code provides no evidence that these were ever changed for production deployments — they are embedded directly in committed XML configuration files. This is a critical security exposure.

The target uses:
- `JWT_SECRET` (min 32 chars, required env var): signs all JWT tokens. No default; startup fails if not set.
- `ADMIN_SEED_EMAIL` + `ADMIN_SEED_PASSWORD` (optional env vars): used only on first-run to seed an admin user; not present in normal operation.
- `DATABASE_URL`: contains the PG connection string including credentials; required env var; no default.
- No user registry in code, config files, or server XML.

The `.env.example` file documents required env var names with placeholder values (never real secrets). The `.gitignore` excludes `.env` files.

### Phase Impact
Phase 1 (Identity): static credentials retired; JWT auth implemented. Pre-cutover (Phase 6): mandatory security scan gate.

### Traceability
INT-009, INT-010, substitution domain — static Liberty user registry → replace-with-library (JWT + bcrypt)

---

## Research Item 9: OQ-008 Resolution — jvm.options Mutation

### Decision
The sed-based in-place mutation of jvm.options (liberty-start.sh:20,21,38,39) is retired. It has no equivalent in the Node.js target. If JVM-style tuning flags are needed for Node.js, they are set via the `NODE_OPTIONS` environment variable in the container environment, not via file mutation.

### Rationale
OQ-008 asks whether sed mutation of jvm.options is acceptable operational practice. It is not: it mutates a committed configuration file at runtime, making container images non-reproducible (running the container twice produces different state after the first run mutates the file). It also fails silently if the sed pattern doesn't match. In a containerised deployment, configuration is injected at runtime via env vars, not by mutating files inside the running container.

Because the target is Node.js (not JVM), jvm.options is entirely absent. There is no equivalent startup script that needs JVM heap sizes. If memory or GC tuning is required, `NODE_OPTIONS` env var (e.g., `NODE_OPTIONS=--max-old-space-size=512`) is set in the `docker run` command or Kubernetes pod spec.

### Phase Impact
Phase 6 (Cutover): liberty-start.sh and all jvm.options references removed from repository.

### Traceability
INT-013, substitution domain — startup scripts (Gradle+Liberty) → retire

---

## Research Item 10: Migration Strategy — Strangler-Fig vs Big-Bang

### Decision
Strangler-fig migration across six phases, one module per phase (after Foundation).

### Rationale
A big-bang rewrite would require freezing all development on the legacy system for the entire duration of the migration, risk data loss if any phase fails partway through, and require a single high-risk cutover event. Given the confirmed data-corruption bug (OQ-003) and the reporting path divergence risk (OQ-002), incremental migration with verifiable gates is strongly preferable.

The strangler-fig pattern allows:
1. Each module to be migrated and verified independently before the next begins.
2. Per-module rollback if a phase fails.
3. The legacy system to remain available (read-only after Phase 4) during phases 1–5.
4. Data audit gates (OQ-003 correction, OQ-002 reconciliation) to be executed with human review before they become one-way doors.

The six phases (Foundation → Identity → Customers → Billing Categories → Hours → Reporting → Cutover) follow the natural dependency order of the data model: users are referenced by billable hours, customers by billable hours and reports, categories by billable hours. No phase introduces a circular dependency.

### Traceability
BR-001..BR-014 collectively (all must be preserved in migration), OQ-002, OQ-003

---

## Research Item 11: Auth Pattern — JWT vs Session

### Decision
Stateless JWT using `@fastify/jwt` + `bcryptjs`. Env var `JWT_SECRET` (min 32 chars), `JWT_TTL` (default 3600 seconds).

### Rationale
The legacy auth is a Liberty basic user registry with static `user1/password` (INT-009) — not a session-based system. The target needs a production-grade auth mechanism that:
1. Eliminates static credentials (OQ-007 decision).
2. Works statelessly across container replicas (no sticky sessions required).
3. Integrates cleanly with Fastify's plugin system.
4. Does not require a separate session store (Redis or DB-backed sessions), which would add an infrastructure dependency not justified by the scale (small team billing tool).

JWT satisfies all four requirements. `@fastify/jwt` provides a well-tested Fastify plugin that handles token signing, verification, and expiry. `bcryptjs` handles password hashing in the `users` table (`password_hash` column, bcrypt cost factor 12). Token refresh is not in scope for this architecture phase; if needed, it can be added in a follow-on feature.

Server-side sessions (`@fastify/session` + Redis) were considered but rejected: they require a third backing service (Redis), add operational complexity, and provide no meaningful advantage for a single-team billing tool where token theft risks are low. Sessions would be appropriate if the system needed immediate token revocation (e.g., financial transaction cancellation); this use case does not require it.

### Traceability
INT-009 (static credentials retired), OQ-007, substitution domain — static Liberty user registry → replace-with-library (JWT + bcrypt)

---

## Research Item 12: Schema Migration Tooling

### Decision
`node-pg-migrate` for PostgreSQL schema migrations.

### Rationale
Schema migration tooling requirements: must work with `pg` (node-postgres) natively, must support up/down migrations, must be runnable from npm scripts without a separate server process, must produce an auditable migration history table in PostgreSQL. `node-pg-migrate` satisfies all requirements: it uses `pg` directly, writes migration history to a `pgmigrations` table, supports TypeScript migration files, and can be invoked as `npx node-pg-migrate up` or via an npm script.

**Alternatives considered:**
- **Flyway**: Java-based; would reintroduce a JVM dependency into the build toolchain. Rejected.
- **Liquibase**: Also Java-based; same issue. Rejected.
- **Prisma Migrate**: Excellent tooling but couples the migration runner to the Prisma ORM. Using Prisma ORM across the codebase would add a code-generation step and reduce direct SQL visibility in the repository layer, which is important for auditing the reporting queries that fix BR-011. Rejected in favour of direct `pg` queries with explicit SQL.
- **Knex migrations**: Reasonable alternative; slightly less ergonomic for TypeScript types. `node-pg-migrate` preferred for its more explicit migration format.

Migration files live in `src/db/migrations/`. The `platform` module's startup process runs pending migrations on container start (BR-012 gate: schema initialised before server accepts traffic).

### Traceability
BR-012 (startup initialises schema), INT-001..INT-004 (Derby schema replaced by PG migrations)

---

## Summary Table

| Item | Decision | Confidence | Traceability |
|------|----------|------------|--------------|
| 1 | HTTP Framework: Fastify 4.x | HIGH | Substitution domain (JSP→route handlers), BR-004, BR-005, BR-006 |
| 2 | OQ-001: Service-layer is authoritative for hours writes | HIGH | BR-004..BR-008 |
| 3 | OQ-002: Pre-migration audit; reconcile if Derby paths diverge | HIGH | BR-003, BR-010, INT-001, INT-002, INT-004, INT-011 |
| 4 | OQ-003: Mandatory data audit + correction + human sign-off gate | HIGH / DATA-LOSS | BR-009 |
| 5 | OQ-004: PostgreSQL `date_trunc + interval` last-day expression | HIGH | BR-003, BR-011 |
| 6 | OQ-005: Weekend warning is advisory in response body, not blocking | MEDIUM | BR-006 |
| 7 | OQ-006: Single DATABASE_URL; hard fail if unavailable; no fallback | HIGH | BR-012, INT-002, INT-011 |
| 8 | OQ-007: No static credentials; all secrets via env vars | HIGH / SECURITY | INT-009, INT-010 |
| 9 | OQ-008: jvm.options sed mutation retired with Liberty | HIGH | INT-013 |
| 10 | Migration: Strangler-fig, 6 phases | HIGH | BR-001..BR-014, OQ-002, OQ-003 |
| 11 | Auth: Stateless JWT (@fastify/jwt + bcryptjs) | HIGH | INT-009, OQ-007 |
| 12 | Schema migration: node-pg-migrate | HIGH | BR-012, INT-001..INT-004 |
