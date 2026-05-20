# Configuration & Externalized Settings Inventory

This document inventories the configuration landscape for the Photo Album project, including Spring properties, container runtime settings, and dependency/runtime version declarations. Configuration is primarily file-based with environment variable overrides used in Docker Compose.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Spring base config | Properties file | `src/main/resources/application.properties` | Default runtime settings including datasource and upload constraints |
| Spring profile config | Properties file | `src/main/resources/application-docker.properties` | Overrides for docker profile runtime |
| Container orchestration config | YAML | `docker-compose.yml` | Service definitions, environment variables, health checks, dependency order |
| Build and dependency config | Maven POM | `pom.xml` | Runtime versions, dependencies, and build plugin declaration |
| Container image config | Dockerfile | `Dockerfile` | Runtime packaging and startup command context |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Default Maven build | `mvn` invocation | Compile/package Spring Boot app | `spring-boot-maven-plugin` |

No explicit Maven `<profiles>` were detected in `pom.xml`.

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | implicit Spring startup | `application.properties` | Oracle datasource (`FREEPDB1`), upload limits, debug logging |
| docker | `SPRING_PROFILES_ACTIVE=docker` in Compose | `application.properties` + `application-docker.properties` | Docker Oracle URL (`XE`), adjusted logging levels |

## Properties Inventory

### Core application properties

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `server.port` | `8080` | default,docker | properties file |
| `server.servlet.encoding.charset` | `UTF-8` | default,docker | properties file |
| `server.servlet.encoding.enabled` | `true` | default,docker | properties file |
| `server.servlet.encoding.force` | `true` | default,docker | properties file |

### Database properties

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `spring.datasource.url` | `jdbc:oracle:thin:@oracle-db:1521/FREEPDB1` | default | properties file / env override available |
| `spring.datasource.url` | `jdbc:oracle:thin:@oracle-db:1521:XE` | docker | profile properties / env override available |
| `spring.datasource.username` | `photoalbum` | default,docker | properties file / env override in Compose |
| `spring.datasource.password` | `[MASKED]` | default,docker | properties file / env override in Compose |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | default,docker | properties file |
| `spring.jpa.database-platform` | `org.hibernate.dialect.OracleDialect` | default,docker | properties file |
| `spring.jpa.hibernate.ddl-auto` | `create` | default,docker | properties file |
| `spring.jpa.show-sql` | `true` | default,docker | properties file |
| `spring.jpa.properties.hibernate.format_sql` | `true` | default,docker | properties file |

### File upload and logging properties

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `spring.servlet.multipart.max-file-size` | `10MB` | default,docker | properties file |
| `spring.servlet.multipart.max-request-size` | `50MB` | default,docker | properties file |
| `app.file-upload.max-file-size-bytes` | `10485760` | default,docker | properties file |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default,docker | properties file |
| `app.file-upload.max-files-per-upload` | `10` | default,docker | properties file |
| `logging.level.com.photoalbum` | `DEBUG` default, `INFO` docker | default,docker | properties file |
| `logging.level.org.springframework.web` | `DEBUG` default, `WARN` docker | default,docker | properties file |
| `logging.level.org.hibernate.SQL` | `DEBUG` | docker | profile properties |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| photoalbum-java-app | `SPRING_PROFILES_ACTIVE=docker`; datasource env overrides in Compose | Not explicitly configured | 1 (Compose service) |
| oracle-db | Oracle image environment variables for app user/password | Not explicitly configured | 1 (Compose service) |

## Startup Dependency Chain

1. `oracle-db` container starts first and must pass its health check (`healthcheck.sh`).
2. `photoalbum-java-app` waits on `oracle-db` readiness via `depends_on: condition: service_healthy`.
3. Once Oracle is healthy, the Spring Boot application starts and opens port 8080.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `spring.datasource.password` / `SPRING_DATASOURCE_PASSWORD` | Database password | Plain property/env value (`[MASKED]`) |
| `ORACLE_PASSWORD` | Database admin/user initialization secret | Docker Compose environment variable (`[MASKED]`) |
| `APP_USER_PASSWORD` | Application DB user password | Docker Compose environment variable (`[MASKED]`) |

### Secrets Provisioning Workflow

Secrets are currently supplied directly through static property files and Docker Compose environment variables. During startup, Oracle credentials initialize the database container and application credentials are injected into the Spring Boot container via environment variables. No managed identity integration or external secret vault binding was detected.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Java | 8 | `pom.xml` (`java.version`) |
| Spring Boot | 2.7.18 | Maven parent POM |
| Spring Data JPA | managed by Spring Boot 2.7.18 | Maven dependency management |
| Thymeleaf starter | managed by Spring Boot 2.7.18 | Maven dependency management |
| Oracle JDBC (`ojdbc8`) | managed by Spring Boot BOM | Maven dependency declaration |
| Commons IO | 2.11.0 | Maven dependency declaration |
| Docker Oracle image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
| Maven build plugin | Spring Boot Maven Plugin | `pom.xml` |
