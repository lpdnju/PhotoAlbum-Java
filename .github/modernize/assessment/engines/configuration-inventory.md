# Configuration & Externalized Settings Inventory

The project uses property files plus Docker Compose environment overrides as its primary configuration sources, with a small set of runtime profiles and upload/database-related settings.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| `application.properties` | Spring runtime properties | `src/main/resources/application.properties` | Default runtime config (port, datasource, JPA, multipart, logging) |
| `application-docker.properties` | Spring profile properties | `src/main/resources/application-docker.properties` | Overrides for docker profile Oracle settings and logging |
| `docker-compose.yml` | Container runtime config | `docker-compose.yml` | Sets profile and datasource env vars for app container; Oracle container creds |
| `pom.xml` | Build configuration | `pom.xml` | Java version and dependency/plugin declarations |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| default Maven build | Automatic | Package Spring Boot JAR | `spring-boot-maven-plugin` |
| test execution | Manual (`mvn test`) | Run JUnit/Spring tests with `test` profile in tests | `spring-boot-starter-test`, `h2` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | Implicit startup | `application.properties` | Oracle URL, JPA options, upload limits, debug logging |
| docker | `SPRING_PROFILES_ACTIVE=docker` | `application.properties` + `application-docker.properties` | Oracle URL variant, logging level adjustments |
| test | `@ActiveProfiles("test")` in tests | Default + test context | Uses test dependencies (H2) for context tests |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `server.port` | `8080` | default,docker | application properties |
| `spring.datasource.url` | Oracle JDBC URL | default,docker (overridden) | application + docker profile + env overrides |
| `spring.datasource.username` | `photoalbum` | default,docker | properties/env |
| `spring.datasource.password` | `[MASKED]` | default,docker | properties/env |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | default,docker | application properties |
| `spring.jpa.database-platform` | `org.hibernate.dialect.OracleDialect` | default,docker | application properties |
| `spring.jpa.hibernate.ddl-auto` | `create` | default,docker | application properties |
| `spring.jpa.show-sql` | `true` | default,docker | application properties |
| `spring.servlet.multipart.max-file-size` | `10MB` | default,docker | application properties |
| `spring.servlet.multipart.max-request-size` | `50MB` | default,docker | application properties |
| `app.file-upload.max-file-size-bytes` | `10485760` | default,docker | application properties |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default,docker | application properties |
| `app.file-upload.max-files-per-upload` | `10` | default,docker | application properties |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| photo-album | No explicit `-Xms/-Xmx` or `-D` flags defined in repo | Not explicitly configured | 1 (Docker Compose single service) |
| oracle-db | Container image defaults | Not explicitly configured | 1 |

## Startup Dependency Chain

1. `oracle-db` starts and must pass Compose health check (`healthcheck.sh`).
2. `photoalbum-java-app` starts after Oracle is healthy via `depends_on` condition.
3. Application initializes datasource/JPA and then serves HTTP traffic on port 8080.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `spring.datasource.password` / `SPRING_DATASOURCE_PASSWORD` | Database credential | Properties file and Docker environment variable (`[MASKED]`) |
| `ORACLE_PASSWORD` | Database admin credential | Docker Compose environment variable (`[MASKED]`) |
| `APP_USER_PASSWORD` | Application DB user credential | Docker Compose environment variable (`[MASKED]`) |

### Secrets Provisioning Workflow

Secrets are currently provided through local property files and Docker Compose environment variables. At runtime, the Spring container resolves datasource credentials from environment variables or property files and binds them into datasource configuration for database access.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Spring Boot | 2.7.18 | `pom.xml` parent |
| Java | 1.8 target | `pom.xml` properties |
| Maven build plugin | Managed by Spring Boot parent | `pom.xml` |
| Oracle JDBC | `ojdbc8` (runtime scope) | `pom.xml` |
| Thymeleaf | Managed by Spring Boot BOM | `pom.xml` |
| Docker Oracle image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
