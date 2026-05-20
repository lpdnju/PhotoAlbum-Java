# Configuration & Externalized Settings Inventory

This inventory summarizes the application's configuration footprint across Spring property files, container configuration, and build/runtime profile controls.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Spring properties | Runtime config | `src/main/resources/application.properties` | Default app, datasource, upload limits, logging |
| Spring properties | Runtime config | `src/main/resources/application-docker.properties` | Docker profile overrides for datasource/logging |
| Maven project file | Build config | `pom.xml` | Dependency and plugin configuration |
| Docker compose | Deployment/runtime config | `docker-compose.yml` | Service environment variables, ports, health checks, startup ordering |
| Dockerfile | Container build config | `Dockerfile` | Java runtime image and startup command |
| Oracle init scripts | DB init config | `oracle-init/` | Oracle user/schema initialization scripts |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| default Maven build | `mvn` lifecycle | Build executable Spring Boot jar | `spring-boot-maven-plugin` |
| test scope | `mvn test` | Test context bootstrap | `spring-boot-starter-test`, `h2` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | implicit (no profile set) | `application.properties` | Oracle FREEPDB1 datasource, DEBUG app logs |
| docker | `SPRING_PROFILES_ACTIVE=docker` | `application.properties` + `application-docker.properties` | Oracle XE datasource, Docker-oriented logging levels |
| test | `@ActiveProfiles("test")` in tests | test context with classpath settings | Spring context smoke test profile |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `server.port` | `8080` | default, docker | application properties |
| `spring.datasource.url` | Oracle JDBC URL | default, docker (different URL) | application properties |
| `spring.datasource.username` | `photoalbum` | default, docker | application properties |
| `spring.datasource.password` | `[MASKED]` | default, docker | application properties |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | default, docker | application properties |
| `spring.jpa.database-platform` | `org.hibernate.dialect.OracleDialect` | default, docker | application properties |
| `spring.jpa.hibernate.ddl-auto` | `create` | default, docker | application properties |
| `spring.jpa.show-sql` | `true` | default, docker | application properties |
| `spring.servlet.multipart.max-file-size` | `10MB` | default, docker | application properties |
| `spring.servlet.multipart.max-request-size` | `50MB` | default, docker | application properties |
| `app.file-upload.max-file-size-bytes` | `10485760` | default, docker | application properties |
| `app.file-upload.allowed-mime-types` | jpeg,png,gif,webp | default, docker | application properties |
| `app.file-upload.max-files-per-upload` | `10` | default, docker | application properties |
| `logging.level.com.photoalbum` | `DEBUG` (default), `INFO` (docker) | default, docker | application properties |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| photoalbum-java-app | `SPRING_PROFILES_ACTIVE=docker` plus datasource env overrides | Not explicitly set in compose | 1 |
| oracle-db | Oracle container environment (`ORACLE_PASSWORD`, `APP_USER`, `APP_USER_PASSWORD`) | Volume-backed data, no explicit limit in compose | 1 |

## Startup Dependency Chain

1. `oracle-db` starts first and must become healthy (`healthcheck.sh`, retries/start_period configured).
2. `photoalbum-java-app` waits on `oracle-db` health via Docker Compose `depends_on` condition.
3. Application begins serving requests after Spring context and datasource initialization complete.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `spring.datasource.password` | Database password | Property file `[MASKED]` |
| `SPRING_DATASOURCE_PASSWORD` | Database password | Docker Compose env var `[MASKED]` |
| `ORACLE_PASSWORD` | Database admin password | Docker Compose env var `[MASKED]` |
| `APP_USER_PASSWORD` | Application DB user password | Docker Compose env var `[MASKED]` |

### Secrets Provisioning Workflow

Secrets are currently provided as static property values or Docker Compose environment variables at startup time. The application container consumes datasource credentials from environment variables and connects to the Oracle container. No managed secret store, managed identity, or centralized secret rotation workflow is configured.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Spring Boot | 2.7.18 | `pom.xml` parent |
| Java target | 8 | `pom.xml` properties |
| Spring Data JPA | Boot-managed | `pom.xml` dependency |
| Thymeleaf | Boot-managed | `pom.xml` dependency |
| Oracle JDBC (`ojdbc8`) | Boot-managed/runtime | `pom.xml` dependency |
| Maven | Project uses Maven build | `pom.xml` |
| Oracle container image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
