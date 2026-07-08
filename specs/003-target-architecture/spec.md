# Feature Specification: Target Architecture

**Feature Branch**: `[003-target-architecture]`

**Created**: 2026-07-08

**Status**: Draft

**Input**: User description: "Specify the target architecture for the modernized system, informed by the rediscovery (phase 1) and substitution audit (phase 2). Deliverables: (1) target runtime and platform, (2) module/service boundaries, (3) data model and migration path from the legacy schema, (4) integration contracts replacing each flagged legacy integration, (5) cross-cutting concerns (auth, logging, config, secrets, observability), (6) migration strategy (default: strangler-fig), (7) risks. Constraint: architecture only — no code, no per-module rewrite tasks yet. Success: every architectural choice traces back to either a preserved business rule or a substitution-audit row."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Architecture Review Panel Validates Platform and Boundaries (Priority: P1)

An architecture review panel receives a complete target-architecture document and can verify that every platform choice and service boundary is justified by preserved business rules or substitution-audit classifications from phases 1 and 2.

**Why this priority**: Without a validated platform and service boundary definition, no migration planning or implementation can proceed. This is the minimum viable architectural decision set.

**Independent Test**: Select any three architectural choices (e.g., target runtime, data store replacement, a service boundary). For each, locate the BR-### or substitution-audit domain entry that justifies it. All three must resolve.

**Acceptance Scenarios**:

1. **Given** the target architecture document is complete, **When** a reviewer reads the platform section, **Then** they can find a direct traceability link for each platform choice back to a phase-1 business rule or phase-2 substitution row.
2. **Given** the service boundaries section, **When** a reviewer maps each business rule (BR-001..BR-014) to a boundary, **Then** every rule is owned by exactly one named boundary.

---

### User Story 2 - Migration Team Uses Data Model Spec and Migration Path (Priority: P2)

A migration engineer receives the data model specification and migration path and can determine the exact schema changes required and the sequence to migrate data from the legacy Derby store to the target data store without data loss or field inversion.

**Why this priority**: The data migration is a blocking dependency for all other migration phases. The User constructor inversion bug (OQ-003) makes this particularly high risk.

**Independent Test**: Given the data model section, a reviewer can enumerate all four legacy tables, identify the field corrections required (specifically OQ-003), and describe the migration sequence without ambiguity.

**Acceptance Scenarios**:

1. **Given** the data model section, **When** a migration engineer reads the schema mapping, **Then** the target schema for every legacy table is explicitly defined with any field corrections documented and referenced.
2. **Given** the migration path description, **When** an engineer follows the steps, **Then** they can migrate the Derby data to the target database in a repeatable, reversible sequence.
3. **Given** the open question OQ-003 about the User constructor inversion, **When** a reviewer reads the migration path, **Then** the spec explicitly records the corrective mapping and its justification.

---

### User Story 3 - Integration Owners Validate Replacement Contracts (Priority: P3)

An integration owner for each legacy integration point (INT-001..INT-013) receives a named replacement contract that specifies what replaces the legacy mechanism and what the configuration surface is.

**Why this priority**: The legacy integrations (embedded Derby, static credentials, hard-coded paths, direct JDBC in JSP) are the primary cloud-hostility drivers. Each must have a designated replacement before migration can start.

**Independent Test**: For each INT-### entry from phase-1 rediscovery, the architecture document names the replacement mechanism (or explicit retirement) and the configuration variable or platform service that replaces it.

**Acceptance Scenarios**:

1. **Given** integration INT-001 (embedded Derby JDBC URL), **When** an owner reads the integration contracts section, **Then** they find a named replacement datasource configuration and the environment variable that carries the connection string.
2. **Given** integration INT-009 (static Liberty user registry credentials), **When** an owner reads the auth section, **Then** they find a replacement auth mechanism that eliminates the static credential.
3. **Given** integration INT-013 (script-based JVM mutation), **When** an owner reads the contracts section, **Then** they find a retirement decision with rationale.

---

### User Story 4 - Security and Platform Teams Approve Cross-Cutting Concerns (Priority: P4)

Security, platform, and SRE teams receive a cross-cutting concerns specification that defines auth, logging, config management, secrets handling, and observability patterns. Each pattern is environment-agnostic and replaces a known legacy concern.

**Why this priority**: Cross-cutting concerns cut across all boundaries; if they are undefined when boundaries are implemented, each team will make inconsistent choices.

**Independent Test**: A security reviewer can verify that all three legacy credential risks (OQ-007, INT-009, INT-010) are addressed by a named pattern in the cross-cutting concerns section.

**Acceptance Scenarios**:

1. **Given** the cross-cutting concerns section, **When** a security reviewer reads the secrets strategy, **Then** static credentials (INT-009, INT-010) have a named replacement mechanism and no secret values appear in any config file.
2. **Given** the observability subsection, **When** an SRE reads it, **Then** health check, structured log format, and metrics exposure are each specified to a level sufficient to configure alerting.

---

### User Story 5 - Stakeholders Review Risks Before Implementation Begins (Priority: P5)

Product and engineering stakeholders receive a risk register that surfaces every unresolved open question (OQ-001..OQ-008) as an architectural risk, with severity, impact on the migration, and a mitigation or decision gate.

**Why this priority**: Several open questions from phase 1 are blocking-level risks for specific migration phases. Stakeholders must acknowledge them before implementation starts.

**Independent Test**: Every OQ-### entry from phase-1 open questions maps to at least one risk entry. Each risk entry has a severity, a migration-phase impact, and a mitigation owner or decision gate.

**Acceptance Scenarios**:

1. **Given** the risk register, **When** a stakeholder looks up OQ-001 (JSP bypassing service validation), **Then** they find the risk, its severity, the migration phase it affects, and the required decision or mitigation.
2. **Given** the risk register, **When** a stakeholder looks up OQ-003 (User constructor inversion), **Then** they find a data-loss risk classification with a migration gate requiring data audit before the Derby-to-target migration executes.

---

### Edge Cases

- What if an architectural choice cannot be traced to a specific BR-### or substitution-audit row? It must be explicitly flagged as an assumption and require stakeholder approval.
- What if two business rules conflict under the target architecture (e.g., OQ-001: JSP direct DAO path vs. service validation path)? The spec must record the design decision and its architectural consequence.
- What if the strangler-fig phases produce a period where Derby and the target data store coexist? The spec must explicitly address data consistency during dual-write phase.
- What if a legacy integration (e.g., INT-011 Liberty fallback) has no viable target replacement? It must be retired with evidence that no business rule depends on fallback behavior.
- What if OQ-002 (Derby path ambiguity) means reports.jsp was reading a different physical DB in production? The migration path must account for potential data divergence.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The architecture MUST specify a target runtime (TypeScript on Node.js LTS), a deployment packaging format (OCI container image), and a target HTTP framework (Express or Fastify), each traceable to a substitution-audit domain.
- **FR-002**: The architecture MUST define named module/service boundaries, with each boundary owning at least one business rule (BR-001..BR-014).
- **FR-003**: The architecture MUST define the target relational schema for all four legacy entities (User, Customer, BillingCategory, BillableHour), including any field corrections required (OQ-003), and a migration path from Derby.
- **FR-004**: The architecture MUST define a replacement contract for each legacy integration point INT-001..INT-013, explicitly naming the replacement mechanism or the retirement justification.
- **FR-005**: The architecture MUST define the auth pattern, replacing the static Liberty user registry (INT-009, OQ-007).
- **FR-006**: The architecture MUST define the logging strategy, including log format and output destination, replacing the embedded/unstructured logging in the legacy app.
- **FR-007**: The architecture MUST define the configuration and secrets strategy, replacing hard-coded values in `server.xml`, `bootstrap.properties`, and `jvm.options` (INT-006, INT-007, INT-009, INT-010, INT-013).
- **FR-008**: The architecture MUST define the observability strategy (health checks, metrics, distributed tracing readiness).
- **FR-009**: The architecture MUST define a strangler-fig migration strategy with named phases, entry/exit criteria per phase, and data-consistency handling during any dual-store transition.
- **FR-010**: The architecture MUST include a risk register where every OQ-### entry from phase-1 open questions maps to at least one risk entry with severity, migration-phase impact, and mitigation.
- **FR-011**: Every architectural choice MUST include a traceability note citing at least one BR-### business rule or substitution-audit domain entry that justifies it.
- **FR-012**: The architecture MUST NOT prescribe code-level implementations, per-module rewrite task lists, or sprint/iteration sequencing.
- **FR-013**: The architecture MUST NOT introduce new scope beyond what is motivated by phase-1 business rules and phase-2 substitution classifications.

### Key Entities *(include if feature involves data)*

- **Architectural Decision Record (ADR)**: A named choice with justification, alternatives considered, traceability to BR-### or substitution-audit domain, and consequences.
- **Integration Replacement Contract**: A mapping from a legacy INT-### entry to its target mechanism, configuration surface, and retirement or replacement rationale.
- **Risk Register Entry**: A mapping from an OQ-### open question or observed risk to severity, affected migration phase, and mitigation/gate.
- **Service Boundary**: A named module owning a cohesive set of business rules, with defined in/out API surface and data ownership.
- **Migration Phase**: A named strangler-fig phase with entry criteria, exit criteria, rollback path, and data-consistency strategy.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of architectural choices in the platform, boundaries, data model, integration contracts, and cross-cutting concerns sections include a traceability note citing at least one BR-### or substitution-audit domain entry.
- **SC-002**: 100% of legacy integration points (INT-001..INT-013) have an explicit replacement contract or retirement justification.
- **SC-003**: 100% of open questions (OQ-001..OQ-008) map to at least one risk register entry with severity and mitigation.
- **SC-004**: All four legacy entities (User, Customer, BillingCategory, BillableHour) are represented in the target schema with field corrections documented.
- **SC-005**: The migration strategy defines a complete strangler-fig phase sequence with entry and exit criteria for each phase.
- **SC-006**: 0 sections contain code-level implementations or per-module rewrite task lists.
- **SC-007**: In architecture review sampling, at least 95% of architectural choices can be traced to source evidence without external explanation.

---

## Assumptions

- Phase-1 rediscovery artifacts in `specs/001-monolith-rediscovery/` are the authoritative baseline. All 14 business rules (BR-001..BR-014) and 13 integration entries (INT-001..INT-013) are accepted as correct observations.
- Phase-2 substitution audit contracts and coverage domains in `specs/002-substitution-audit/` govern the substitution classification framing, even though the working table rows are not yet fully populated; domain-level evidence from phase-1 is sufficient to derive architectural substitution choices.
- **Target language and runtime**: TypeScript on Node.js (LTS). The system is rewritten in TypeScript — not retained in Java. This is an explicit stakeholder preference.
- **Target framework**: Lightweight HTTP framework — Express or Fastify. No enterprise framework (NestJS, Spring, Liberty). The plan phase will confirm the specific framework.
- **Target data store**: PostgreSQL. Replaces embedded Apache Derby. No NoSQL or document store is required by any business rule.
- **Deployment target**: OCI container image (Docker). The system ships as a container, not a WAR or Liberty server. This is an explicit stakeholder preference.
- The migration strategy defaults to strangler-fig unless a stakeholder explicitly overrides it before plan.md authoring.
- OQ-003 (User constructor inversion) is treated as a bug requiring correction in migration, not an intentional mapping, pending stakeholder confirmation.
- OQ-001 (JSP direct DAO path) resolution will determine whether a validation enforcement gate is added to the billable-hours boundary in the target; the spec records both options and defers the final choice to the plan phase.
- The modernized system is a single deployable unit (modular monolith) initially; microservice decomposition is not a phase-1 migration target.
- No external system integrations (ERP, CRM, payment) are present in the legacy codebase; the target architecture does not introduce any.
- Reviewers have access to phase-1 and phase-2 artifacts when evaluating this specification.
