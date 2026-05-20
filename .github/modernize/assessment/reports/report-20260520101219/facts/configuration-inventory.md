# Configuration & Externalized Settings Inventory

This inventory captures the project's configuration sources, runtime profile behavior, startup dependencies, and sensitive settings handling for the current Spring Boot photo service.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Spring default properties | Application config | `src/main/resources/application.properties` | Base runtime settings including datasource, JPA, upload limits, logging |
| Spring docker profile properties | Application config | `src/main/resources/application-docker.properties` | Overrides for docker profile Oracle connectivity and logging |
| Docker Compose service config | Container orchestration config | `docker-compose.yml` | Defines `oracle-db` and `photoalbum-java-app` services with environment overrides |
| Docker image build config | Container build config | `Dockerfile` | Sets JVM options and runtime packaging behavior |
| Oracle initialization SQL | Database bootstrap scripts | `oracle-init/*.sql`, `oracle-init/create-user.sh` | Provisioning scripts for database user and health checks |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| default Maven build | `mvn` invocation | Build executable Spring Boot JAR | `spring-boot-maven-plugin` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | implicit when `SPRING_PROFILES_ACTIVE` not set | `application.properties` | Oracle datasource URL (`FREEPDB1`), DEBUG-level app/web logs |
| docker | `SPRING_PROFILES_ACTIVE=docker` via docker-compose | `application.properties` + `application-docker.properties` | Alternate Oracle URL (`:XE`), INFO/WARN logging defaults |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `server.port` | `8080` | default, docker | application properties |
| `spring.datasource.url` | `jdbc:oracle:thin:@oracle-db:1521/FREEPDB1` | default (overridden in docker profile/env) | properties + compose env |
| `spring.datasource.username` | `photoalbum` | default, docker | properties + compose env |
| `spring.datasource.password` | `[MASKED]` | default, docker | properties + compose env |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | default, docker | properties |
| `spring.jpa.database-platform` | `org.hibernate.dialect.OracleDialect` | default, docker | properties |
| `spring.jpa.hibernate.ddl-auto` | `create` | default, docker | properties |
| `spring.servlet.multipart.max-file-size` | `10MB` | default, docker | properties |
| `spring.servlet.multipart.max-request-size` | `50MB` | default, docker | properties |
| `app.file-upload.max-file-size-bytes` | `10485760` | default, docker | properties |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default, docker | properties |
| `app.file-upload.max-files-per-upload` | `10` | default, docker | properties |
| `logging.level.com.photoalbum` | `DEBUG` | default (docker overrides to INFO) | properties |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| photoalbum-java-app | `JAVA_OPTS="-Xmx512m -Xms256m"`; `-jar app.jar` | JVM heap min 256m / max 512m | 1 (compose default) |
| oracle-db | Oracle container defaults | Persistent volume-backed DB container | 1 (compose default) |

## Startup Dependency Chain

1. `oracle-db` starts first and must pass Docker health check (`healthcheck.sh`).
2. `photoalbum-java-app` waits on `depends_on: condition: service_healthy`.
3. Application startup depends on successful Oracle JDBC connectivity and schema creation behavior.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `spring.datasource.password` | Database credential | Application properties / environment variable override (`[MASKED]`) |
| `ORACLE_PASSWORD` | Oracle system credential | Docker Compose environment (`[MASKED]`) |
| `APP_USER_PASSWORD` | App DB user credential | Docker Compose environment (`[MASKED]`) |

### Secrets Provisioning Workflow

Credentials are currently provisioned through local configuration files and Docker Compose environment variables at container startup. The Oracle container initializes users via SQL scripts, then the Spring Boot service consumes datasource credentials from profile/environment values. No external secret manager, managed identity, or RBAC-based secret retrieval flow is configured.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Spring Boot | 2.7.18 | `pom.xml` parent |
| Java target | 8 | `pom.xml` properties |
| Maven (container build stage) | 3.9.6 | `Dockerfile` build image |
| JPA/Hibernate stack | Managed by Boot 2.7.18 | Maven starter |
| Oracle JDBC driver | `ojdbc8` | `pom.xml` dependency |
| Bootstrap UI library | 5.3.0 (CDN) | Thymeleaf templates |
| Oracle container image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
| Runtime JRE image | `eclipse-temurin:8-jre` | `Dockerfile` |
