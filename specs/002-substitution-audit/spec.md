# Feature Specification: Substitution Audit

**Feature Branch**: `[002-substitution-audit]`

**Created**: 2026-07-08

**Status**: Draft

**Input**: User description: "Specify a substitution audit over the rediscovery artifacts (phase 1). Deliverables: (1) a substitution-audit table with columns legacy element, proposal (keep-as-is | replace-with-library | replace-with-platform | retire), reason, trade-off; (2) coverage spanning home-grown code with modern equivalents, dated integrations, unfit data stores, EOL runtimes/frameworks, and cloud-hostile operational assumptions. Constraint: audit only — do not design the new architecture. Success: every row links back to a specific rediscovery entry."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Traceable Substitution Decisions (Priority: P1)

A modernization reviewer receives a substitution-audit table where each row maps one legacy element to a proposal category and is traceable back to an explicit rediscovery artifact entry.

**Why this priority**: This is the primary output and explicit success condition for the feature.

**Independent Test**: Select any 10 rows from the audit table and verify each row contains a proposal category, reason, trade-off, and a valid reference to a rediscovery artifact entry.

**Acceptance Scenarios**:

1. **Given** the substitution-audit document is complete, **When** a reviewer checks any row, **Then** they can locate the referenced rediscovery entry and validate the row rationale.
2. **Given** a legacy element appears in multiple rediscovery artifacts, **When** it is represented in the audit table, **Then** the row includes the authoritative reference used for classification.

---

### User Story 2 - Complete Coverage Across Required Risk Domains (Priority: P2)

A technical lead receives coverage that explicitly spans home-grown code candidates for replacement, dated integrations, unfit data stores, EOL runtimes/frameworks, and cloud-hostile operational assumptions.

**Why this priority**: Coverage breadth determines whether the audit is useful for downstream planning and risk reduction.

**Independent Test**: Verify the audit includes at least one dedicated coverage section for each required domain and that each section links to supporting rediscovery entries.

**Acceptance Scenarios**:

1. **Given** the audit package, **When** a reviewer checks required domains, **Then** all five required coverage domains are present and populated.
2. **Given** a domain has no qualifying candidates, **When** the reviewer inspects that section, **Then** the section states explicit evidence for why no candidates were found.

---

### User Story 3 - Audit-Only Governance (Priority: P3)

A governance reviewer verifies that the document remains an audit artifact and does not include target architecture design, implementation sequencing, or migration design details.

**Why this priority**: The user explicitly constrained this work to audit-only scope.

**Independent Test**: Review all sections and confirm there are no architecture blueprints, migration plans, or implementation-level design prescriptions.

**Acceptance Scenarios**:

1. **Given** the completed document, **When** a reviewer scans recommendations, **Then** each entry stays at substitution-decision level only.
2. **Given** architectural alternatives are mentioned, **When** reviewed, **Then** they are framed only as trade-offs tied to substitution categories, not as a future system design.

---

### Edge Cases

- What if a legacy element fits more than one proposal category depending on context?
- What if rediscovery artifacts contain conflicting observations for the same legacy element?
- How is a row handled when supporting evidence exists but confidence remains low due to unresolved open questions?
- How is coverage reported when a required domain is present but has only indirect evidence?
- How are duplicated legacy elements across service, DAO, and JSP layers deduplicated without losing traceability?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The output MUST include a substitution-audit table.
- **FR-002**: Every audit row MUST include columns `legacy element`, `proposal`, `reason`, and `trade-off`.
- **FR-003**: The `proposal` value MUST be one of: `keep-as-is`, `replace-with-library`, `replace-with-platform`, `retire`.
- **FR-004**: Every audit row MUST link to at least one specific rediscovery artifact entry as source evidence.
- **FR-005**: The audit MUST include explicit coverage for home-grown code with modern equivalents.
- **FR-006**: The audit MUST include explicit coverage for dated integrations.
- **FR-007**: The audit MUST include explicit coverage for unfit data stores.
- **FR-008**: The audit MUST include explicit coverage for EOL runtimes/frameworks.
- **FR-009**: The audit MUST include explicit coverage for cloud-hostile operational assumptions.
- **FR-010**: The audit MUST remain architecture-agnostic and MUST NOT include target architecture design.
- **FR-011**: The audit MUST include handling notes for uncertain classifications tied to existing open questions.
- **FR-012**: The final artifact MUST be reviewable by mapping each row from decision to evidence source without external context.

### Key Entities *(include if feature involves data)*

- **Substitution Audit Row**: A decision record containing legacy element, proposal category, reason, trade-off, and evidence link.
- **Coverage Domain Section**: A grouped view of rows for one required domain (home-grown equivalents, dated integrations, unfit data stores, EOL runtimes/frameworks, cloud-hostile assumptions).
- **Evidence Link**: A reference to a concrete rediscovery entry proving why a row was classified as proposed.
- **Uncertainty Marker**: A note attached to rows where open questions affect confidence.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of substitution-audit rows include all required columns and a valid proposal category.
- **SC-002**: 100% of substitution-audit rows include at least one direct link to a specific rediscovery entry.
- **SC-003**: 100% of required coverage domains are represented with explicit findings or explicit evidence-based null findings.
- **SC-004**: 0 sections contain target architecture design or implementation planning details.
- **SC-005**: In reviewer sampling, at least 95% of sampled rows are judged immediately traceable from proposal to rediscovery evidence.

## Assumptions

- Rediscovery artifacts in `specs/001-monolith-rediscovery/` are the authoritative evidence baseline for this audit.
- The audit is evaluative and comparative only; downstream design decisions are out of scope.
- Proposal categories are sufficient to classify legacy elements at audit level without implementation detail.
- If rediscovery evidence is ambiguous, the audit records uncertainty rather than inferring missing behavior.
- Reviewers have access to both the substitution audit and referenced rediscovery artifacts.
