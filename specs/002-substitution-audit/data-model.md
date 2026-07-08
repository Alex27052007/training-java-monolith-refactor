# Data Model: Substitution Audit Artifacts

## 1) Core Entities

### SubstitutionAuditRow
- Purpose: Represents one classified legacy element from rediscovery.
- Fields:
  - `rowId` (string, required)
  - `legacyElement` (string, required)
  - `proposal` (enum, required: `keep-as-is` | `replace-with-library` | `replace-with-platform` | `retire`)
  - `reason` (string, required)
  - `tradeOff` (string, required)
  - `evidenceLinks` (list of rediscovery entry links, required, min 1)
  - `coverageDomain` (enum, required)
  - `confidence` (enum: `high` | `medium` | `low`, required)
  - `openQuestionRefs` (list, optional)

### CoverageDomainSection
- Purpose: Groups rows by mandatory coverage domain.
- Fields:
  - `domainId` (enum, required)
  - `domainName` (string, required)
  - `rows` (list of `SubstitutionAuditRow`, may be empty if documented null finding)
  - `nullFindingRationale` (string, conditionally required when rows empty)

### EvidenceLink
- Purpose: Provides traceability from audit row to rediscovery evidence.
- Fields:
  - `artifactPath` (string, required; must reference `specs/001-monolith-rediscovery/*`)
  - `entryLocator` (string, required; row/rule/question identifier or heading)
  - `contextNote` (string, optional)

### UncertaintyNote
- Purpose: Records residual uncertainty impacting row confidence.
- Fields:
  - `noteId` (string, required)
  - `rowId` (string, required)
  - `openQuestionRef` (string, required)
  - `impact` (string, required)

## 2) Enumerations

### ProposalEnum
- `keep-as-is`
- `replace-with-library`
- `replace-with-platform`
- `retire`

### CoverageDomainEnum
- `home-grown-modern-equivalent`
- `dated-integration`
- `unfit-data-store`
- `eol-runtime-framework`
- `cloud-hostile-assumption`

## 3) Relationships

- One `CoverageDomainSection` contains many `SubstitutionAuditRow` records.
- One `SubstitutionAuditRow` references one or more `EvidenceLink` records.
- One `SubstitutionAuditRow` may reference zero or more `UncertaintyNote` records.
- One `UncertaintyNote` must reference exactly one open question from rediscovery artifacts.

## 4) Validation Rules

- `proposal` must use `ProposalEnum` values only.
- `legacyElement`, `reason`, and `tradeOff` must be non-empty.
- At least one `evidenceLink` is required for each row.
- `evidenceLinks.artifactPath` must point to phase-1 rediscovery docs.
- Every required `CoverageDomainEnum` value must appear at least once as a section.
- If a section has no rows, `nullFindingRationale` must explain evidence-based absence.
- `openQuestionRefs` can be present only when matching entries exist in rediscovery open questions.

## 5) State/Progress Model

### Row lifecycle
- `identified` -> `classified` -> `trace-verified` -> `review-ready`

### Section lifecycle
- `not-started` -> `populated` -> `coverage-verified` -> `review-ready`

The lifecycle is for audit completeness tracking only and does not represent implementation phases.
