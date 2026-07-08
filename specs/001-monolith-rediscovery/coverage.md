# Coverage Statement

## Analyzed Areas

- Java domain/service logic:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/entity/`
- Data access and schema bootstrap:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/`
- App lifecycle behavior:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/StartupListener.java`
- JSP behavior and direct SQL/reporting paths:
  - `src/main/webapp/index.jsp`
  - `src/main/webapp/customers.jsp`
  - `src/main/webapp/users.jsp`
  - `src/main/webapp/categories.jsp`
  - `src/main/webapp/hours.jsp`
  - `src/main/webapp/reports.jsp`
- Runtime and environment configuration:
  - `src/main/liberty/config/server.xml`
  - `src/main/liberty/config/bootstrap.properties`
  - `src/main/liberty/config/jvm.options`
- Operational scripts:
  - `liberty-dev.sh`
  - `liberty-start.sh`

## Excluded Areas

- Runtime production database contents (not available in repository)
- Production environment overlays/secrets external to repository
- External systems not referenced by source/config/scripts in this repo
- Historical branches and deployment pipelines outside checked-out workspace

## Exclusion Rationale

Phase 1 rediscovery is code-and-artifact based. Items excluded above require runtime access, external operational systems, or historical deployment context not present in repository.

## Finalization

- Finalized on: 2026-07-08
- Phase status: Rediscovery core complete and handoff package prepared
- Validation references:
  - `validation-report.md`
  - `quality-gate-report.md`
