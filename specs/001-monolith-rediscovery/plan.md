# Implementation Plan: Legacy Monolith Rediscovery

**Branch**: `[001-monolith-rediscovery]` | **Date**: 2026-07-08 | **Spec**: `/specs/001-monolith-rediscovery/spec.md`

**Input**: Feature specification from `/specs/001-monolith-rediscovery/spec.md`

## Summary

Produce a documentation-only rediscovery package for the legacy Java monolith that is fully evidence-backed and reviewer-verifiable. The plan delivers five artifacts: this plan, `research.md`, `data-model.md`, `contracts/`, and `quickstart.md`. The approach is evidence-first: all claims must trace to concrete repository references, uncertain behaviors must become open questions, and no implementation changes or modernization proposals are allowed.

## Technical Context

**Language/Version**: Java 21 (project), JSP (view layer), SQL (Derby), Shell scripts (bash)

**Primary Dependencies**: Jakarta EE 10 APIs, Open Liberty Gradle plugin, Apache Derby (embedded), Joda-Time, JUnit 5

**Storage**: Apache Derby relational database (embedded URL and Liberty managed datasource)

**Testing**: JUnit 5 (existing project test framework); documentation validation via manual traceability checks

**Target Platform**: macOS/Linux development with Gradle wrapper; Open Liberty runtime at local HTTP/HTTPS ports

**Project Type**: Server-rendered Java web application (WAR) with JSP pages and DAO/service layers

**Performance Goals**: N/A for this feature (documentation/analysis only)

**Constraints**:
- No code changes
- No behavior guessing; unresolved uncertainty must be recorded as open questions
- No modernization proposals
- Every rule/invariant/integration entry requires concrete evidence references

**Scale/Scope**:
- In scope: `src/main/java`, `src/main/webapp`, `src/main/liberty/config`, startup scripts, build/runtime config files
- Out of scope: runtime production data introspection, non-repository systems not referenced by code/config

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

The current constitution file contains placeholders and no enforceable principles or gates.

### Pre-Phase 0 Gate Result

- Gate status: PASS (provisional)
- Reason: no active constitutional constraints are defined; therefore no violations can be evaluated.
- Additional safety checks applied for this feature:
  - Observational-only scope enforced
  - Evidence-required entries enforced
  - Uncertainty -> open-question rule enforced

### Post-Phase 1 Re-check Result

- Gate status: PASS (provisional)
- Reason: generated design artifacts remain documentation-only and satisfy feature constraints.

## Project Structure

### Documentation (this feature)

```text
specs/001-monolith-rediscovery/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── rediscovery-artifacts-contract.md
└── tasks.md
```

### Source Code (repository root)

```text
src/
├── main/
│   ├── java/com/sourcegraph/demo/bigbadmonolith/
│   │   ├── dao/
│   │   ├── entity/
│   │   ├── service/
│   │   └── util/
│   ├── webapp/
│   │   ├── index.jsp
│   │   ├── customers.jsp
│   │   ├── users.jsp
│   │   ├── categories.jsp
│   │   ├── hours.jsp
│   │   └── reports.jsp
│   └── liberty/config/
├── resources/
└── test/

.specify/
├── templates/
├── scripts/bash/
└── memory/
```

**Structure Decision**: Use the existing single-project monolith structure and add only feature documentation artifacts under `specs/001-monolith-rediscovery/`. No source layout changes are needed because the feature outputs are analysis documents.

## Complexity Tracking

No constitution violations detected that require justification.
