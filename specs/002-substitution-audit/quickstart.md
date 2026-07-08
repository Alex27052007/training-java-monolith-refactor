# Quickstart: Validate Substitution Audit Plan Outputs

## Goal

Validate that substitution-audit deliverables are complete, traceable to phase-1 rediscovery artifacts, and constrained to audit-only scope.

## Prerequisites

- Repository contains completed phase-1 rediscovery artifacts in `specs/001-monolith-rediscovery/`.
- Feature specification and plan exist in `specs/002-substitution-audit/`.

## Validation Scenario 1: Table Schema Conformance

1. Open `substitution-audit.md`.
2. Verify every row has columns:
   - `legacy element`
   - `proposal`
   - `reason`
   - `trade-off`
   - rediscovery evidence link
3. Confirm `proposal` values are only from the allowed enum.

Expected result:
- 100% row schema conformance.

## Validation Scenario 2: Evidence Traceability

1. Sample at least 10 rows from `substitution-audit.md`.
2. Follow each rediscovery link to `specs/001-monolith-rediscovery/*`.
3. Verify row `reason` is supported by linked evidence.

Expected result:
- All sampled rows are directly traceable.

## Validation Scenario 3: Required Domain Coverage

1. Review sections for all mandatory domains:
   - home-grown modern equivalents
   - dated integrations
   - unfit data stores
   - EOL runtimes/frameworks
   - cloud-hostile assumptions
2. Confirm each domain has findings or explicit null-finding rationale with links.

Expected result:
- 100% mandatory domains covered.

## Validation Scenario 4: Audit-Only Boundary

1. Scan all artifacts for architecture design language (target topology, migration blueprint, implementation sequencing).
2. Ensure findings remain substitution classifications and trade-off notes only.

Expected result:
- No target architecture design content present.

## Validation Scenario 5: Uncertainty Handling

1. Identify rows referencing unresolved rediscovery questions.
2. Verify each includes open-question references and confidence notes.

Expected result:
- Uncertainty is explicit and not silently inferred.

## Completion Criteria

Audit planning artifacts are ready when:
- Contract conformance passes.
- Traceability pass succeeds.
- Coverage pass succeeds.
- Audit-only boundary is preserved.
