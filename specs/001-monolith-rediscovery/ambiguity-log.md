# Ambiguity Triage Log

## Triage Fields

- `Ambiguity ID`
- `Topic`
- `Observed Signals`
- `Competing Interpretations`
- `Current Decision`
- `Escalated To`
- `Linked Open Question`
- `Status`

## Entries

### AMB-001
- Topic: Validation authority (service vs JSP)
- Observed Signals: Service has richer validation, JSP writes directly to DAO.
- Competing Interpretations:
  - Service validation is intended authoritative path.
  - JSP path is actual authoritative behavior in production.
- Current Decision: Treat as unresolved.
- Escalated To: Domain owner + maintainers.
- Linked Open Question: OQ-001
- Status: Open

### AMB-002
- Topic: Reporting data source path
- Observed Signals: Reports JSP uses app-relative Derby URL; Liberty datasource points to server output path.
- Competing Interpretations:
  - Both paths resolve to same physical DB in deployment.
  - They resolve to different DB files depending on launch mode.
- Current Decision: Treat as unresolved.
- Escalated To: Ops/Liberty runtime owner.
- Linked Open Question: OQ-002
- Status: Open

### AMB-003
- Topic: User constructor parameter order in users UI
- Observed Signals: JSP passes `(name, email)` to constructor defined as `(email, name)`.
- Competing Interpretations:
  - Bug that swaps fields.
  - Intentional field overloading with downstream correction (not visible).
- Current Decision: Treat as unresolved pending human confirmation.
- Escalated To: Legacy maintainers.
- Linked Open Question: OQ-003
- Status: Open
