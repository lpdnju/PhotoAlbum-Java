# Configuration & Externalized Settings Inventory

The application relies on a small but clear set of configuration sources: Maven build metadata, Spring property files for default, Docker, and test execution, Docker Compose runtime overrides, and Oracle initialization scripts. Secrets are currently handled through checked-in plaintext values rather than an external secret manager.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Maven project descriptor | Build configuration | `pom.xml` | Declares Java version, dependencies, and Spring Boot packaging |
| Spring default properties | Runtime configuration | `src/main/resources/application.properties` | Default server port, Oracle datasource, upload limits, and logging |
| Spring Docker profile | Runtime configuration | `src/main/resources/application-docker.properties` | Docker-specific Oracle connection and logging overrides |
| Spring test profile | Runtime configuration | `src/test/resources/application-test.properties` | H2 datasource and test-specific upload settings |
| Docker Compose | Container runtime configuration | `docker-compose.yml` | Defines app and database services, environment overrides, and startup dependency |
| Dockerfile | Container build and runtime configuration | `Dockerfile` | Multi-stage build, exposed port, and JVM heap options |
| Oracle init SQL and shell scripts | Database bootstrap configuration | `oracle-init/*` | Creates the database user and verifies privileges |
| AppCAT assessment config | Assessment tooling | `.github/modernize/appcat/assessment-config.yaml` | Generated assessment configuration for AppCAT |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Default Maven build | Automatic | Compiles and packages the Spring Boot application as a runnable jar | `spring-boot-maven-plugin` |
| Docker image build | Manual `docker build` or `docker-compose up --build` | Creates the container image with Maven and Temurin 8 images | Docker multi-stage build, Maven 3.9.6 image |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | No explicit profile required | `application.properties` | Oracle datasource, port 8080, upload limits, DEBUG logging |
| docker | `SPRING_PROFILES_ACTIVE=docker` in Docker Compose | `application.properties`, `application-docker.properties`, Docker env vars | Oracle XE-style datasource URL, INFO and WARN logging, container-oriented deployment settings |
| test | `@ActiveProfiles("test")` in Spring Boot tests | `application-test.properties` | H2 in-memory datasource, `create-drop` schema handling, test upload path |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `server.port` | `8080` | default, docker | `application.properties`, `application-docker.properties` |
| `server.servlet.encoding.charset` | `UTF-8` | default, docker | Spring property files |
| `server.servlet.encoding.enabled` | `true` | default, docker | Spring property files |
| `server.servlet.encoding.force` | `true` | default, docker | Spring property files |
| `spring.datasource.url` | `jdbc:oracle:thin:@oracle-db:1521/FREEPDB1` | default | `application.properties` |
| `spring.datasource.url` | `jdbc:oracle:thin:@oracle-db:1521:XE` | docker | `application-docker.properties`, overridden by Docker Compose env var |
| `spring.datasource.url` | `jdbc:h2:mem:testdb` | test | `application-test.properties` |
| `spring.datasource.username` | `photoalbum` | default, docker | Spring property files and Docker env vars |
| `spring.datasource.password` | `[MASKED]` | default, docker, test | Spring property files and Docker env vars |
| `spring.datasource.driver-class-name` | Oracle or H2 driver | default, docker, test | Spring property files |
| `spring.jpa.database-platform` | Oracle or H2 dialect | default, docker, test | Spring property files |
| `spring.jpa.hibernate.ddl-auto` | `create` or `create-drop` | default, docker, test | Spring property files |
| `spring.jpa.show-sql` | `true` in runtime, `false` in tests | default, docker, test | Spring property files |
| `spring.jpa.properties.hibernate.format_sql` | `true` | default, docker | Spring property files |
| `spring.servlet.multipart.max-file-size` | `10MB` | default, docker | Spring property files |
| `spring.servlet.multipart.max-request-size` | `50MB` | default, docker | Spring property files |
| `app.file-upload.max-file-size-bytes` | `10485760` | default, docker, test | Spring property files |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default, docker, test | Spring property files |
| `app.file-upload.max-files-per-upload` | `10` | default, docker, test | Spring property files |
| `app.file-upload.upload-path` | `target/test-uploads` | test | `application-test.properties` |
| `logging.level.com.photoalbum` | `DEBUG` or `INFO` | default, docker, test | Spring property files |
| `logging.level.org.springframework.web` | `DEBUG` or `WARN` | default, docker | Spring property files |
| `logging.level.org.hibernate.SQL` | `DEBUG` | docker | `application-docker.properties` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| photoalbum-java-app | `JAVA_OPTS="-Xmx512m -Xms256m"`; `SPRING_PROFILES_ACTIVE=docker`; datasource env var overrides in Docker Compose | JVM heap capped at 512 MB; README recommends at least 4 GB Docker host RAM to accommodate Oracle as well | 1 |
| oracle-db | Container health check `healthcheck.sh`; Oracle user bootstrap scripts mounted from `oracle-init` | Persistent volume `oracle_data`; Oracle container is the largest memory consumer in the stack | 1 |

## Startup Dependency Chain

1. `oracle-db` starts first and initializes the database files plus the `photoalbum` user through the mounted init scripts.
2. `photoalbum-java-app` waits for `oracle-db` to become healthy via Docker Compose `depends_on` with `condition: service_healthy`.
3. Once Oracle is healthy, the application starts with the `docker` profile, connects to Oracle, and allows Hibernate to create the schema.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `spring.datasource.password` | Database password | Checked-in Spring property files as `[MASKED]` |
| `SPRING_DATASOURCE_PASSWORD` | Container environment variable | Docker Compose environment section as `[MASKED]` |
| `ORACLE_PASSWORD` | Oracle system password | Docker Compose environment section as `[MASKED]` |
| `APP_USER_PASSWORD` | Oracle application user password | Docker Compose environment section as `[MASKED]` |
| `photoalbum/photoalbum` documented credentials | Local developer credential | README examples and Oracle init scripts as `[MASKED]` |

### Secrets Provisioning Workflow

Secrets are provisioned through source-controlled configuration rather than a secret store. Docker Compose injects Oracle system and application-user passwords directly into the database and application containers, and the local-development instructions repeat the same credentials for manual setup. No managed identity, Key Vault, Vault, or deployment-time secret binding is configured.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | No feature-flag framework or conditional feature toggle was found |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Java | 8 | `pom.xml` properties and Docker base images |
| Spring Boot | 2.7.18 | `pom.xml` parent |
| Spring Framework | 5.3.x via Spring Boot | Managed by Spring Boot parent |
| Thymeleaf | Managed by Spring Boot 2.7.18 | `pom.xml` starter dependency |
| Hibernate / Spring Data JPA | Managed by Spring Boot 2.7.18 | `pom.xml` starter dependency |
| Oracle JDBC | `ojdbc8` managed by Spring Boot | `pom.xml` runtime dependency |
| Maven build image | 3.9.6 with Temurin 8 | `Dockerfile` build stage |
| Runtime base image | Eclipse Temurin 8 JRE | `Dockerfile` final stage |
| Oracle container image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
| Bootstrap | 5.3.0 | Thymeleaf templates CDN reference |
