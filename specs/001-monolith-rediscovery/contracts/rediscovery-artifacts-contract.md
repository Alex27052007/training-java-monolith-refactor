# Contract: Rediscovery Artifacts

## Purpose

Define the required structure, fields, and validation rules for rediscovery deliverables so reviewers can validate every claim against concrete evidence.

## Artifact Set

Required outputs:
- `business-rules.md`
- `data-model.md`
- `integration-inventory.md`
- `open-questions.md`

This contract defines what each file must contain.

## 1) Business Rules Contract

Each rule entry MUST include:
- `Rule ID`: stable identifier (e.g., BR-001)
- `Rule Statement`: plain English behavioral claim
- `Evidence References`: one or more concrete repository references
- `Concrete Example`: one realistic scenario or data example
- `Confidence`: `Observed` or `Unknown` (never inferred-as-fact)
- `Notes`: optional clarifications

Validation rules:
- At least one evidence reference is required.
- At least one concrete example is required.
- If confidence is not `Observed`, the item MUST be mirrored in `open-questions.md`.

## 2) Data Model Contract

The data model output MUST include:
- Entity list with fields and types
- Relationship list with cardinality and optionality
- Invariant matrix with enforcement source classification

Invariant classification enum:
- `DB Constraint`
- `Defensive Code`
- `Both`
- `Not Enforced`

Validation rules:
- Every invariant MUST include enforcement source and evidence reference.
- Relationships MUST identify participating entities and direction/cardinality.

## 3) Integration Inventory Contract

Each inventory entry MUST include:
- `Integration ID`
- `Category` (enum below)
- `Description`
- `Location` (path and line-level evidence where possible)
- `Hard-coded Value` (if applicable)
- `Operational Impact`

Category enum:
- `External System`
- `Filesystem Path`
- `Hard-coded Endpoint`
- `Runtime Dependency`
- `Configuration Dependency`

Validation rules:
- Every discovered touchpoint in scope must appear once.
- Duplicate entries for same touchpoint are not allowed.

## 4) Open Questions Contract

Each open question MUST include:
- `Question ID`
- `Unknown`
- `Why code/config evidence is insufficient`
- `Required clarification source` (person, environment, or missing artifact)
- `Blocking level` (`High`, `Medium`, `Low`)

Validation rules:
- Any unresolved uncertainty from rule/model/inventory artifacts must be listed.
- No guessed behavior can replace an open question.

## 5) Evidence Reference Format

Recommended format:
- `path/to/file.ext:line`

Alternative accepted format:
- `path/to/file.ext#Lline`

Validation rules:
- References must resolve to existing repository files.
- References should point to the smallest useful location proving the claim.

## 6) Coverage Statement

The final package MUST include a coverage statement listing:
- Analyzed areas
- Excluded areas
- Reason for exclusions

## 7) Non-Goals / Prohibitions

- No code changes
- No modernization recommendations
- No unverifiable behavioral assertions

## 8) Reviewer Acceptance Checklist

For review sign-off, the following must all be true:

- Business rules contain plain-English statements, concrete references, and examples.
- Data model lists entities, relationships, and invariants with enforcement source.
- Integration inventory includes filesystem paths, runtime/config dependencies, and endpoints.
- Open questions capture every unresolved uncertainty with explicit closure path.
- Coverage statement identifies analyzed and excluded areas.
- No guessed behavior is presented as confirmed fact.
