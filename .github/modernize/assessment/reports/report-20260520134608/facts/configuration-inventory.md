# Configuration & Externalized Settings Inventory

The project uses a small set of Spring property files plus Docker Compose environment overrides to control runtime behavior, database connectivity, upload limits, and logging.

## Configuration Sources

| Source | Type | Path/Location | Notes |
| --- | --- | --- | --- |
| Spring default config | Properties file | `src/main/resources/application.properties` | Main runtime settings including datasource and upload limits |
| Spring docker profile config | Properties file | `src/main/resources/application-docker.properties` | Profile-specific Oracle connection and logging overrides |
| Spring test config | Properties file | `src/test/resources/application-test.properties` | H2 test datasource and test upload settings |
| Container runtime config | Docker Compose env | `docker-compose.yml` | Injects active profile and datasource variables |
| Build descriptor | Maven POM | `pom.xml` | Declares Java version, dependencies, and plugin configuration |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
| --- | --- | --- | --- |
| Default Maven build | Automatic (`mvn ...`) | Compile/package Spring Boot app | `spring-boot-maven-plugin` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
| --- | --- | --- | --- |
| default | Implicit when no profile set | `application.properties` | Oracle datasource, upload constraints, debug logging |
| docker | `SPRING_PROFILES_ACTIVE=docker` | `application.properties` + `application-docker.properties` | Oracle URL variant, production-oriented logging levels |
| test | Test runtime configuration | `application-test.properties` | H2 datasource and create-drop schema behavior |

## Properties Inventory

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| `server.port` | `8080` | default, docker | application properties |
| `spring.datasource.url` | Oracle JDBC URL | default, docker override, test override | application/test properties + compose env |
| `spring.datasource.username` | `photoalbum` (or `sa` in test) | default, docker, test | application/test properties + compose env |
| `spring.datasource.password` | `photoalbum` (blank in test) | default, docker, test | application/test properties + compose env |
| `spring.datasource.driver-class-name` | Oracle driver (`H2` in test) | default, docker, test | application/test properties |
| `spring.jpa.database-platform` | Oracle dialect (`H2Dialect` in test) | default, docker, test | application/test properties |
| `spring.jpa.hibernate.ddl-auto` | `create` (`create-drop` in test) | default, docker, test | application/test properties |
| `spring.servlet.multipart.max-file-size` | `10MB` | default, docker | application properties |
| `spring.servlet.multipart.max-request-size` | `50MB` | default, docker | application properties |
| `app.file-upload.max-file-size-bytes` | `10485760` | default, docker, test | application/test properties |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default, docker, test | application/test properties |
| `app.file-upload.max-files-per-upload` | `10` | default, docker, test | application/test properties |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
| --- | --- | --- | --- |
| photo-album | No explicit JVM options discovered in repo | Not explicitly configured in repo | 1 container instance in docker-compose |
| oracle-db | N/A (database container) | Not explicitly configured in repo | 1 container instance in docker-compose |

## Startup Dependency Chain

1. `oracle-db` starts first and must report healthy (`healthcheck.sh`).
2. `photoalbum-java-app` waits on `oracle-db` through Docker Compose `depends_on` with `service_healthy` condition.
3. Application initializes datasource/JPA; endpoints become available after successful DB connectivity.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
| --- | --- | --- |
| `spring.datasource.password` | Database password | `[MASKED]` in app properties / env var override |
| `ORACLE_PASSWORD` | Oracle system password | `[MASKED]` in docker-compose environment |
| `APP_USER_PASSWORD` | Application schema password | `[MASKED]` in docker-compose environment |

### Secrets Provisioning Workflow

Secrets are currently injected directly via plaintext properties and Docker Compose environment variables. No managed identity, external secret manager, or RBAC-driven retrieval flow is defined in repository configuration.

## Feature Flags

| Flag Name | Default | Controlled By |
| --- | --- | --- |
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
| --- | --- | --- |
| Spring Boot | 2.7.18 | `pom.xml` parent |
| Java target | 8 | `pom.xml` properties |
| Spring Framework (via Boot BOM) | Managed by Boot 2.7.18 | Maven BOM inheritance |
| Oracle JDBC driver | `ojdbc8` (version BOM-managed) | `pom.xml` dependency |
| Maven build | Wrapper not present; project uses Maven model | repository build instructions |
| Docker Oracle image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
