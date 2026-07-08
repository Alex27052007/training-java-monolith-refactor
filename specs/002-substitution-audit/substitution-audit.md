# Substitution Audit

## Scope

This document records substitution decisions for legacy elements discovered in phase-1 rediscovery artifacts.

Constraints:
- Audit-only output
- No target architecture design content
- Every row must include source evidence

## Proposal Enum (Authoritative)

Allowed values for `proposal`:
- `keep-as-is`
- `replace-with-library`
- `replace-with-platform`
- `retire`

Any value outside this enum is invalid.

## Row Template

Use this exact schema for every row:

| row id | legacy element | proposal | reason | trade-off | rediscovery link |
|--------|----------------|----------|--------|-----------|------------------|
| R-001 | <element name> | <keep-as-is\|replace-with-library\|replace-with-platform\|retire> | <why this proposal fits evidence> | <cost/risk/limitation> | <path:line or section ref> |

Validation rules:
- `reason` must be non-empty
- `trade-off` must be non-empty
- `rediscovery link` must point to a specific rediscovery entry

## Working Table

| row id | legacy element | proposal | reason | trade-off | rediscovery link |
|--------|----------------|----------|--------|-----------|------------------|
| R-001 | _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |
| R-002 | _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |
| R-003 | _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## Notes

- If one legacy element appears in multiple rediscovery artifacts, keep one authoritative row and list supporting links in the `rediscovery link` cell.
- For uncertain rows, add confidence/open-question references as defined in `uncertainty-notes.md`.
