# Traceability Matrix

## Authoritative Rediscovery Source List

Derived from `specs/001-monolith-rediscovery/rediscovery-package-index.md`.

Core deliverables:
- `specs/001-monolith-rediscovery/spec.md`
- `specs/001-monolith-rediscovery/plan.md`
- `specs/001-monolith-rediscovery/research.md`
- `specs/001-monolith-rediscovery/business-rules.md`
- `specs/001-monolith-rediscovery/data-model.md`
- `specs/001-monolith-rediscovery/integration-inventory.md`
- `specs/001-monolith-rediscovery/open-questions.md`

Validation/governance artifacts:
- `specs/001-monolith-rediscovery/coverage.md`
- `specs/001-monolith-rediscovery/ambiguity-log.md`
- `specs/001-monolith-rediscovery/traceability-matrix.md`
- `specs/001-monolith-rediscovery/reference-catalog.md`
- `specs/001-monolith-rediscovery/source-inventory.md`
- `specs/001-monolith-rediscovery/checklists/reviewer-validation.md`
- `specs/001-monolith-rediscovery/contracts/rediscovery-artifacts-contract.md`
- `specs/001-monolith-rediscovery/quickstart.md`
- `specs/001-monolith-rediscovery/validation-report.md`
- `specs/001-monolith-rediscovery/quality-gate-report.md`

## FR-to-Artifact Mapping (FR-001..FR-012)

| Requirement | Primary audit artifact | Supporting artifact(s) | Verification note |
|-------------|------------------------|-------------------------|-------------------|
| FR-001 | substitution-audit.md | quickstart.md | Table exists and is non-empty |
| FR-002 | substitution-audit.md | contracts/substitution-audit-contract.md | Required columns present in all rows |
| FR-003 | substitution-audit.md | contracts/substitution-audit-contract.md | Proposal values constrained to enum |
| FR-004 | substitution-audit.md | traceability-matrix.md | Every row links to rediscovery evidence |
| FR-005 | coverage.md | substitution-audit.md | Home-grown equivalents covered |
| FR-006 | coverage.md | substitution-audit.md | Dated integrations covered |
| FR-007 | coverage.md | substitution-audit.md | Unfit data stores covered |
| FR-008 | coverage.md | substitution-audit.md | EOL runtimes/frameworks covered |
| FR-009 | coverage.md | substitution-audit.md | Cloud-hostile assumptions covered |
| FR-010 | substitution-audit.md | coverage.md | Audit-only language; no target architecture |
| FR-011 | uncertainty-notes.md | substitution-audit.md | Uncertain rows mapped to open questions |
| FR-012 | traceability-matrix.md | substitution-audit.md | Decision-to-evidence review path is explicit |

## Row Link Mapping (to be completed in US1/US2/US3)

| row id | decision location | rediscovery evidence link(s) | covered FR(s) |
|--------|-------------------|------------------------------|---------------|
| R-001 | substitution-audit.md | _TBD_ | _TBD_ |
| R-002 | substitution-audit.md | _TBD_ | _TBD_ |
