# Configuration & Externalized Settings Inventory

Configuration is primarily file-based through Spring Boot properties plus Docker Compose environment overrides. Profiles are simple (`default` and `docker`) and secrets are currently provided as plaintext values in local configuration.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Spring properties | Application config | `src/main/resources/application.properties` | Default runtime config, DB, upload, logging |
| Spring properties | Profile config | `src/main/resources/application-docker.properties` | Docker-specific Oracle endpoint and logging |
| Docker Compose | Runtime environment | `docker-compose.yml` | Overrides profile and datasource values for containers |
| Dockerfile | Container runtime settings | `Dockerfile` | JVM options and startup command |
| Maven POM | Build/dependency config | `pom.xml` | Dependency and Java version management |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| default Maven build | Automatic | Build runnable Spring Boot JAR | `spring-boot-maven-plugin` |

No explicit Maven `<profiles>` blocks were detected.

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | Startup default when no profile set | `application.properties` | Oracle URL `.../FREEPDB1`, DEBUG logging |
| docker | `SPRING_PROFILES_ACTIVE=docker` | `application-docker.properties` | Oracle URL `...:XE`, logging tuned for container |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `server.port` | `8080` | default,docker | properties files |
| `spring.datasource.url` | Oracle JDBC URL | default,docker (different values) | properties + compose env override |
| `spring.datasource.username` | `photoalbum` | default,docker | properties + compose env override |
| `spring.datasource.password` | `[MASKED]` | default,docker | properties + compose env override |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | default,docker | properties files |
| `spring.jpa.hibernate.ddl-auto` | `create` | default,docker | properties files |
| `spring.servlet.multipart.max-file-size` | `10MB` | default,docker | properties files |
| `spring.servlet.multipart.max-request-size` | `50MB` | default,docker | properties files |
| `app.file-upload.max-file-size-bytes` | `10485760` | default,docker | properties files |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default,docker | properties files |
| `logging.level.com.photoalbum` | `DEBUG` / `INFO` | default,docker | profile-specific |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| photoalbum-java-app | `java $JAVA_OPTS -jar app.jar` | `JAVA_OPTS=-Xmx512m -Xms256m` (Dockerfile) | 1 (compose) |
| oracle-db | Oracle container defaults | Not explicitly constrained in compose | 1 (compose) |

## Startup Dependency Chain

1. `oracle-db` starts and must become healthy (`healthcheck.sh`).
2. `photoalbum-java-app` waits on `oracle-db` with `depends_on: condition: service_healthy`.
3. Application endpoints become usable after DB connectivity succeeds.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `spring.datasource.password` | Database password | Plaintext in `application*.properties` / compose env (masked in this document) |
| `ORACLE_PASSWORD` | Oracle admin password | Docker Compose environment variable (masked) |
| `APP_USER_PASSWORD` | Application DB user password | Docker Compose environment variable (masked) |

### Secrets Provisioning Workflow

Secrets are injected locally via static config files and Docker Compose environment values. No managed secret store, managed identity, or external vault integration is configured in this repository.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| No explicit feature flags detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Spring Boot | 2.7.18 | `pom.xml` parent |
| Java | 8 (`java.version=1.8`) | `pom.xml` |
| Maven builder image | 3.9.6 + Temurin 8 | `Dockerfile` |
| Runtime base image | eclipse-temurin 8-jre | `Dockerfile` |
| Oracle JDBC | ojdbc8 (BOM-managed) | `pom.xml` |
