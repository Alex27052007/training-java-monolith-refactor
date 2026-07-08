# Feature Specification: Legacy Monolith Rediscovery

**Feature Branch**: `[001-monolith-rediscovery]`

**Created**: 2026-07-08

**Status**: Draft

**Input**: User description: "Specify a rediscovery of this legacy Java monolith. Deliverables: (1) a business-rules document in plain English with concrete code references and examples, (2) a data-model summary — entities, relationships, invariants (including those enforced only by defensive code or DB constraints), (3) an integration inventory (every external system, filesystem path, hard-coded endpoint), (4) an open-questions list for anything the code alone cannot answer. Constraints: no code changes, no modernization proposals, stop and ask before guessing behavior. Success: a domain reviewer can validate each rule against a concrete code reference."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Traceable Business Rules (Priority: P1)

A domain reviewer receives a plain-English business-rules document where each rule is linked to concrete source evidence and at least one concrete example, so they can verify what the monolith currently does without relying on interpretation.

**Why this priority**: This is the core rediscovery outcome and the primary success condition defined by the requester.

**Independent Test**: Can be fully tested by selecting any documented rule and confirming the rule statement, supporting example, and code reference can be validated directly against the codebase.

**Acceptance Scenarios**:

1. **Given** the rediscovery package is complete, **When** a reviewer inspects any listed business rule, **Then** they can find a concrete source reference and example that supports the rule.
2. **Given** a rule depends on multiple code locations, **When** the reviewer checks the rule entry, **Then** the entry identifies all required references needed to validate behavior.

---

### User Story 2 - Verified Data Model Summary (Priority: P2)

A domain reviewer receives a summary of entities, relationships, and invariants that distinguishes where constraints are enforced by schema constraints versus defensive application code.

**Why this priority**: Data semantics and invariants are essential to understanding current behavior and avoiding incorrect assumptions.

**Independent Test**: Can be fully tested by reviewing each entity/relationship/invariant entry and validating that each claim cites concrete evidence for either database-level enforcement or code-level enforcement.

**Acceptance Scenarios**:

1. **Given** the data-model summary, **When** a reviewer checks any invariant, **Then** it is labeled with its enforcement source (database constraint, defensive code, or both) and supporting reference.
2. **Given** a relationship is listed, **When** the reviewer validates it, **Then** both the participating entities and relationship cardinality/optionality are explicitly stated.

---

### User Story 3 - Integration Inventory and Open Questions (Priority: P3)

A technical reviewer receives a complete inventory of external dependencies (systems, file paths, hard-coded endpoints) plus an explicit open-questions list for ambiguities that code alone cannot resolve.

**Why this priority**: Completeness of dependency mapping and explicit unknowns reduce hidden risk during later analysis and planning.

**Independent Test**: Can be fully tested by scanning all code/config/runtime scripts for external touchpoints, then confirming each is present in the inventory and unresolved items are captured as open questions rather than guessed.

**Acceptance Scenarios**:

1. **Given** the integration inventory, **When** a reviewer cross-checks known integration surfaces in code and config, **Then** each surface is present with at least one source reference.
2. **Given** behavior cannot be determined from code alone, **When** the rediscovery outputs are reviewed, **Then** the uncertainty appears in open questions and is not represented as a factual rule.

---

### Edge Cases

- How are contradictory behaviors handled when JSP-layer logic and service-layer logic differ for the same domain concept?
- How are nullable fields represented when one layer treats null as valid but another implicitly assumes non-null?
- How are date boundaries handled when monthly reports use fixed day bounds and logged hours can include any calendar day?
- How is behavior documented when exception handling swallows root causes and only user-facing messages remain?
- How are fallback paths documented when runtime can switch between Liberty-managed datasource and embedded Derby mode?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The rediscovery output package MUST include a business-rules document written in plain English.
- **FR-002**: Every business rule MUST include at least one concrete code reference and one concrete example that demonstrates the rule in practice.
- **FR-003**: The business-rules document MUST distinguish between observed behavior and inferred interpretation; inferred behavior is prohibited unless explicitly confirmed.
- **FR-004**: The rediscovery output package MUST include a data-model summary covering entities, relationships, and invariants currently present in the system.
- **FR-005**: Each invariant in the data-model summary MUST identify whether enforcement is from schema constraints, defensive code, or both.
- **FR-006**: The rediscovery output package MUST include an integration inventory of all externally relevant touchpoints found in scope, including external systems, filesystem paths, and hard-coded endpoints.
- **FR-007**: Every integration inventory entry MUST include source evidence and classification (system integration, filesystem path, endpoint, runtime dependency, or configuration dependency).
- **FR-008**: The rediscovery output package MUST include an open-questions list for all material uncertainties that cannot be resolved from code and configuration alone.
- **FR-009**: For uncertain behavior, the process MUST stop at uncertainty and record a question instead of guessing behavior.
- **FR-010**: The scope MUST be observational only: no code changes, no modernization proposals, and no recommendations that alter current behavior.
- **FR-011**: Each deliverable entry MUST be independently verifiable by a reviewer using only the cited references and examples.
- **FR-012**: The package MUST include a coverage statement that identifies analyzed source areas and any explicitly excluded areas.

### Key Entities *(include if feature involves data)*

- **Business Rule Record**: A traceable statement of system behavior with plain-English wording, validation references, examples, and confidence status.
- **Data Model Fact**: A documented entity, relationship, or invariant with enforcement source and supporting evidence.
- **Integration Entry**: A catalog item for an external interaction surface, including type, location, and evidence.
- **Open Question**: A bounded unknown that cannot be proven from available artifacts, including why evidence is insufficient and what clarification is needed.
- **Evidence Reference**: A normalized citation pointing to concrete code/config/script location used to validate a claim.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of documented business rules include at least one concrete evidence reference and one concrete example.
- **SC-002**: 100% of documented invariants identify their enforcement source (schema, defensive code, or both) with verifiable evidence.
- **SC-003**: 100% of discovered external touchpoints in analyzed scope are represented in the integration inventory with evidence references.
- **SC-004**: 100% of unresolved behavioral uncertainties are captured in open questions; none are represented as confirmed facts without evidence.
- **SC-005**: In reviewer validation, at least 95% of sampled rule entries are judged directly traceable from statement to evidence without additional author clarification.

## Assumptions

- Rediscovery scope includes application source, JSP views, runtime/config files, and startup scripts present in the repository.
- Database runtime inspection is out of scope; schema-level facts are derived from schema creation statements and SQL usage present in code/config.
- Existing comments, sample data, and user-facing labels are treated as supplementary clues and not authoritative evidence unless corroborated by executable behavior.
- The reviewer evaluating rule validity has access to the same repository snapshot used for rediscovery.
- Deliverables are documentation artifacts only and do not require runtime environment changes.
