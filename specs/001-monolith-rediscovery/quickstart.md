# Quickstart: Validate Rediscovery Deliverables

## Goal

Run a repeatable validation pass showing that rediscovery artifacts are complete, traceable, and compliant with constraints.

## Prerequisites

- Repository checked out at the same snapshot used for rediscovery.
- Access to feature artifacts in `specs/001-monolith-rediscovery/`.
- Basic shell tools (`rg`, `sed`, `awk`) available.

## Artifact Checklist

Confirm these files exist:
- `specs/001-monolith-rediscovery/spec.md`
- `specs/001-monolith-rediscovery/plan.md`
- `specs/001-monolith-rediscovery/research.md`
- `specs/001-monolith-rediscovery/data-model.md`
- `specs/001-monolith-rediscovery/contracts/rediscovery-artifacts-contract.md`
- `specs/001-monolith-rediscovery/quickstart.md`

## Validation Scenario 1: Rule Traceability

1. Pick any 5 business-rule entries from the eventual `business-rules.md` deliverable.
2. For each entry, verify:
   - Rule statement is plain English.
   - At least one evidence reference resolves to repository code/config.
   - At least one concrete example is provided.
3. Expected result:
   - 5/5 entries are directly verifiable without guesswork.

## Validation Scenario 2: Invariant Enforcement Classification

1. Select invariants from `data-model.md`.
2. For each invariant, verify enforcement source is one of:
   - `DB Constraint`, `Defensive Code`, `Both`, `Not Enforced`.
3. Cross-check evidence references against source files.
4. Expected result:
   - 100% of sampled invariants have valid classification + evidence.

## Validation Scenario 3: Integration Inventory Completeness

1. Build a raw touchpoint list using repository search:
   - JDBC URLs (`jdbc:`)
   - JNDI lookups (`lookup(`)
   - localhost endpoints (`http://localhost`)
   - explicit file/data paths (`/data/`, `./data/`)
2. Compare findings with `integration-inventory.md`.
3. Expected result:
   - Every touchpoint discovered in scope appears exactly once in inventory.

## Validation Scenario 4: Uncertainty Protocol

1. Inspect all deliverables for statements that appear uncertain.
2. Confirm each unresolved claim exists in `open-questions.md`.
3. Confirm no guessed behavior is marked as observed fact.
4. Expected result:
   - All unresolved behavior is captured as open questions.

## Validation Scenario 5: Constraint Compliance

1. Confirm deliverables contain no code changes or migration proposals.
2. Confirm outputs are documentation-only and observational.
3. Expected result:
   - Constraint compliance passes fully.

## Optional Runtime Cross-check (Read-only)

Use existing scripts only if you want to compare documentation against running behavior:
- `./liberty-dev.sh` or `./liberty-start.sh`

This step is optional and does not modify source code.

## Success Criteria Mapping

Validation is successful when:
- Every sampled rule is traceable to concrete evidence.
- Every sampled invariant has clear enforcement source.
- Integration inventory covers all discovered touchpoints in scope.
- Open questions capture all unresolved uncertainty.
- No prohibited content (guesses, modernization proposals, code edits) appears.
