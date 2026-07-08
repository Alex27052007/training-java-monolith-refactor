# Integration Inventory

## Categories

- External System
- Filesystem Path
- Hard-coded Endpoint
- Runtime Dependency
- Configuration Dependency

## Inventory Entries

### INT-001
- Category: Runtime Dependency
- Description: Embedded Derby JDBC connection used by `ConnectionManager`.
- Location: `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/ConnectionManager.java:9`
- Hard-coded value: `jdbc:derby:./data/bigbadmonolith;create=true`
- Operational impact: Determines local database location and creation behavior.

### INT-002
- Category: Configuration Dependency
- Description: Liberty datasource JNDI name looked up at runtime.
- Location:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/LibertyConnectionManager.java:11`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/LibertyConnectionManager.java:25`
  - `src/main/liberty/config/server.xml:53`
- Hard-coded value: `jdbc/DefaultDataSource`
- Operational impact: Selects managed datasource path vs embedded fallback.

### INT-003
- Category: Filesystem Path
- Description: Liberty Derby DB path under server output directory.
- Location:
  - `src/main/liberty/config/server.xml:55`
  - `src/main/liberty/config/bootstrap.properties:12`
- Hard-coded value: `${server.output.dir}/data/bigbadmonolith`
- Operational impact: Runtime data location differs from relative app path.

### INT-004
- Category: Filesystem Path
- Description: JSP reporting page uses app-relative Derby path.
- Location: `src/main/webapp/reports.jsp:20`
- Hard-coded value: `jdbc:derby:./data/bigbadmonolith;create=true`
- Operational impact: May target different DB location than Liberty datasource path.

### INT-005
- Category: Runtime Dependency
- Description: Reports JSP loads Derby embedded driver directly.
- Location:
  - `src/main/webapp/reports.jsp:136`
  - `src/main/webapp/reports.jsp:232`
  - `src/main/webapp/reports.jsp:310`
- Hard-coded value: `org.apache.derby.jdbc.EmbeddedDriver`
- Operational impact: Reporting functionality bypasses DAO/service connection abstraction.

### INT-006
- Category: Hard-coded Endpoint
- Description: Liberty HTTP/HTTPS ports are configured in server config.
- Location:
  - `src/main/liberty/config/server.xml:17`
  - `src/main/liberty/config/server.xml:18`
  - `src/main/liberty/config/bootstrap.properties:4`
  - `src/main/liberty/config/bootstrap.properties:5`
- Hard-coded value: `9080`, `9443`
- Operational impact: Fixed ports for app access and TLS endpoint.

### INT-007
- Category: Hard-coded Endpoint
- Description: Web application context root path.
- Location: `src/main/liberty/config/server.xml:26`
- Hard-coded value: `/big-bad-monolith`
- Operational impact: Defines base URL path for all JSP routes.

### INT-008
- Category: Hard-coded Endpoint
- Description: Startup scripts print fixed local access endpoint.
- Location:
  - `liberty-dev.sh:20`
  - `liberty-start.sh:30`
- Hard-coded value: `http://localhost:9080/big-bad-monolith`
- Operational impact: Operational docs/scripts assume local host and fixed port.

### INT-009
- Category: Configuration Dependency
- Description: Basic user registry and credential are configured directly in Liberty config.
- Location:
  - `src/main/liberty/config/server.xml:35`
  - `src/main/liberty/config/server.xml:36`
- Hard-coded value: `user1/password`
- Operational impact: Default auth identity available in configured environment.

### INT-010
- Category: Configuration Dependency
- Description: Embedded Derby datasource credentials in server config.
- Location:
  - `src/main/liberty/config/server.xml:57`
  - `src/main/liberty/config/server.xml:58`
- Hard-coded value: `app/app`
- Operational impact: Shared static DB credentials in config.

### INT-011
- Category: Runtime Dependency
- Description: Liberty fallback to embedded Derby if JNDI initialization fails.
- Location:
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/LibertyConnectionManager.java:14`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/LibertyConnectionManager.java:34`
  - `src/main/java/com/sourcegraph/demo/bigbadmonolith/StartupListener.java:21`
- Hard-coded value: fallback behavior controlled by datasource availability
- Operational impact: Environment-dependent DB path and behavior.

### INT-012
- Category: Runtime Dependency
- Description: Build and server lifecycle commands hard-coded in startup scripts.
- Location:
  - `liberty-dev.sh:11`
  - `liberty-dev.sh:31`
  - `liberty-start.sh:11`
  - `liberty-start.sh:25`
- Hard-coded value: `./gradlew clean build`, `./gradlew libertyDev`, `./gradlew libertyStart`
- Operational impact: Scripts assume Gradle wrapper and Liberty plugin tasks.

### INT-013
- Category: Configuration Dependency
- Description: Production startup script mutates JVM debug options via inline sed commands.
- Location:
  - `liberty-start.sh:20`
  - `liberty-start.sh:21`
  - `liberty-start.sh:38`
  - `liberty-start.sh:39`
- Hard-coded value: in-place edits to `src/main/liberty/config/jvm.options`
- Operational impact: Runtime script modifies tracked config file state.

## Completeness Notes

- Inventory includes database, config, runtime scripts, endpoints, and filesystem paths in scope.
- Potentially ambiguous precedence and environment-specific behavior is captured in `open-questions.md`.
