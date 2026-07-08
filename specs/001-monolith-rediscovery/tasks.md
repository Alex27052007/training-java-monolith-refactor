# Tasks: Legacy Monolith Rediscovery

**Input**: Design documents from `/specs/001-monolith-rediscovery/`

**Prerequisites**: `plan.md` (required), `spec.md` (required), `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: No explicit test-automation requirement was requested in the feature spec; this task list focuses on documentation deliverables and reviewer validation artifacts.

**Organization**: Tasks are grouped by user story to enable independent implementation and validation.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Initialize rediscovery documentation workspace and shared templates.

- [x] T001 Create rediscovery deliverable files `specs/001-monolith-rediscovery/business-rules.md`, `specs/001-monolith-rediscovery/integration-inventory.md`, `specs/001-monolith-rediscovery/open-questions.md`, and `specs/001-monolith-rediscovery/coverage.md`
- [x] T002 Create traceability map skeleton in `specs/001-monolith-rediscovery/traceability-matrix.md`
- [x] T003 [P] Create source inventory index in `specs/001-monolith-rediscovery/source-inventory.md`
- [x] T004 [P] Create evidence reference catalog scaffold in `specs/001-monolith-rediscovery/reference-catalog.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Build shared analysis foundations required by all user stories.

**⚠️ CRITICAL**: No user story documentation work should start until this phase is complete.

- [x] T005 Populate in-scope and excluded-area coverage statement in `specs/001-monolith-rediscovery/coverage.md`
- [x] T006 Create ambiguity triage log format in `specs/001-monolith-rediscovery/ambiguity-log.md`
- [x] T007 [P] Extend deliverable contract with reviewer acceptance checklist in `specs/001-monolith-rediscovery/contracts/rediscovery-artifacts-contract.md`
- [x] T008 [P] Create reviewer validation checklist in `specs/001-monolith-rediscovery/checklists/reviewer-validation.md`
- [x] T009 Build initial code/config/script reference catalog entries in `specs/001-monolith-rediscovery/reference-catalog.md`
- [x] T010 Build requirement-to-artifact mapping rows for FR-001..FR-012 in `specs/001-monolith-rediscovery/traceability-matrix.md`

**Checkpoint**: Foundations complete, user story work can proceed.

---

## Phase 3: User Story 1 - Traceable Business Rules (Priority: P1) 🎯 MVP

**Goal**: Produce a plain-English business rules document where every rule has concrete evidence and an example.

**Independent Test**: Sample any 5 rules in `specs/001-monolith-rediscovery/business-rules.md`; each must include a verifiable code reference and a concrete example.

### Implementation for User Story 1

- [x] T011 [P] [US1] Extract service-layer billing/reporting rules from `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java` into `specs/001-monolith-rediscovery/business-rules.md`
- [x] T012 [P] [US1] Extract JSP workflow and validation rules from `src/main/webapp/hours.jsp`, `src/main/webapp/reports.jsp`, `src/main/webapp/users.jsp`, and `src/main/webapp/categories.jsp` into `specs/001-monolith-rediscovery/business-rules.md`
- [x] T013 [P] [US1] Extract startup and initialization rules from `src/main/java/com/sourcegraph/demo/bigbadmonolith/StartupListener.java` and `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/DataInitializationService.java` into `specs/001-monolith-rediscovery/business-rules.md`
- [x] T014 [US1] Normalize all rule records (Rule ID, statement, evidence, example, confidence) in `specs/001-monolith-rediscovery/business-rules.md`
- [x] T015 [US1] Link each rule to FR coverage rows in `specs/001-monolith-rediscovery/traceability-matrix.md`
- [x] T016 [US1] Record unresolved rule uncertainties in `specs/001-monolith-rediscovery/open-questions.md` and `specs/001-monolith-rediscovery/ambiguity-log.md`

**Checkpoint**: User Story 1 is independently reviewable and satisfies MVP scope.

---

## Phase 4: User Story 2 - Verified Data Model Summary (Priority: P2)

**Goal**: Deliver entity, relationship, and invariant documentation with explicit enforcement-source classification.

**Independent Test**: Review all invariants in `specs/001-monolith-rediscovery/data-model.md`; each must state enforcement source (DB, defensive code, both, or not enforced) and include evidence.

### Implementation for User Story 2

- [x] T017 [P] [US2] Document entity field definitions and nullability from `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/ConnectionManager.java` and entity classes in `src/main/java/com/sourcegraph/demo/bigbadmonolith/entity/` into `specs/001-monolith-rediscovery/data-model.md`
- [x] T018 [P] [US2] Document relationship cardinality and optionality from schema FKs and reporting joins in `src/main/webapp/reports.jsp` into `specs/001-monolith-rediscovery/data-model.md`
- [x] T019 [P] [US2] Populate invariant enforcement matrix (DB vs defensive code vs both) using `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/CustomerDAO.java` and `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java` in `specs/001-monolith-rediscovery/data-model.md`
- [x] T020 [US2] Add invariants enforced only in UI or not enforced server-side using `src/main/webapp/hours.jsp` and `src/main/webapp/categories.jsp` in `specs/001-monolith-rediscovery/data-model.md`
- [x] T021 [US2] Capture unresolved data-model ambiguities in `specs/001-monolith-rediscovery/open-questions.md` and `specs/001-monolith-rediscovery/ambiguity-log.md`
- [x] T022 [US2] Update FR-004 and FR-005 traceability links in `specs/001-monolith-rediscovery/traceability-matrix.md`

**Checkpoint**: User Story 2 is independently reviewable with enforcement-source clarity.

---

## Phase 5: User Story 3 - Integration Inventory and Open Questions (Priority: P3)

**Goal**: Deliver complete integration inventory and unresolved-uncertainty capture without behavior guessing.

**Independent Test**: Search repository for integration touchpoints (`jdbc:`, `lookup(`, `http://localhost`, `/data/`) and verify each appears exactly once in `specs/001-monolith-rediscovery/integration-inventory.md`, with unresolved cases in `specs/001-monolith-rediscovery/open-questions.md`.

### Implementation for User Story 3

- [x] T023 [P] [US3] Inventory datasource/runtime dependency touchpoints from `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/LibertyConnectionManager.java`, `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/ConnectionManager.java`, and `src/main/liberty/config/server.xml` into `specs/001-monolith-rediscovery/integration-inventory.md`
- [x] T024 [P] [US3] Inventory filesystem paths and hard-coded endpoints from `liberty-dev.sh`, `liberty-start.sh`, `src/main/liberty/config/bootstrap.properties`, and `src/main/webapp/reports.jsp` into `specs/001-monolith-rediscovery/integration-inventory.md`
- [x] T025 [P] [US3] Inventory configuration dependencies and default credentials/ports from `src/main/liberty/config/server.xml` and `src/main/liberty/config/bootstrap.properties` into `specs/001-monolith-rediscovery/integration-inventory.md`
- [x] T026 [US3] Deduplicate entries and classify each inventory record by contract taxonomy in `specs/001-monolith-rediscovery/integration-inventory.md`
- [x] T027 [US3] Capture unresolved integration precedence questions (for example, JSP direct JDBC vs DAO/service paths) in `specs/001-monolith-rediscovery/open-questions.md` and `specs/001-monolith-rediscovery/ambiguity-log.md`
- [x] T028 [US3] Update FR-006..FR-009 and FR-012 coverage links in `specs/001-monolith-rediscovery/traceability-matrix.md` and `specs/001-monolith-rediscovery/coverage.md`

**Checkpoint**: User Story 3 is independently reviewable with complete integration mapping and explicit unknowns.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final quality gate and reviewer handoff across all stories.

- [x] T029 [P] Run quickstart validation scenarios and record results in `specs/001-monolith-rediscovery/validation-report.md`
- [x] T030 [P] Run contract-conformance pass across `specs/001-monolith-rediscovery/business-rules.md`, `specs/001-monolith-rediscovery/data-model.md`, `specs/001-monolith-rediscovery/integration-inventory.md`, and `specs/001-monolith-rediscovery/open-questions.md` and document outcomes in `specs/001-monolith-rediscovery/quality-gate-report.md`
- [x] T031 Finalize analyzed/excluded scope and rationale in `specs/001-monolith-rediscovery/coverage.md`
- [x] T032 Create reviewer handoff index linking all deliverables in `specs/001-monolith-rediscovery/rediscovery-package-index.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- Setup (Phase 1): no dependencies
- Foundational (Phase 2): depends on Setup completion and blocks all user stories
- User Stories (Phases 3-5): depend on Foundational completion
- Polish (Phase 6): depends on completion of all selected user stories

### User Story Dependencies

- US1 (P1): can start immediately after Phase 2 and is the MVP
- US2 (P2): can start after Phase 2; independent of US1 outputs except shared templates
- US3 (P3): can start after Phase 2; independent of US1/US2 outputs except shared templates

### Suggested Story Completion Order

1. US1 (MVP)
2. US2
3. US3

---

## Parallel Opportunities

- Setup: T003 and T004 can run in parallel after T001-T002
- Foundational: T007 and T008 can run in parallel; T009 and T010 can run in parallel after T005-T006
- US1: T011, T012, and T013 can run in parallel, then merge at T014
- US2: T017, T018, and T019 can run in parallel, then merge at T020
- US3: T023, T024, and T025 can run in parallel, then merge at T026
- Polish: T029 and T030 can run in parallel before T031-T032

---

## Parallel Example: User Story 1

- T011 [US1] in `specs/001-monolith-rediscovery/business-rules.md`
- T012 [US1] in `specs/001-monolith-rediscovery/business-rules.md`
- T013 [US1] in `specs/001-monolith-rediscovery/business-rules.md`

These tasks can be split by rule category and merged before T014 normalization.

## Parallel Example: User Story 2

- T017 [US2] in `specs/001-monolith-rediscovery/data-model.md`
- T018 [US2] in `specs/001-monolith-rediscovery/data-model.md`
- T019 [US2] in `specs/001-monolith-rediscovery/data-model.md`

These tasks can be split by entity/relationship/invariant sections and merged before T020.

## Parallel Example: User Story 3

- T023 [US3] in `specs/001-monolith-rediscovery/integration-inventory.md`
- T024 [US3] in `specs/001-monolith-rediscovery/integration-inventory.md`
- T025 [US3] in `specs/001-monolith-rediscovery/integration-inventory.md`

These tasks can be split by integration category and merged before T026 deduplication.

---

## Implementation Strategy

### MVP First (US1 only)

1. Complete Phase 1 and Phase 2.
2. Complete US1 tasks (T011-T016).
3. Validate US1 independently using its checkpoint criteria.
4. Pause for reviewer feedback before continuing.

### Incremental Delivery

1. Deliver US1 documentation package.
2. Add US2 data-model refinements and validate independently.
3. Add US3 integration/open-question inventory and validate independently.
4. Run Phase 6 quality and handoff.

### Team Parallel Strategy

1. One contributor owns shared setup/foundation tasks.
2. After Phase 2, assign:
   - Contributor A: US1
   - Contributor B: US2
   - Contributor C: US3
3. Rejoin for Phase 6 cross-cutting validation and handoff.

---

## Notes

- All tasks are documentation-only and preserve the no-code-change constraint.
- Task descriptions include explicit file paths for direct execution by an LLM.
- Story phases are independently testable by design.
