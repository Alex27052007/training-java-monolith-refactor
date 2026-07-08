# Contract: Substitution Audit Deliverables

## Purpose

Define required structure and validation criteria for substitution-audit outputs derived from phase-1 rediscovery artifacts.

## Required Artifacts

- `substitution-audit.md` (primary table and coverage sections)
- `coverage.md` (domain completion summary and null-finding rationale when needed)
- `traceability-matrix.md` (optional helper if needed for reviewer navigation)

## Substitution Audit Table Contract

Each row MUST contain:
- `legacy element`
- `proposal` (`keep-as-is` | `replace-with-library` | `replace-with-platform` | `retire`)
- `reason`
- `trade-off`
- `rediscovery link` (path + entry locator)

Validation:
- Proposal values outside enum are invalid.
- Empty `reason` or `trade-off` fields are invalid.
- Missing rediscovery link is invalid.

## Coverage Contract

Mandatory coverage domains:
- Home-grown code with modern equivalents
- Dated integrations
- Unfit data stores
- EOL runtimes/frameworks
- Cloud-hostile operational assumptions

Validation:
- Every domain must have either at least one row or an evidence-backed null finding.
- Domain-level claims without rediscovery links are invalid.

## Audit-Only Constraint Contract

Prohibited content:
- Target architecture designs
- Migration implementation sequencing
- Detailed system blueprinting

Allowed content:
- Evaluation of substitution category
- Comparative trade-offs
- Evidence-backed risk notes

## Uncertainty Handling Contract

If a row depends on unresolved rediscovery ambiguity:
- Add open-question reference
- Mark confidence level
- Keep recommendation architecture-agnostic

## Reviewer Acceptance Criteria

A reviewer must be able to:
- Pick any row and trace it to rediscovery evidence without external explanation.
- Confirm all five coverage domains are explicitly addressed.
- Verify no target-architecture design details are present.
