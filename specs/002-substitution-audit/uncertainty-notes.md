# Uncertainty Notes

## Purpose

Track substitutions affected by unresolved rediscovery ambiguities and tie them to explicit open questions.

## Open-Question Cross-Reference Map

Source: `specs/001-monolith-rediscovery/open-questions.md`

| Open Question | Blocking Level | Affected audit row(s) | Confidence | Handling note |
|---------------|----------------|-----------------------|------------|---------------|
| OQ-001 | High | _TBD_ | _TBD_ | Service-vs-JSP validation path conflict must be cited in reason/trade-off. |
| OQ-002 | High | _TBD_ | _TBD_ | Reporting DB path mismatch requires explicit caveat in row evidence. |
| OQ-003 | High | _TBD_ | _TBD_ | User constructor argument ambiguity may change substitution confidence. |
| OQ-004 | Medium | _TBD_ | _TBD_ | Month-end date behavior uncertainty must be documented as risk note. |
| OQ-005 | Medium | _TBD_ | _TBD_ | Weekend warning semantics unresolved; preserve uncertainty marker. |
| OQ-006 | Medium | _TBD_ | _TBD_ | Runtime mode variance affects operational substitution assumptions. |
| OQ-007 | High | _TBD_ | _TBD_ | Default credentials exposure must be treated as unresolved governance risk. |
| OQ-008 | Low | _TBD_ | _TBD_ | Script mutation practice uncertainty should be captured as ops-risk caveat. |

## Confidence Scale

- High: strong evidence, low ambiguity
- Medium: evidence present, but one unresolved open question affects certainty
- Low: multiple unresolved open questions materially affect classification confidence

## Usage Rules

- Every uncertain row in `substitution-audit.md` must reference one or more OQ IDs from this file.
- Keep notes architecture-agnostic and audit-only.
