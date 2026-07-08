# Traceability Matrix

## Requirement to Artifact Mapping

| Requirement | Covered In | Notes |
|---|---|---|
| FR-001 | `business-rules.md` | Plain-English business rules documented |
| FR-002 | `business-rules.md` | Every rule includes evidence + concrete example |
| FR-003 | `business-rules.md`, `open-questions.md` | Unknown behavior diverted to open questions |
| FR-004 | `data-model.md` | Entities/relationships/invariants documented |
| FR-005 | `data-model.md` | Enforcement source matrix included |
| FR-006 | `integration-inventory.md` | External touchpoints inventoried |
| FR-007 | `integration-inventory.md` | Entries include category, location, value, impact |
| FR-008 | `open-questions.md` | Unresolved behavior captured explicitly |
| FR-009 | `open-questions.md`, `ambiguity-log.md` | No guessing protocol applied |
| FR-010 | `spec.md`, `plan.md`, all rediscovery docs | Documentation-only output; no code change |
| FR-011 | `business-rules.md`, `data-model.md`, `integration-inventory.md` | Each entry includes concrete references |
| FR-012 | `coverage.md` | In-scope and excluded areas documented |

## User Story to Artifact Mapping

| User Story | Primary Artifact(s) |
|---|---|
| US1 Traceable Business Rules | `business-rules.md`, `open-questions.md` |
| US2 Data Model Summary | `data-model.md`, `open-questions.md` |
| US3 Integration Inventory + Open Questions | `integration-inventory.md`, `open-questions.md` |

## Business Rule to Requirement Mapping

| Rule ID | Requirement(s) |
|---|---|
| BR-001 | FR-001, FR-002, FR-011 |
| BR-002 | FR-001, FR-002, FR-011 |
| BR-003 | FR-001, FR-002, FR-011 |
| BR-004 | FR-001, FR-002, FR-011 |
| BR-005 | FR-001, FR-002, FR-011 |
| BR-006 | FR-001, FR-002, FR-011 |
| BR-007 | FR-001, FR-002, FR-011 |
| BR-008 | FR-001, FR-002, FR-011 |
| BR-009 | FR-001, FR-002, FR-003, FR-008 |
| BR-010 | FR-001, FR-002, FR-006, FR-011 |
| BR-011 | FR-001, FR-002, FR-003, FR-008 |
| BR-012 | FR-001, FR-002, FR-011 |
| BR-013 | FR-001, FR-002, FR-011 |
| BR-014 | FR-001, FR-002, FR-005, FR-011 |
