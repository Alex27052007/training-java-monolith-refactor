# Research: Substitution Audit Over Rediscovery Artifacts

## Scope

This research defines audit methodology and decision patterns for classifying legacy elements based on phase-1 rediscovery evidence only.

## Decisions

### 1) Decision taxonomy for substitution rows
- Decision: Use only four proposal values: `keep-as-is`, `replace-with-library`, `replace-with-platform`, `retire`.
- Rationale: This is an explicit feature constraint and keeps evaluation architecture-agnostic.
- Alternatives considered:
  - Introducing extra states like `rewrite`: rejected because it implies solution design.
  - Free-form recommendation text: rejected because it reduces comparability.

### 2) Row-level evidence requirement
- Decision: Every row must include at least one direct link to a rediscovery entry (not only source code references).
- Rationale: Success criteria require direct mapping from substitution conclusion to rediscovery evidence.
- Alternatives considered:
  - Linking directly to source code only: rejected because this bypasses the rediscovery audit trail.

### 3) Coverage-first sectioning
- Decision: Structure audit output by five required coverage domains plus cross-domain summary.
- Rationale: Ensures required breadth is verifiable and no domain is accidentally omitted.
- Alternatives considered:
  - Flat list of rows only: rejected because coverage completeness would be harder to verify.

### 4) Handling uncertainty
- Decision: When confidence is reduced by unresolved rediscovery questions, keep row classification but attach explicit uncertainty note and open-question reference.
- Rationale: Maintains audit completeness while avoiding hidden assumptions.
- Alternatives considered:
  - Dropping uncertain rows: rejected because it creates audit blind spots.
  - Guessing likely answers: rejected by feature constraints.

### 5) Home-grown equivalence judgment
- Decision: Evaluate home-grown components against modern equivalents at capability level, not architecture blueprint level.
- Rationale: Preserves audit-only scope and avoids prescribing target design.
- Alternatives considered:
  - Proposing concrete replacement architectures: rejected as out of scope.

### 6) Dated integration classification
- Decision: Treat explicit embedded/runtime-coupled paths, direct JDBC in UI, and static credentials as dated integration candidates.
- Rationale: These are evidence-backed in rediscovery outputs and have clear modernization pressure.
- Alternatives considered:
  - Restricting to external APIs only: rejected because current system risk includes internal integration patterns.

### 7) Unfit data-store assessment
- Decision: Flag data-store fit issues based on operational constraints evidenced in rediscovery (path coupling, embedded assumptions, environment ambiguity).
- Rationale: Unfit-for-purpose can be detected without prescribing target datastore.
- Alternatives considered:
  - Limiting to technology age/EOL alone: rejected because operational fit is a separate concern.

### 8) EOL/runtime coverage method
- Decision: Use repository-declared stack/runtime dependencies and known lifecycle risk categories to audit EOL exposure.
- Rationale: Enables evidence-based risk categorization without implementation design.
- Alternatives considered:
  - Runtime scanning in deployed environments: rejected as out of scope for repository-only audit.

### 9) Cloud-hostile assumptions coverage
- Decision: Identify assumptions such as local filesystem coupling, mutable config scripts, localhost-centric endpoints, and static credentials.
- Rationale: These assumptions are directly observable in phase-1 artifacts.
- Alternatives considered:
  - Deferring cloud-hostile analysis to architecture phase: rejected because it is a required coverage domain now.

## Resolved Clarifications

No unresolved `NEEDS CLARIFICATION` markers remain for planning this audit.

## Research Outcome Summary

The substitution audit will be produced as a traceable, evidence-linked classification matrix with mandatory domain coverage and explicit uncertainty handling, while remaining strictly architecture-agnostic.
