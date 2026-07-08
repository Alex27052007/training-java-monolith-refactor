# Behavior Open Questions

## OQ-001
- Unknown: Which behavior is authoritative when service-layer validation conflicts with JSP direct DAO writes?
- Why code/config evidence is insufficient: `BillingService.validateBillableHour` defines strict checks, but `hours.jsp` persists directly through DAO without invoking service validation.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:89`
  - `src/main/webapp/hours.jsp:60`
- Required clarification source: Product/domain owner familiar with production data-entry path.
- Blocking level: High

## OQ-002
- Unknown: In production, does reporting use the same physical database as DAO/service operations?
- Why code/config evidence is insufficient: Reports JSP uses `jdbc:derby:./data/...` while Liberty config uses `${server.output.dir}/data/...` datasource path.
- Evidence:
  - `src/main/webapp/reports.jsp:20`
  - `src/main/liberty/config/server.xml:55`
- Required clarification source: Runtime environment configuration owner (Liberty ops).
- Blocking level: High

## OQ-003
- Unknown: Is `new User(name, email)` in `users.jsp` a known bug, intentional mapping, or unused path?
- Why code/config evidence is insufficient: Constructor signature is `(email, name)` and no transformation layer is present.
- Evidence:
  - `src/main/webapp/users.jsp:15`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/entity/User.java:10`
- Required clarification source: Original maintainer of users UI or DB data audit.
- Blocking level: High

## OQ-004
- Unknown: Expected behavior for months with fewer than 31 days in monthly reports.
- Why code/config evidence is insufficient: JSP constructs end date with `-31` for all months, but runtime behavior depends on date conversion and query execution path.
- Evidence:
  - `src/main/webapp/reports.jsp:236`
  - `src/main/webapp/reports.jsp:250`
- Required clarification source: Business stakeholder validating monthly statement semantics.
- Blocking level: Medium

## OQ-005
- Unknown: Should weekend warning be user-blocking, advisory-only, or ignored in production process?
- Why code/config evidence is insufficient: Service emits warning text but UI path may bypass service validation entirely.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java:119`
  - `src/main/webapp/hours.jsp:60`
- Required clarification source: Operations/billing policy owner.
- Blocking level: Medium

## OQ-006
- Unknown: Which startup mode is used in production: Liberty-managed datasource, embedded mode fallback, or both by environment?
- Why code/config evidence is insufficient: Startup listener branches based on datasource availability at runtime; repository lacks deployment profile matrix.
- Evidence:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/StartupListener.java:18`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/StartupListener.java:21`
- Required clarification source: Deployment/runbook owner.
- Blocking level: Medium

## OQ-007
- Unknown: Are default credentials (`user1/password`, `app/app`) active beyond local development?
- Why code/config evidence is insufficient: Config includes these values, but repository does not define environment overrides used in deployed environments.
- Evidence:
  - `src/main/liberty/config/server.xml:36`
  - `src/main/liberty/config/server.xml:57`
  - `src/main/liberty/config/server.xml:58`
- Required clarification source: Security/compliance owner and deployment config.
- Blocking level: High

## OQ-008
- Unknown: Is script-based mutation of `jvm.options` acceptable operational practice for production startup?
- Why code/config evidence is insufficient: Script edits tracked config in place and partially reverts only on failure path.
- Evidence:
  - `liberty-start.sh:20`
  - `liberty-start.sh:21`
  - `liberty-start.sh:38`
  - `liberty-start.sh:39`
- Required clarification source: Platform/operations team.
- Blocking level: Low
