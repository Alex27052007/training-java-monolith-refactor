# Reviewer Validation Checklist

## Artifact Presence

- [x] `business-rules.md` exists
- [x] `data-model.md` exists
- [x] `integration-inventory.md` exists
- [x] `open-questions.md` exists
- [x] `coverage.md` exists

## Business Rules Quality

- [x] Rules are plain English
- [x] Each rule includes concrete code evidence
- [x] Each rule includes a concrete example
- [x] Unknown behavior is not asserted as fact

## Data Model Quality

- [x] Entities documented with fields
- [x] Relationships documented
- [x] Invariant matrix includes enforcement source
- [x] Defensive-code-only and DB-only constraints are identified

## Integration Inventory Quality

- [x] Databases and datasource paths included
- [x] Hard-coded endpoints/ports included
- [x] Filesystem paths included
- [x] Runtime and configuration dependencies included

## Open Questions Quality

- [x] Unknowns are explicit
- [x] Each question states why evidence is insufficient
- [x] Each question identifies who/what can answer it

## Constraints Compliance

- [x] No modernization proposals included
- [x] No code changes made as part of rediscovery artifacts
- [x] Documentation is reviewable by domain experts
