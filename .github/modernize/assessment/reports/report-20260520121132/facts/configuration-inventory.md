# Configuration & Externalized Settings Inventory

The project uses a compact configuration model based on Spring properties files plus Docker Compose environment overrides. Configuration is primarily local-file driven with no external configuration server or secret manager integration detected.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| `application.properties` | Spring runtime config | `src/main/resources/application.properties` | Default runtime settings including datasource, JPA, upload limits, logging |
| `application-docker.properties` | Spring profile runtime config | `src/main/resources/application-docker.properties` | Docker-specific datasource/logging overrides |
| `docker-compose.yml` | Container orchestration config | `docker-compose.yml` | Defines Oracle and app services, env overrides, dependency health checks |
| `pom.xml` | Build config | `pom.xml` | Defines Java version, dependencies, packaging and plugin setup |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Default Maven build | Implicit (`mvn ...`) | Build Spring Boot jar | `spring-boot-maven-plugin` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | No explicit `SPRING_PROFILES_ACTIVE` | `application.properties` | Oracle datasource URL/service name, DEBUG logging |
| docker | `SPRING_PROFILES_ACTIVE=docker` (docker-compose) | `application.properties` + `application-docker.properties` | Oracle connection variant, reduced app/web log verbosity |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `server.port` | `8080` | default,docker | application properties |
| `server.servlet.encoding.charset` | `UTF-8` | default,docker | application properties |
| `server.servlet.encoding.enabled` | `true` | default,docker | application properties |
| `server.servlet.encoding.force` | `true` | default,docker | application properties |
| `spring.datasource.url` | `jdbc:oracle:thin:@oracle-db:1521/FREEPDB1` | default | application.properties |
| `spring.datasource.url` | `jdbc:oracle:thin:@oracle-db:1521:XE` | docker | application-docker.properties |
| `spring.datasource.username` | `photoalbum` | default,docker | application properties |
| `spring.datasource.password` | `[MASKED]` | default,docker | application properties / env override |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | default,docker | application properties |
| `spring.jpa.database-platform` | `org.hibernate.dialect.OracleDialect` | default,docker | application properties |
| `spring.jpa.hibernate.ddl-auto` | `create` | default,docker | application properties |
| `spring.jpa.show-sql` | `true` | default,docker | application properties |
| `spring.jpa.properties.hibernate.format_sql` | `true` | default,docker | application properties |
| `spring.servlet.multipart.max-file-size` | `10MB` | default,docker | application properties |
| `spring.servlet.multipart.max-request-size` | `50MB` | default,docker | application properties |
| `app.file-upload.max-file-size-bytes` | `10485760` | default,docker | application properties |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default,docker | application properties |
| `app.file-upload.max-files-per-upload` | `10` | default,docker | application properties |
| `logging.level.com.photoalbum` | `DEBUG` (default), `INFO` (docker) | default,docker | profile property files |
| `logging.level.org.springframework.web` | `DEBUG` (default), `WARN` (docker) | default,docker | profile property files |
| `logging.level.org.hibernate.SQL` | `DEBUG` | docker | application-docker.properties |
| `SPRING_PROFILES_ACTIVE` | `docker` in compose | docker | docker-compose env |
| `SPRING_DATASOURCE_URL` | `jdbc:oracle:thin:@oracle-db:1521/FREEPDB1` in compose | docker | docker-compose env |
| `SPRING_DATASOURCE_USERNAME` | `photoalbum` in compose | docker | docker-compose env |
| `SPRING_DATASOURCE_PASSWORD` | `[MASKED]` in compose | docker | docker-compose env |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| photoalbum-java-app | No explicit JVM `-Xms/-Xmx` or `-D` flags detected | Not specified | 1 (docker-compose) |
| oracle-db | Oracle container defaults | Not specified | 1 (docker-compose) |

## Startup Dependency Chain

1. `oracle-db` starts first and must report healthy via container health check.
2. `photoalbum-java-app` starts after Oracle health success (`depends_on` with `condition: service_healthy`).
3. Application endpoints become available after Spring Boot initialization and datasource connectivity.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `spring.datasource.password` / `SPRING_DATASOURCE_PASSWORD` | Database credential | Properties file and docker-compose env (masked) |
| `ORACLE_PASSWORD` | Database admin credential | docker-compose env (masked) |
| `APP_USER_PASSWORD` | Application DB user credential | docker-compose env (masked) |

### Secrets Provisioning Workflow

Secrets are currently provisioned statically through property files and Docker Compose environment variables. Container startup injects credentials directly into Oracle and application runtime environments, and the Spring datasource binds them at boot. No managed identity, external vault, or RBAC-governed secret retrieval flow was detected.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Java target | 8 | `pom.xml` properties |
| Spring Boot | 2.7.18 | parent POM |
| Spring Data JPA / Hibernate | Boot-managed (Hibernate 5.6.x) | starter dependencies |
| Thymeleaf | Boot-managed | starter dependency |
| Oracle JDBC driver | Boot-managed `ojdbc8` | runtime dependency |
| Maven build plugin | Spring Boot Maven Plugin (managed) | `pom.xml` |
| Oracle container image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
