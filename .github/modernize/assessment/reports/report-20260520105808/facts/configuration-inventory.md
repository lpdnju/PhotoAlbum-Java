# Configuration & Externalized Settings Inventory

This inventory captures the application's active configuration sources, profiles, and runtime parameters used to control database, upload, and logging behavior. The configuration approach is file-based with profile overrides and environment-variable injection in container runs.

## Configuration Sources

| Source | Type | Path/Location | Notes |
| --- | --- | --- | --- |
| `application.properties` | Spring properties | `src/main/resources/application.properties` | Default runtime configuration |
| `application-docker.properties` | Spring profile properties | `src/main/resources/application-docker.properties` | Docker profile overrides |
| `application-test.properties` | Spring test profile properties | `src/test/resources/application-test.properties` | Test-only in-memory DB configuration |
| `docker-compose.yml` | Container environment configuration | `docker-compose.yml` | Injects profile and datasource environment variables |
| Maven POM | Build and dependency configuration | `pom.xml` | Defines Java level, dependencies, and Spring Boot plugin |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
| --- | --- | --- | --- |
| default Maven build | Automatic on `mvn` commands | Compile/package Spring Boot app | `spring-boot-maven-plugin`, Java 8 compiler settings |
| Spring Boot devtools optional path | Dependency presence (`optional=true`) | Developer productivity support | `spring-boot-devtools` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
| --- | --- | --- | --- |
| default | No explicit profile or local app start | `application.properties` | Oracle datasource URL (`FREEPDB1`), DEBUG logging |
| docker | `SPRING_PROFILES_ACTIVE=docker` (compose env) | `application.properties` + `application-docker.properties` | Alternate Oracle URL (`XE`), INFO/WARN logging profile |
| test | `@ActiveProfiles("test")` in tests | `application-test.properties` | H2 datasource and `create-drop` schema mode |

## Properties Inventory

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| `server.port` | `8080` | default, docker | application properties files |
| `spring.datasource.url` | Oracle JDBC URL | default, docker, test override | profile properties + env override in compose |
| `spring.datasource.username` | `photoalbum` (default/docker), `sa` (test) | default, docker, test | profile properties |
| `spring.datasource.password` | `[MASKED]` | default, docker, test | profile properties + compose env |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` (`org.h2.Driver` in test) | default, docker, test | profile properties |
| `spring.jpa.database-platform` | Oracle dialect (`H2Dialect` in test) | default, docker, test | profile properties |
| `spring.jpa.hibernate.ddl-auto` | `create` (`create-drop` in test) | default, docker, test | profile properties |
| `spring.servlet.multipart.max-file-size` | `10MB` | default, docker | application properties files |
| `spring.servlet.multipart.max-request-size` | `50MB` | default, docker | application properties files |
| `app.file-upload.max-file-size-bytes` | `10485760` | default, docker, test | application properties files |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default, docker, test | application properties files |
| `logging.level.com.photoalbum` | `DEBUG` (default/test), `INFO` (docker) | default, docker, test | profile properties |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
| --- | --- | --- | --- |
| photo-album | No explicit `-Xms/-Xmx` or `-D` JVM options detected in repo | Not explicitly configured | 1 container instance in compose |
| oracle-db | Oracle container env (`ORACLE_PASSWORD`, `APP_USER`, `APP_USER_PASSWORD`) | Not explicitly configured in compose | 1 container instance in compose |

## Startup Dependency Chain

1. `oracle-db` starts first and must pass container health check (`healthcheck.sh`).
2. `photoalbum-java-app` starts after Oracle is healthy via `depends_on` condition.
3. Application profile and datasource env vars are injected at container startup.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
| --- | --- | --- |
| `spring.datasource.password` | Database password | Application properties (`[MASKED]`) |
| `ORACLE_PASSWORD` | Oracle admin password | Docker Compose environment (`[MASKED]`) |
| `APP_USER_PASSWORD` | Oracle app user password | Docker Compose environment (`[MASKED]`) |

### Secrets Provisioning Workflow

Secrets are currently provisioned statically through local configuration files and Docker Compose environment entries. At runtime, the Spring app reads datasource credentials from either profile property files or injected environment variables. No managed identity, Key Vault, or external secret store integration is detected.

## Feature Flags

| Flag Name | Default | Controlled By |
| --- | --- | --- |
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
| --- | --- | --- |
| Spring Boot | 2.7.18 | `pom.xml` parent |
| Java | 8 | `pom.xml` properties |
| Spring Boot Maven Plugin | managed by Spring Boot parent | `pom.xml` |
| Oracle JDBC (`ojdbc8`) | managed by Spring Boot BOM | `pom.xml` |
| Commons IO | 2.11.0 | `pom.xml` |
| Docker Oracle image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
