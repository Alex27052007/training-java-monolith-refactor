# Source Inventory

## Application Code

- `src/main/java/com/sourcegraph/demo/bigbadmonolith/StartupListener.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/ConnectionManager.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/LibertyConnectionManager.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/UserDAO.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/CustomerDAO.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/BillingCategoryDAO.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/dao/BillableHourDAO.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/entity/User.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/entity/Customer.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/entity/BillingCategory.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/entity/BillableHour.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/BillingService.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/DatabaseService.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/service/DataInitializationService.java`
- `src/main/java/com/sourcegraph/demo/bigbadmonolith/util/DateTimeUtils.java`

## Presentation Layer

- `src/main/webapp/index.jsp`
- `src/main/webapp/customers.jsp`
- `src/main/webapp/users.jsp`
- `src/main/webapp/categories.jsp`
- `src/main/webapp/hours.jsp`
- `src/main/webapp/reports.jsp`

## Runtime/Config

- `src/main/liberty/config/server.xml`
- `src/main/liberty/config/bootstrap.properties`
- `src/main/liberty/config/jvm.options`
- `src/main/webapp/WEB-INF/web.xml`

## Operational Scripts

- `liberty-dev.sh`
- `liberty-start.sh`
- `liberty-dev.bat`
- `liberty-start.sh`

## Build/Project Metadata

- `build.gradle`
- `settings.gradle`
- `README.md`
- `SETUP.md`
