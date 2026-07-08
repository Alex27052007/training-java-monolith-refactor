# Implementation Plan: Substitution Audit Over Rediscovery Artifacts

**Branch**: `[002-substitution-audit]` | **Date**: 2026-07-08 | **Spec**: `/specs/002-substitution-audit/spec.md`

**Input**: Feature specification from `/specs/002-substitution-audit/spec.md`

## Summary

Produce an audit-only substitution package that classifies legacy elements using four proposals (`keep-as-is`, `replace-with-library`, `replace-with-platform`, `retire`) and maps every decision row back to specific phase-1 rediscovery entries. The plan emphasizes coverage across five required risk domains and explicitly avoids target-architecture design.

## Technical Context

**Language/Version**: Markdown documentation artifacts (repository documentation workflow)

**Primary Dependencies**: Phase-1 rediscovery documents under `specs/001-monolith-rediscovery/`

**Storage**: File-based documentation in `specs/002-substitution-audit/`

**Testing**: Manual artifact validation using checklist/quickstart scenarios

**Target Platform**: Repository reviewers and maintainers consuming markdown artifacts

**Project Type**: Documentation/audit deliverable (no source-code implementation)

**Performance Goals**: N/A (quality and traceability focused)

**Constraints**:
- Audit only; do not design target architecture
- Every substitution row must link to specific rediscovery evidence
- Cover all required domains: home-grown equivalents, dated integrations, unfit data stores, EOL runtimes/frameworks, cloud-hostile assumptions
- Uncertain classifications must remain explicit and tied to open questions

**Scale/Scope**:
- In scope: all phase-1 rediscovery artifacts in `specs/001-monolith-rediscovery/`
- Out of scope: implementation sequencing, migration architecture, code changes

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Constitution file currently contains placeholders and no enforceable active gates.

### Pre-Phase 0 Gate Result

- Gate status: PASS (provisional)
- Reason: no enforceable constitutional rules exist in current constitution file.
- Additional guardrails applied:
  - Audit-only scope enforced
  - Evidence traceability required for every row
  - No target architecture design content allowed

### Post-Phase 1 Re-check Result

- Gate status: PASS (provisional)
- Reason: generated artifacts remain audit documentation and respect stated constraints.

## Project Structure

### Documentation (this feature)

```text
specs/002-substitution-audit/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── substitution-audit-contract.md
└── tasks.md
```

### Source Code (repository root)

```text
specs/
├── 001-monolith-rediscovery/
│   ├── business-rules.md
│   ├── data-model.md
│   ├── integration-inventory.md
│   ├── open-questions.md
│   ├── reference-catalog.md
│   └── rediscovery-package-index.md
└── 002-substitution-audit/
    ├── spec.md
    ├── plan.md
    ├── research.md
    ├── data-model.md
    ├── quickstart.md
    └── contracts/
```

**Structure Decision**: Keep work entirely in `specs/002-substitution-audit/`, using phase-1 rediscovery docs as evidence inputs. No source code folders (`src/`) are modified for this feature.

## Complexity Tracking

No constitution violations requiring justification.
