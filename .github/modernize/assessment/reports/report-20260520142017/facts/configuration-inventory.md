# Configuration & Externalized Settings Inventory

The configuration surface is compact and primarily file-based, with profile-specific property overrides and container-provided environment variables for database connectivity. Sensitive values are currently configured directly in local property files and compose definitions.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Spring application properties | Runtime config | `src/main/resources/application.properties` | Default profile settings including datasource and upload limits |
| Spring profile properties | Runtime config | `src/main/resources/application-docker.properties` | Docker profile Oracle and logging overrides |
| Docker Compose | Deployment/runtime config | `docker-compose.yml` | Defines Oracle and app services plus environment variables |
| Maven POM | Build config | `pom.xml` | Java version, dependencies, and plugin configuration |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| default Maven build | automatic | Build Spring Boot executable jar | `spring-boot-maven-plugin` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | implicit when no profile set | `application.properties` | Oracle datasource URL, JPA create ddl mode, debug logging |
| docker | `SPRING_PROFILES_ACTIVE=docker` | `application-docker.properties` + env vars | Docker Oracle URL format, INFO/WARN logging defaults |

## Properties Inventory

### photo-album

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `server.port` | `8080` | default,docker | application properties |
| `spring.datasource.url` | `jdbc:oracle:thin:@oracle-db:1521/FREEPDB1` (default), `...:1521:XE` (docker) | default,docker | application properties / docker profile |
| `spring.datasource.username` | `photoalbum` | default,docker | application properties |
| `spring.datasource.password` | `[MASKED]` | default,docker | application properties |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | default,docker | application properties |
| `spring.jpa.database-platform` | `org.hibernate.dialect.OracleDialect` | default,docker | application properties |
| `spring.jpa.hibernate.ddl-auto` | `create` | default,docker | application properties |
| `spring.jpa.show-sql` | `true` | default,docker | application properties |
| `spring.servlet.multipart.max-file-size` | `10MB` | default,docker | application properties |
| `spring.servlet.multipart.max-request-size` | `50MB` | default,docker | application properties |
| `app.file-upload.max-file-size-bytes` | `10485760` | default,docker | application properties |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default,docker | application properties |
| `app.file-upload.max-files-per-upload` | `10` | default,docker | application properties |
| `logging.level.com.photoalbum` | `DEBUG` (default), `INFO` (docker) | default,docker | application properties |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| photo-album | No explicit JVM flags in repo; profile via `SPRING_PROFILES_ACTIVE` | Not explicitly configured | 1 (docker-compose single service) |
| oracle-db | Container env for DB bootstrap | Not explicitly configured | 1 |

## Startup Dependency Chain

1. `oracle-db` starts first and reports healthy via container healthcheck.
2. `photoalbum-java-app` waits on `oracle-db` health (`depends_on` condition: service_healthy).
3. Application startup continues with Spring datasource initialization and request handling on port 8080.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `spring.datasource.password` | Database password | `application.properties` / `application-docker.properties` (`[MASKED]`) |
| `ORACLE_PASSWORD` | Database admin password | `docker-compose.yml` environment (`[MASKED]`) |
| `APP_USER_PASSWORD` | Application DB user password | `docker-compose.yml` environment (`[MASKED]`) |

### Secrets Provisioning Workflow

Secrets are currently supplied through local static configuration files and Docker Compose environment variables. There is no managed identity, key vault integration, or external secret manager configured in the repository. The app service consumes datasource credentials directly at startup to establish Oracle connections.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| No dedicated feature flag framework detected | n/a | n/a |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Spring Boot | 2.7.18 | `pom.xml` parent |
| Java target | 1.8 | `pom.xml` properties |
| Maven compiler target/source | 8 | `pom.xml` properties |
| Oracle JDBC | ojdbc8 (managed by Boot BOM) | `pom.xml` dependency |
| Commons IO | 2.11.0 | `pom.xml` dependency |
| Docker Oracle image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
