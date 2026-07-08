# Integration Contracts: Target Architecture

**Phase**: 1 — Design
**Date**: 2026-07-08
**Feature**: 003-target-architecture

## Overview

This document specifies the replacement decision for each of the 13 integration inventory items identified in the phase-1 analysis (INT-001..INT-013). For each item the legacy element is described, a substitution decision is recorded (one of: `replace-with-platform`, `replace-with-library`, or `retire`), and the exact replacement mechanism is specified including environment variable names and operational semantics.

---

### INT-001 — Embedded Derby JDBC URL

**Legacy**: Hardcoded connection string `jdbc:derby:./data/bigbadmonolith;create=true` in `ConnectionManager.java:9`. This URL uses a relative path from the JVM working directory, making the database location process-working-directory–dependent and non-portable across deployment environments.

**Proposal**: replace-with-platform

**Replacement**: PostgreSQL 16 accessed via `pg.Pool` (node-postgres) using the `DATABASE_URL` environment variable. The connection string format is `postgresql://user:password@host:port/database`. The pool is initialised once in `src/db/connection.ts` and shared across all module repositories. No hardcoded connection strings anywhere in the application code.

**Configuration surface**:
- `DATABASE_URL` — PostgreSQL connection string. Required. No default. Application exits with code 1 at startup if not set.

**Retirement rationale**: Apache Derby is a single-process embedded database with no network access mode suitable for multi-replica deployment. It cannot be used as a shared backing service in a containerised environment. PostgreSQL 16 provides ACID guarantees, concurrent access, window functions (needed for reporting), and is a standard cloud-managed service.

**Traceability**: Substitution domain — unfit data stores (Apache Derby embedded DB → PostgreSQL). BR-012 (startup initialises schema via migrations).

---

### INT-002 — Liberty JNDI Datasource

**Legacy**: Liberty JNDI lookup `jdbc/DefaultDataSource` in `LibertyConnectionManager.java:11,25` and declared in `server.xml:53`. This is a Java EE container-managed datasource that only exists within the Liberty runtime. Application code that uses JNDI cannot run outside of a Liberty container.

**Proposal**: replace-with-platform

**Replacement**: `pg.Pool` instance created from `DATABASE_URL` env var in `src/db/connection.ts`. The pool is a module-level singleton exported and imported by repository files in each module. No JNDI, no container-managed connection factory.

**Configuration surface**:
- `DATABASE_URL` — same as INT-001.

**Retirement rationale**: JNDI is a Java EE mechanism with no Node.js equivalent or benefit. The target runtime has no application server; connection management is the responsibility of the `pg` pool configured entirely from env vars.

**Traceability**: Substitution domain — Liberty Java EE runtime → Node.js LTS. INT-001 (same ultimate replacement).

---

### INT-003 — Liberty Derby Filesystem Path

**Legacy**: Liberty server.xml defines the Derby database at `${server.output.dir}/data/bigbadmonolith` (server.xml:55, bootstrap.properties:12). This path resolves to a location inside the Liberty server output directory, coupling the database location to the Liberty installation path.

**Proposal**: retire

**Replacement**: Not applicable — the path no longer exists. The database is PostgreSQL running as a separate container/service; its data directory is managed by the PostgreSQL container, not by the application server.

**Configuration surface**: None. PostgreSQL container manages its own data directory (`PGDATA`). The application connects only via `DATABASE_URL`.

**Retirement rationale**: Liberty server output directories have no equivalent in a Node.js container deployment. Filesystem-coupled database paths are incompatible with containerised, volume-mounted PostgreSQL. The path is retired alongside the Liberty runtime.

**Traceability**: Substitution domain — Liberty Java EE runtime → retire. INT-001, INT-002.

---

### INT-004 — JSP Direct Derby JDBC Path

**Legacy**: `reports.jsp:20` opens its own JDBC connection to `jdbc:derby:./data/bigbadmonolith;create=true` — a separate, direct Derby connection independent of the DAO/service layer connections. This creates the divergence risk identified in OQ-002.

**Proposal**: retire

**Replacement**: All database access in the target goes through the module repository layer (`reporting.repository.ts`). There are no direct database connections in route handlers or view templates. The `reporting` module uses the shared `pg.Pool` from `src/db/connection.ts`.

**Configuration surface**: None beyond the shared `DATABASE_URL`.

**Retirement rationale**: Direct JDBC in JSP is a confirmed architectural defect (BR-010). It bypasses service-layer logic, creates potential data divergence (OQ-002), and cannot coexist with a Node.js runtime. Retired in Phase 5 as part of reporting module migration.

**Traceability**: BR-010 (reports.jsp direct JDBC), OQ-002 (Derby path divergence).

---

### INT-005 — Derby Embedded JDBC Driver

**Legacy**: `reports.jsp:136,232,310` explicitly loads `org.apache.derby.jdbc.EmbeddedDriver` via `Class.forName(...)`. This is a Java-only, Derby-specific mechanism for registering a JDBC driver.

**Proposal**: retire

**Replacement**: Not applicable. Node.js uses `pg` (node-postgres) which is a native PostgreSQL client library. There is no concept of a JDBC driver class in the target runtime. The `pg` module is registered as an npm dependency in `package.json`.

**Configuration surface**: None. `pg` is loaded automatically when imported.

**Retirement rationale**: JDBC and its class-loading mechanism are Java-only. They do not exist in Node.js. Retired with the Liberty/JSP layer.

**Traceability**: Substitution domain — JSP presentation layer → retire. INT-004 (same Derby connection path).

---

### INT-006 — Hard-coded Liberty Ports

**Legacy**: Liberty server.xml hardcodes `httpEndpoint port="9080"` and `httpsPort="9443"` (server.xml:17,18). These values are also duplicated in bootstrap.properties:4,5. Port selection requires editing committed XML files.

**Proposal**: replace-with-platform

**Replacement**: Fastify listens on the port specified by the `PORT` environment variable, defaulting to `3000` if not set. The `docker run` or `docker-compose` port mapping controls the external port binding. No port values are committed to source code.

**Configuration surface**:
- `PORT` — HTTP listen port for the Fastify server. Default: `3000`. Set in container environment or `docker-compose.yml` for local dev.

**Retirement rationale**: Hardcoded ports in configuration files prevent environment-specific deployment without source changes. The 12-factor app pattern mandates port binding via environment variables. The target has no HTTPS termination inside the container (HTTPS is handled by a reverse proxy or load balancer in production).

**Traceability**: Substitution domain — hard-coded config in server.xml/bootstrap.properties → replace-with-platform (env vars, 12-factor). INT-008 (startup scripts print fixed URL).

---

### INT-007 — Liberty Context Root

**Legacy**: Liberty server.xml defines `contextRoot="/big-bad-monolith"` (server.xml:26). All JSP application URLs are prefixed with this path.

**Proposal**: retire

**Replacement**: In the target, the Fastify app serves all routes at the root `/` path (or a configurable `APP_PREFIX` env var if a path prefix is required by a reverse proxy). The default deployment does not use a path prefix. If the deployment environment requires `/big-bad-monolith` for backward compatibility during a blue-green cutover, the `APP_PREFIX=/big-bad-monolith` env var can be set and applied as a Fastify route prefix.

**Configuration surface**:
- `APP_PREFIX` — optional route path prefix. Default: `""` (empty, routes at root). Only set if needed for reverse-proxy compatibility during cutover.

**Retirement rationale**: Context roots are a Java EE / Liberty concept. Fastify routes are defined with explicit paths; a prefix plugin (`@fastify/prefix` or Fastify's native `prefix` option on plugin registration) handles any prefix requirement without committing it to source.

**Traceability**: Substitution domain — Liberty Java EE runtime → retire.

---

### INT-008 — Hard-coded Startup URL in Scripts

**Legacy**: `liberty-dev.sh:20` and `liberty-start.sh:30` print `http://localhost:9080/big-bad-monolith` as a fixed string. This combines INT-006 (hardcoded port) and INT-007 (hardcoded context root).

**Proposal**: retire

**Replacement**: The `Dockerfile` and `docker-compose.yml` document the correct service URL for local development. The `npm run dev` or `docker compose up` commands surface the correct URL from the `PORT` env var. Startup logs emit: `{"msg":"Server listening","url":"http://0.0.0.0:${PORT}"}` using the actual configured port from the pino logger.

**Configuration surface**: Derived from `PORT` (see INT-006).

**Retirement rationale**: Shell scripts that hardcode application URLs are fragile and degrade silently when configuration changes. The target emits the actual listening URL at startup from structured JSON logs, which CI/CD and local dev tooling can parse reliably.

**Traceability**: INT-006, INT-007, substitution domain — startup scripts → retire.

---

### INT-009 — Static Liberty User Registry Credentials

**Legacy**: Liberty server.xml defines a basic user registry with static credentials `user1/password` (server.xml:35,36). These credentials are committed to the repository in plaintext and provide the sole authentication mechanism for the legacy application.

**Proposal**: replace-with-library

**Replacement**: `@fastify/jwt` plugin with `bcryptjs` password hashing. Users are stored in the `users` table with `password_hash` (bcrypt, cost factor 12). Authentication flow: `POST /auth/login` receives `{email, password}`, verifies bcrypt hash, returns a signed JWT. All protected routes require `Authorization: Bearer <token>` header. No static credentials in source code or committed configuration files.

**Configuration surface**:
- `JWT_SECRET` — JWT signing secret. Required. Minimum 32 characters. No default; startup fails if not set.
- `JWT_TTL` — Token time-to-live in seconds. Default: `3600` (1 hour).
- `ADMIN_SEED_EMAIL` — Email address for first-run admin user seed. Optional; if not set, no admin is seeded.
- `ADMIN_SEED_PASSWORD` — Plain-text password for first-run admin user seed. Hashed with bcrypt before storage. Optional; required if `ADMIN_SEED_EMAIL` is set.

**Retirement rationale**: Static credentials committed to source are a critical security vulnerability (OQ-007). The Liberty basic user registry has no production-grade password hashing, no token expiry, and no audit trail. JWT + bcrypt provides all of these with a standard, auditable implementation.

**Traceability**: OQ-007, substitution domain — static Liberty user registry → replace-with-library (JWT + bcrypt).

---

### INT-010 — Embedded Derby Database Credentials

**Legacy**: `server.xml:57,58` contains Derby database credentials `user=app` / `password=app` in plaintext within the committed XML configuration. These credentials are part of the Liberty datasource definition.

**Proposal**: replace-with-platform

**Replacement**: PostgreSQL credentials are embedded in the `DATABASE_URL` connection string: `postgresql://pguser:pgpassword@host:5432/billing`. The `DATABASE_URL` is an environment variable; it is never committed to source code. The `.env.example` file documents the variable name with a placeholder value.

**Configuration surface**:
- `DATABASE_URL` — contains host, port, database name, and credentials. Required env var. Example: `postgresql://app:changeme@localhost:5432/billing`.

**Retirement rationale**: Plaintext credentials in committed XML files are a security anti-pattern. Environment-variable injection follows the 12-factor app standard for credentials. Docker secrets or a secrets manager can provide the env var value in production without exposing it in code.

**Traceability**: OQ-007, substitution domain — unfit data stores → PostgreSQL, hard-coded config → replace-with-platform (env vars).

---

### INT-011 — Liberty Fallback to Embedded Derby

**Legacy**: `LibertyConnectionManager.java:14,34` and `StartupListener.java:21` implement a fallback: if Liberty JNDI datasource init fails, the application falls back to a direct embedded Derby connection. This silently changes the active database without operator awareness.

**Proposal**: retire

**Replacement**: No fallback. The target application performs a database connectivity check during startup (`GET /ready` → checks `pg.Pool.query('SELECT 1')`). If the check fails, the process exits with code 1 and the container's liveness probe fails. This forces the orchestration layer (Docker Compose, Kubernetes) to restart the container rather than silently degrading to a different data store.

**Configuration surface**: None. Failure behavior is controlled by `DATABASE_URL` validity.

**Retirement rationale**: Silent fallback to a different database is a data-integrity hazard (related to OQ-002). The target architecture deliberately has no fallback: one database, one connection string, fail fast if unavailable (OQ-006 resolution).

**Traceability**: OQ-006 (production startup mode resolution), OQ-002 (Derby path divergence), BR-012.

---

### INT-012 — Hard-coded Gradle/Liberty Build Commands in Startup Scripts

**Legacy**: `liberty-dev.sh:11,31` and `liberty-start.sh:11,25` hardcode Gradle wrapper commands (`./gradlew libertyDev`, `./gradlew libertyStart`) and Liberty lifecycle commands. These scripts couple the development workflow to the Gradle+Liberty toolchain.

**Proposal**: retire

**Replacement**: Standard npm scripts in `package.json`:
- `npm run build` — TypeScript compilation (`tsc`)
- `npm run dev` — Development mode with ts-node or tsx watch
- `npm start` — Run compiled JS from `dist/`
- `npm test` — Run Vitest test suite
- `npm run migrate` — Run node-pg-migrate migrations
- `docker build .` — OCI container build
- `docker compose up` — Local development with PostgreSQL

**Configuration surface**: None. npm scripts are self-documenting in `package.json`.

**Retirement rationale**: Gradle and Liberty are no longer in the technology stack. Their replacement commands (`npm run *` / `docker`) are standard toolchain commands that do not require custom shell scripts. Traceability: substitution domain — startup scripts (Gradle+Liberty) → retire (replaced by `npm run build` + `docker build`).

**Traceability**: Substitution domain — startup scripts (Gradle+Liberty) → retire.

---

### INT-013 — jvm.options Sed Mutation in Production Script

**Legacy**: `liberty-start.sh:20,21,38,39` uses `sed` to mutate `jvm.options` in-place at production startup. The script modifies JVM heap size flags (`-Xmx`, `-Xms`) by pattern-matching and replacing lines in the committed file. This makes container images non-reproducible and can fail silently if the sed pattern doesn't match.

**Proposal**: retire

**Replacement**: Not applicable. The target runtime is Node.js, not a JVM. There is no `jvm.options` file. If Node.js memory limits are needed, `NODE_OPTIONS=--max-old-space-size=512` (or appropriate value) is set as an environment variable in the container definition (`docker-compose.yml`, Kubernetes Deployment, or `docker run` command). This is an environment-level concern, not an application-level concern.

**Configuration surface**:
- `NODE_OPTIONS` — Standard Node.js env var for runtime flags. Set in container environment if needed. Example: `NODE_OPTIONS=--max-old-space-size=512`.

**Retirement rationale**: Mutating committed files at runtime is an operational anti-pattern that violates container image immutability. The equivalent configuration mechanism in the target (NODE_OPTIONS env var) achieves the same result without touching any files. Retired with the Liberty runtime in Phase 6 (OQ-008 resolution).

**Traceability**: OQ-008, substitution domain — jvm.options sed mutation → retire (no JVM in target).
