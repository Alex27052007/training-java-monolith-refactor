# Research: Legacy Monolith Rediscovery

## Scope

Research resolves planning unknowns and selects documentation patterns for producing evidence-first rediscovery deliverables.

## Decisions

### 1) Evidence format for business rules
- Decision: Use a strict Rule Record format: `Claim`, `Evidence References`, `Concrete Example`, `Confidence`.
- Rationale: This ensures every rule is reviewer-verifiable and aligns with traceability requirements in the feature spec.
- Alternatives considered:
  - Free-form prose: rejected because validation becomes subjective.
  - Reference-only format: rejected because examples are required for reviewer confidence.

### 2) Ambiguity handling protocol
- Decision: Any behavior that cannot be directly proven from code/config is captured in `Open Questions` and not asserted as fact.
- Rationale: The feature explicitly forbids guessing behavior and requires uncertainty capture.
- Alternatives considered:
  - Infer likely behavior from naming/comments: rejected as non-verifiable.
  - Mark assumptions inline with rules: rejected because assumptions can be mistaken as facts.

### 3) Invariant classification model
- Decision: Classify each invariant by enforcement source: `DB Constraint`, `Defensive Code`, `Both`, `Not Enforced`.
- Rationale: The feature requires explicit differentiation between database and application-level enforcement.
- Alternatives considered:
  - Single invariant list: rejected because it obscures enforcement location.
  - DB-only perspective: rejected because important safeguards exist only in code.

### 4) Integration inventory taxonomy
- Decision: Use inventory categories `External System`, `Filesystem Path`, `Hard-coded Endpoint`, `Runtime Dependency`, `Configuration Dependency`.
- Rationale: This maps directly to requested deliverables and supports complete integration coverage.
- Alternatives considered:
  - Single flat list: rejected because reviewers cannot audit coverage by integration class.

### 5) Cross-layer conflict treatment
- Decision: Treat UI/DAO/service divergence as a first-class finding and convert unresolved precedence into open questions.
- Rationale: Repository evidence shows divergent logic paths (for example, direct JDBC in JSP vs DAO/service flows), which can produce behavior mismatches.
- Alternatives considered:
  - Prefer service layer by convention: rejected because UI directly executes data logic in several paths.
  - Prefer UI behavior only: rejected because service rules may still govern other execution paths.

### 6) Traceability granularity
- Decision: Cite line-level references for sensitive claims (validation logic, SQL predicates, hard-coded paths/endpoints).
- Rationale: Fine-grained references minimize interpretation drift and speed reviewer validation.
- Alternatives considered:
  - File-level references only: rejected as too coarse for disputed behaviors.

### 7) Documentation contract for deliverables
- Decision: Define a formal contract document for deliverable structure and required fields.
- Rationale: A contract makes review objective and supports future repeatability.
- Alternatives considered:
  - Rely on narrative conventions only: rejected due to inconsistent output risk.

### 8) Agent context update step
- Decision: Use available Speckit integration command (`specify integration status`) as the explicit context-refresh step because no repository-local update-agent-context script exists.
- Rationale: The planning instructions require running an agent update step; this command verifies active integration and managed-file alignment.
- Alternatives considered:
  - Skip update step: rejected to preserve workflow completeness.
  - Run non-existent script: rejected because no executable script is present in repository scripts.

## Resolved Technical Context Clarifications

All technical context fields are now resolved; no `NEEDS CLARIFICATION` markers remain.

## Research Outcome Summary

The selected approach is a strict evidence-first documentation workflow with explicit ambiguity boundaries, source-level traceability, and structured contracts for deliverables. This satisfies rediscovery constraints while keeping scope observational-only.
