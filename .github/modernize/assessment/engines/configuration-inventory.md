# Configuration & Externalized Settings Inventory

The application uses 3 Spring Boot property files (default, `docker`, `test` profiles) plus a Docker Compose environment section as configuration sources; secrets are currently stored as plaintext in property files and Docker Compose, with no external secret store in use.

## Configuration Sources

| Source | Type | Path / Location | Notes |
|---|---|---|---|
| `application.properties` | Spring Boot properties | `src/main/resources/application.properties` | Default (active for all profiles unless overridden) |
| `application-docker.properties` | Spring Boot profile properties | `src/main/resources/application-docker.properties` | Activated via `SPRING_PROFILES_ACTIVE=docker` |
| `application-test.properties` | Spring Boot profile properties | `src/test/resources/application-test.properties` | Activated by `@ActiveProfiles("test")` in test classes |
| `docker-compose.yml` environment section | Docker Compose | `docker-compose.yml` | Overrides `SPRING_PROFILES_ACTIVE`, `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD` for the `photoalbum-java-app` container |
| Oracle container init scripts | Shell + SQL | `oracle-init/01-create-user.sql`, `oracle-init/02-verify-user.sql`, `oracle-init/create-user.sh` | Runs once at Oracle container first-start to create the `PHOTOALBUM` database user; not a versioned migration tool |

No Spring Cloud Config server, external configuration repository, HashiCorp Vault, Azure Key Vault, or AWS Secrets Manager integration is present.

## Build Profiles

No Maven build profiles are declared in `pom.xml`. The parent POM (`spring-boot-starter-parent 2.7.18`) does not define environment-targeted build profiles. Build behavior is uniform across all environments: `mvn clean package -DskipTests` is used inside the Docker build stage.

| Profile | Activation | Purpose | Key Changes |
|---|---|---|---|
| _(none defined)_ | N/A | Single build artifact for all environments | Configuration differences handled entirely through runtime Spring profiles |

## Runtime Profiles

| Profile | Activation Method | Config Files Used | Key Overrides vs. Default |
|---|---|---|---|
| default (no profile) | No `spring.profiles.active` set | `application.properties` | Baseline; connects to `oracle-db:1521/FREEPDB1`; `ddl-auto=create`; DEBUG logging |
| `docker` | `SPRING_PROFILES_ACTIVE=docker` environment variable (Docker Compose) | `application.properties` + `application-docker.properties` | JDBC URL uses SID syntax (`oracle-db:1521:XE`); logging reduced to INFO/WARN; adds `org.hibernate.SQL=DEBUG` |
| `test` | `@ActiveProfiles("test")` annotation in test classes | `application.properties` + `application-test.properties` | Switches to H2 in-memory; `ddl-auto=create-drop`; `show-sql=false`; uses `sa`/empty password |

## Properties Inventory

### photoalbum-java-app — Server & Encoding

| Property Key | Default | docker Profile | test Profile | Source |
|---|---|---|---|---|
| `server.port` | `8080` | `8080` | _(inherited)_ | `application.properties` |
| `server.servlet.encoding.charset` | `UTF-8` | `UTF-8` | _(inherited)_ | `application.properties` |
| `server.servlet.encoding.enabled` | `true` | `true` | _(inherited)_ | `application.properties` |
| `server.servlet.encoding.force` | `true` | `true` | _(inherited)_ | `application.properties` |

### photoalbum-java-app — Data Source

| Property Key | Default | docker Profile | test Profile | Source |
|---|---|---|---|---|
| `spring.datasource.url` | `jdbc:oracle:thin:@oracle-db:1521/FREEPDB1` | `jdbc:oracle:thin:@oracle-db:1521:XE` | `jdbc:h2:mem:testdb` | Per-profile files |
| `spring.datasource.username` | `photoalbum` | `photoalbum` | `sa` | Per-profile files |
| `spring.datasource.password` | `photoalbum` _(plaintext)_ | `photoalbum` _(plaintext)_ | _(empty)_ | Per-profile files — **sensitive; see Secrets section** |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | `oracle.jdbc.OracleDriver` | `org.h2.Driver` | Per-profile files |

### photoalbum-java-app — JPA / Hibernate

| Property Key | Default | docker Profile | test Profile | Source |
|---|---|---|---|---|
| `spring.jpa.database-platform` | `org.hibernate.dialect.OracleDialect` | `org.hibernate.dialect.OracleDialect` | `org.hibernate.dialect.H2Dialect` | Per-profile files |
| `spring.jpa.hibernate.ddl-auto` | `create` | `create` | `create-drop` | Per-profile files |
| `spring.jpa.show-sql` | `true` | `true` | `false` | Per-profile files |
| `spring.jpa.properties.hibernate.format_sql` | `true` | `true` | _(not set)_ | `application.properties` / `application-docker.properties` |

### photoalbum-java-app — File Upload

| Property Key | Default | docker Profile | test Profile | Source |
|---|---|---|---|---|
| `spring.servlet.multipart.max-file-size` | `10MB` | `10MB` | _(inherited)_ | `application.properties` |
| `spring.servlet.multipart.max-request-size` | `50MB` | `50MB` | _(inherited)_ | `application.properties` |
| `app.file-upload.max-file-size-bytes` | `10485760` (10 MiB) | `10485760` | `10485760` | Per-profile files |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | same | same | Per-profile files |
| `app.file-upload.max-files-per-upload` | `10` | `10` | `10` | Per-profile files |
| `app.file-upload.upload-path` | _(not set in default)_ | _(not set)_ | `target/test-uploads` | `application-test.properties` only |

### photoalbum-java-app — Logging

| Property Key | Default | docker Profile | test Profile | Source |
|---|---|---|---|---|
| `logging.level.com.photoalbum` | `DEBUG` | `INFO` | `DEBUG` | Per-profile files |
| `logging.level.org.springframework.web` | `DEBUG` | `WARN` | _(not set)_ | Per-profile files |
| `logging.level.org.hibernate.SQL` | _(not set)_ | `DEBUG` | _(not set)_ | `application-docker.properties` only |

## Startup Parameters & Resource Requirements

| Service | JVM / Runtime Options | Container Memory | Instance Count | Notes |
|---|---|---|---|---|
| `photoalbum-java-app` | `-Xmx512m -Xms256m` (set via `JAVA_OPTS` env var in Dockerfile) | No explicit `mem_limit` in Docker Compose | 1 | JVM heap ceiling of 512 MiB; no GC tuning specified |
| `oracle-db` | N/A (third-party container) | No explicit limit | 1 | Oracle Free 23ai; persistent volume `oracle_data` |

No Kubernetes resource requests/limits, cloud deployment scaling configuration, or additional `-D` system properties are set.

## Startup Dependency Chain

```
oracle-db  →  (healthy)  →  photoalbum-java-app
```

| Step | Service | Wait Mechanism | Timeout / Retries |
|---|---|---|---|
| 1 | `oracle-db` | Docker Compose `healthcheck`: runs `healthcheck.sh` inside container | Interval: 30 s, timeout: 10 s, retries: 15, start_period: 180 s |
| 2 | `photoalbum-java-app` | `depends_on: oracle-db: condition: service_healthy` | Waits indefinitely until Oracle health check passes; `restart: on-failure` if Spring app crashes before Oracle is ready |

No Spring Boot Actuator readiness/liveness probes are configured; there is no Kubernetes readiness probe path or `dockerize` wait-for-TCP mechanism at the application level.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Profile(s) | Storage | Notes |
|---|---|---|---|---|
| `spring.datasource.password` | Database password | default, docker | Plaintext in `application.properties` / `application-docker.properties` — **[MASKED: photoalbum]** | Hard-coded; committed to source control |
| `spring.datasource.username` | Database username | default, docker | Plaintext in property files | Hard-coded; committed to source control |
| `SPRING_DATASOURCE_PASSWORD` | Database password | docker (Docker Compose override) | Plaintext in `docker-compose.yml` environment section — **[MASKED]** | Overrides the property file value inside the container |
| `ORACLE_PASSWORD` | Oracle SYS/SYSTEM password | docker (Oracle container) | Plaintext in `docker-compose.yml` — **[MASKED]** | Used by `gvenzl/oracle-free` image to set SYSTEM password |
| `APP_USER_PASSWORD` | Oracle app-user password | docker (Oracle container) | Plaintext in `docker-compose.yml` — **[MASKED]** | Sets the `photoalbum` user password at container init |

No encryption (Jasypt, DPAPI, sealed secrets) is applied. No external secret store (Azure Key Vault, HashiCorp Vault, AWS Secrets Manager) is referenced.

### Secrets Provisioning Workflow

All secrets are currently hard-coded in source-controlled files (`application.properties`, `docker-compose.yml`). There is no runtime secret injection, managed identity, or secret rotation mechanism. The provisioning sequence is:

1. Developer / CI clones the repository — secrets are present in plain text.
2. `docker compose up` starts `oracle-db` with `ORACLE_PASSWORD` and `APP_USER_PASSWORD` from `docker-compose.yml`.
3. Oracle container runs `oracle-init/01-create-user.sql` on first start, creating the `photoalbum` user with the hard-coded password.
4. `photoalbum-java-app` container starts with `SPRING_DATASOURCE_PASSWORD` injected from `docker-compose.yml`; Spring Boot resolves it to `spring.datasource.password`.

**Risk**: All secrets are exposed in version control. For production deployment, secrets should be migrated to environment-variable injection (CI/CD secret variables), a managed secret store (Azure Key Vault, AWS Secrets Manager), or Kubernetes Secrets, and removed from property files.

## Feature Flags

No feature flag framework (LaunchDarkly, Unleash, Spring Feature Flags, .NET FeatureManagement) is used. No `@ConditionalOnProperty`, `@ConditionalOnExpression`, or similar conditional bean registration annotations were found in the source code. There are no A/B testing or gradual-rollout configurations.

| Flag Name | Default | Controlled By |
|---|---|---|
| _(none detected)_ | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Java (source/target) | 8 (1.8) | `pom.xml` `<java.version>` / `<maven.compiler.source>` |
| Spring Boot | 2.7.18 | `pom.xml` parent BOM |
| Spring MVC | 5.3.x (Boot managed) | `spring-boot-starter-web` transitive |
| Thymeleaf | 3.0.x (Boot managed) | `spring-boot-starter-thymeleaf` transitive |
| Spring Data JPA | 2.7.x (Boot managed) | `spring-boot-starter-data-jpa` transitive |
| Hibernate ORM | 5.6.x (Boot managed) | `spring-boot-starter-data-jpa` transitive |
| Oracle JDBC (ojdbc8) | 21.x (Boot BOM managed) | `com.oracle.database.jdbc:ojdbc8` |
| Apache Commons IO | 2.11.0 | `pom.xml` explicit version |
| Jackson | 2.13.x (Boot managed) | `spring-boot-starter-json` transitive |
| H2 (test) | Boot managed | `com.h2database:h2` (test scope) |
| JUnit 5 (test) | Boot managed | `spring-boot-starter-test` transitive |
| Maven | 3.x (project-level) | `mvn` CLI; build stage uses `maven:3.9.6-eclipse-temurin-8` Docker image |
| Docker build base image | `maven:3.9.6-eclipse-temurin-8` | `Dockerfile` (build stage) |
| Docker runtime base image | `eclipse-temurin:8-jre` | `Dockerfile` (final stage) |
| Oracle DB container image | `gvenzl/oracle-free:latest` | `docker-compose.yml` |
