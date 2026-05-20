# Configuration & Externalized Settings Inventory

This application uses a small set of local configuration files and Docker Compose settings to control database connectivity, upload limits, logging, and runtime startup behavior. Secrets handling is file- and environment-based rather than delegated to a dedicated secret store.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|------|------|------|------|
| Main application properties | Spring Boot properties | `src/main/resources/application.properties` | Default runtime config including port, datasource, JPA, upload limits, and logging |
| Docker profile properties | Spring Boot profile properties | `src/main/resources/application-docker.properties` | Overrides datasource and logging for the Docker deployment profile |
| Test properties | Spring Boot test properties | `src/test/resources/application-test.properties` | Switches tests to H2 and test upload directory settings |
| Docker Compose | Container runtime config | `docker-compose.yml` | Defines Oracle and application services, environment variables, ports, health check, and startup dependency |
| Docker image build | Container build config | `Dockerfile` | Defines container packaging for the Spring Boot application |
| Oracle initialization scripts | Database bootstrap scripts | `oracle-init/*` | Creates database user and validates Oracle startup inside the container |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|------|------|------|------|
| Default Maven build | Standard `mvn` invocation | Compiles and packages the Spring Boot application | `spring-boot-maven-plugin` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|------|------|------|------|
| default | Implicit Spring Boot default | `application.properties` | Oracle datasource URL, upload limits, port 8080, debug logging |
| `docker` | `SPRING_PROFILES_ACTIVE=docker` in Docker Compose | `application.properties`, `application-docker.properties` | Docker-specific Oracle JDBC URL and reduced logging levels |
| test | Spring Boot test configuration | `application-test.properties` | H2 datasource, create-drop schema behavior, test upload path |

## Properties Inventory

### `photo-album`

| Property Key | Default | Profiles | Source |
|------|------|------|------|
| `server.port` | `8080` | default, docker | `application.properties`, `application-docker.properties` |
| `server.servlet.encoding.charset` | `UTF-8` | default, docker | `application.properties`, `application-docker.properties` |
| `server.servlet.encoding.enabled` | `true` | default, docker | `application.properties`, `application-docker.properties` |
| `server.servlet.encoding.force` | `true` | default, docker | `application.properties`, `application-docker.properties` |
| `spring.datasource.url` | Oracle JDBC URL | default, docker, test override | properties files plus Docker Compose env overrides |
| `spring.datasource.username` | `photoalbum` | default, docker, test override | properties files plus Docker Compose env overrides |
| `spring.datasource.password` | `[MASKED]` | default, docker, test override | properties files plus Docker Compose env overrides |
| `spring.datasource.driver-class-name` | Oracle or H2 driver | default, docker, test | profile-specific properties |
| `spring.jpa.database-platform` | Oracle dialect or H2 dialect | default, docker, test | profile-specific properties |
| `spring.jpa.hibernate.ddl-auto` | `create` or `create-drop` | default, docker, test | profile-specific properties |
| `spring.jpa.show-sql` | `true` or `false` | default, docker, test | profile-specific properties |
| `spring.jpa.properties.hibernate.format_sql` | `true` | default, docker | properties files |
| `spring.servlet.multipart.max-file-size` | `10MB` | default, docker | properties files |
| `spring.servlet.multipart.max-request-size` | `50MB` | default, docker | properties files |
| `app.file-upload.max-file-size-bytes` | `10485760` | default, docker, test | properties files |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default, docker, test | properties files |
| `app.file-upload.max-files-per-upload` | `10` | default, docker, test | properties files |
| `app.file-upload.upload-path` | `target/test-uploads` | test only | `application-test.properties` |
| `logging.level.com.photoalbum` | `DEBUG` or `INFO` | default, docker, test | properties files |
| `logging.level.org.springframework.web` | `DEBUG` or `WARN` | default, docker | properties files |
| `logging.level.org.hibernate.SQL` | `DEBUG` | docker | `application-docker.properties` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|------|------|------|------|
| `photo-album` | No explicit JVM flags detected; runtime profile selected with `SPRING_PROFILES_ACTIVE=docker` in Compose | Not specified in repo configuration | 1 container |
| `oracle-db` | Oracle container environment variables configure database user and password | README notes at least 4 GB RAM available for Oracle locally | 1 container |

## Startup Dependency Chain

1. `oracle-db` starts first and runs its container health check plus initialization scripts.
2. `photoalbum-java-app` waits for the Oracle service to become healthy via Docker Compose `depends_on` with `condition: service_healthy`.
3. Once the database is reachable, the Spring Boot application starts on port 8080 using the `docker` profile.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|------|------|------|
| `spring.datasource.password` | Database password | Plaintext in properties files and overridden by Docker Compose environment variable `[MASKED]` |
| `ORACLE_PASSWORD` | Oracle admin/bootstrap password | Docker Compose environment variable `[MASKED]` |
| `APP_USER_PASSWORD` | Application database user password | Docker Compose environment variable `[MASKED]` |

### Secrets Provisioning Workflow

Secrets are provisioned locally through repository-managed configuration rather than a remote vault. Docker Compose injects Oracle credentials into the database and application containers, and the Spring Boot application binds those values to datasource properties at startup. No managed identity, RBAC-backed secret store, or deployment-time secret broker was detected.

## Feature Flags

| Flag Name | Default | Controlled By |
|------|------|------|
| None detected | N/A | No feature-flag framework or conditional property-based feature toggle was found |

## Framework & Runtime Versions

| Component | Version | Source |
|------|------|------|
| Java | 1.8 | `pom.xml` |
| Spring Boot | 2.7.18 | `pom.xml` parent |
| Spring Framework | Managed by Spring Boot 2.7.18 | Maven dependency management |
| Maven | Project uses Maven; wrapper not present | `pom.xml` |
| Thymeleaf | Managed by Spring Boot 2.7.18 | `pom.xml` |
| Bootstrap | 5.3.0 | `templates/index.html`, `templates/detail.html` CDN link |
| Oracle Docker image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
