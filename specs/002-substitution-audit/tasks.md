# Tasks: Substitution Audit Over Rediscovery Artifacts

**Input**: Design documents from /specs/002-substitution-audit/

**Prerequisites**: plan.md (required), spec.md (required), research.md, data-model.md, contracts/

**Tests**: No explicit automated test requirement in the specification; validation tasks are documentation and review checks.

**Organization**: Tasks are grouped by user story to ensure independent execution and validation.

## Phase 1: Setup (Project Initialization)

**Purpose**: Create core audit work files and shared structure.

- [ ] T001 Create substitution audit working file at specs/002-substitution-audit/substitution-audit.md
- [ ] T002 Create domain coverage summary file at specs/002-substitution-audit/coverage.md
- [ ] T003 [P] Create traceability helper matrix at specs/002-substitution-audit/traceability-matrix.md
- [ ] T004 [P] Create confidence and uncertainty log at specs/002-substitution-audit/uncertainty-notes.md

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Prepare shared evidence inputs and row taxonomy before user-story work.

**⚠️ CRITICAL**: User story tasks should not begin until this phase is complete.

- [x] T005 Compile authoritative rediscovery source list in specs/002-substitution-audit/traceability-matrix.md from specs/001-monolith-rediscovery/rediscovery-package-index.md
- [x] T006 Define and document allowed proposal enum usage in specs/002-substitution-audit/substitution-audit.md using specs/002-substitution-audit/contracts/substitution-audit-contract.md
- [x] T007 [P] Define required coverage domains and section skeleton in specs/002-substitution-audit/coverage.md
- [x] T008 [P] Create row template with required columns in specs/002-substitution-audit/substitution-audit.md
- [x] T009 Build initial open-question cross-reference map in specs/002-substitution-audit/uncertainty-notes.md from specs/001-monolith-rediscovery/open-questions.md
- [x] T010 Create FR-to-artifact mapping rows FR-001..FR-012 in specs/002-substitution-audit/traceability-matrix.md

**Checkpoint**: Foundations complete; user-story execution can begin.

---

## Phase 3: User Story 1 - Traceable Substitution Decisions (Priority: P1) 🎯 MVP

**Goal**: Produce substitution-audit rows with required columns and direct links to rediscovery entries.

**Independent Test**: Sample any 10 rows in specs/002-substitution-audit/substitution-audit.md and confirm proposal, reason, trade-off, and rediscovery link are present and valid.

### Implementation for User Story 1

- [ ] T011 [P] [US1] Extract candidate legacy elements from specs/001-monolith-rediscovery/business-rules.md into specs/002-substitution-audit/substitution-audit.md
- [ ] T012 [P] [US1] Extract candidate legacy elements from specs/001-monolith-rediscovery/data-model.md into specs/002-substitution-audit/substitution-audit.md
- [ ] T013 [P] [US1] Extract candidate legacy elements from specs/001-monolith-rediscovery/integration-inventory.md into specs/002-substitution-audit/substitution-audit.md
- [ ] T014 [US1] Classify extracted rows with allowed proposal enum in specs/002-substitution-audit/substitution-audit.md
- [ ] T015 [US1] Add reason and trade-off text for each classified row in specs/002-substitution-audit/substitution-audit.md
- [ ] T016 [US1] Attach rediscovery entry links for every row in specs/002-substitution-audit/substitution-audit.md
- [ ] T017 [US1] Link row IDs to FR mappings in specs/002-substitution-audit/traceability-matrix.md

**Checkpoint**: US1 is independently reviewable and provides MVP value.

---

## Phase 4: User Story 2 - Complete Coverage Across Required Risk Domains (Priority: P2)

**Goal**: Ensure complete, explicit coverage across all required substitution-audit domains.

**Independent Test**: Verify all five required domains are present in specs/002-substitution-audit/coverage.md and each domain has rows or evidence-based null findings with rediscovery links.

### Implementation for User Story 2

- [ ] T018 [P] [US2] Populate home-grown-modern-equivalent findings in specs/002-substitution-audit/coverage.md using specs/001-monolith-rediscovery/business-rules.md and specs/001-monolith-rediscovery/reference-catalog.md
- [ ] T019 [P] [US2] Populate dated-integration findings in specs/002-substitution-audit/coverage.md using specs/001-monolith-rediscovery/integration-inventory.md
- [ ] T020 [P] [US2] Populate unfit-data-store findings in specs/002-substitution-audit/coverage.md using specs/001-monolith-rediscovery/integration-inventory.md and specs/001-monolith-rediscovery/open-questions.md
- [ ] T021 [P] [US2] Populate eol-runtime-framework findings in specs/002-substitution-audit/coverage.md using build/runtime references from specs/001-monolith-rediscovery/reference-catalog.md
- [ ] T022 [P] [US2] Populate cloud-hostile-assumption findings in specs/002-substitution-audit/coverage.md using specs/001-monolith-rediscovery/integration-inventory.md and specs/001-monolith-rediscovery/open-questions.md
- [ ] T023 [US2] Add explicit null-finding rationale where any domain has no qualifying rows in specs/002-substitution-audit/coverage.md
- [ ] T024 [US2] Add per-domain rediscovery links and update FR-005..FR-009 mappings in specs/002-substitution-audit/traceability-matrix.md

**Checkpoint**: US2 is independently reviewable with complete domain coverage.

---

## Phase 5: User Story 3 - Audit-Only Governance (Priority: P3)

**Goal**: Enforce audit-only boundaries and explicit uncertainty handling without architecture design content.

**Independent Test**: Review substitution-audit and coverage artifacts and confirm no target architecture or migration design content exists, while uncertain rows include open-question references.

### Implementation for User Story 3

- [ ] T025 [P] [US3] Annotate rows affected by unresolved rediscovery questions in specs/002-substitution-audit/uncertainty-notes.md using specs/001-monolith-rediscovery/open-questions.md
- [ ] T026 [US3] Add row-level confidence markers and open-question references in specs/002-substitution-audit/substitution-audit.md
- [ ] T027 [US3] Run audit-only boundary pass and remove architecture-design wording from specs/002-substitution-audit/substitution-audit.md
- [ ] T028 [US3] Run audit-only boundary pass and remove architecture-design wording from specs/002-substitution-audit/coverage.md
- [ ] T029 [US3] Update governance traceability for FR-010, FR-011, and FR-012 in specs/002-substitution-audit/traceability-matrix.md

**Checkpoint**: US3 is independently reviewable and governance-compliant.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final consistency checks and reviewer handoff packaging.

- [ ] T030 [P] Run quickstart validation scenarios and capture results in specs/002-substitution-audit/validation-report.md
- [ ] T031 [P] Execute contract conformance review and capture results in specs/002-substitution-audit/quality-gate-report.md
- [ ] T032 Consolidate final reviewer handoff index in specs/002-substitution-audit/substitution-audit-index.md
- [ ] T033 Final pass: verify every substitution row links to rediscovery evidence in specs/002-substitution-audit/substitution-audit.md

---

## Dependencies & Execution Order

### Phase Dependencies

- Phase 1 (Setup): no dependencies
- Phase 2 (Foundational): depends on Phase 1 and blocks all user stories
- Phase 3+ (User Stories): depend on Phase 2 completion
- Phase 6 (Polish): depends on completion of desired user stories

### User Story Dependencies

- US1 (P1): can start after Phase 2 and provides MVP
- US2 (P2): can start after Phase 2; independent of US1 implementation details but uses shared row taxonomy
- US3 (P3): can start after Phase 2; independent except for shared artifact files

### Story Completion Order

1. US1
2. US2
3. US3

---

## Parallel Opportunities

- Phase 1: T003 and T004 parallel after T001 and T002
- Phase 2: T007 and T008 parallel; T009 and T010 parallel after T005 and T006
- US1: T011, T012, T013 parallel; merge in T014-T017
- US2: T018-T022 parallel by domain; merge in T023-T024
- US3: T025 parallel with early governance scan, then T026-T029
- Phase 6: T030 and T031 parallel, then T032-T033

---

## Parallel Example: User Story 1

- Task: T011 in specs/002-substitution-audit/substitution-audit.md
- Task: T012 in specs/002-substitution-audit/substitution-audit.md
- Task: T013 in specs/002-substitution-audit/substitution-audit.md

These can be done in parallel by evidence source (business rules, data model, integrations) and merged for unified classification.

## Parallel Example: User Story 2

- Task: T018 in specs/002-substitution-audit/coverage.md
- Task: T019 in specs/002-substitution-audit/coverage.md
- Task: T022 in specs/002-substitution-audit/coverage.md

These can run in parallel by domain section owners.

## Parallel Example: User Story 3

- Task: T025 in specs/002-substitution-audit/uncertainty-notes.md
- Task: T027 in specs/002-substitution-audit/substitution-audit.md
- Task: T028 in specs/002-substitution-audit/coverage.md

These can run in parallel before final governance traceability update.

---

## Implementation Strategy

### MVP First (US1 only)

1. Complete Setup and Foundational phases.
2. Complete US1 tasks (T011-T017).
3. Validate independent test for row-level traceability.
4. Stop for reviewer check before expanding coverage.

### Incremental Delivery

1. Deliver US1 traceable substitution table.
2. Add US2 domain coverage sections and validate.
3. Add US3 audit-governance and uncertainty handling.
4. Complete polish validation and handoff artifacts.

### Parallel Team Strategy

1. One owner completes Setup/Foundational tasks.
2. After Phase 2:
   - Reviewer A owns US1 row extraction/classification.
   - Reviewer B owns US2 domain coverage.
   - Reviewer C owns US3 governance/uncertainty checks.
3. Rejoin for final quality-gate and handoff tasks.

---

## Notes

- All tasks are documentation-only and preserve audit-only constraint.
- User story tasks are independently testable.
- No architecture design tasks are included by design.
