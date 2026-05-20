# Configuration & Externalized Settings Inventory

The application uses 3 Spring properties files (default, docker, and test profiles) plus Docker Compose environment variable overrides as its only configuration sources; there is no external config server, secret store, or feature flag framework.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|--------|------|--------------|-------|
| application.properties | Spring properties (default profile) | `src/main/resources/application.properties` | Active when no profile is explicitly set; targets Oracle `oracle-db:1521/FREEPDB1` |
| application-docker.properties | Spring properties (docker profile) | `src/main/resources/application-docker.properties` | Activated by `SPRING_PROFILES_ACTIVE=docker`; same Oracle settings, with logging adjustments |
| application-test.properties | Spring properties (test profile) | `src/test/resources/application-test.properties` | Used during JUnit test runs; substitutes Oracle with H2 in-memory DB |
| docker-compose.yml (env section) | Docker Compose environment variables | `docker-compose.yml` (photoalbum-java-app service) | Overrides `SPRING_PROFILES_ACTIVE`, `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD` at container startup |
| Dockerfile ENV | Docker image environment variable | `Dockerfile` (`ENV JAVA_OPTS`) | Sets default JVM heap options (`-Xmx512m -Xms256m`) baked into the image |

No Spring Cloud Config server, Azure App Configuration, AWS AppConfig, Consul KV, HashiCorp Vault, or Azure Key Vault is referenced anywhere in the project.

## Build Profiles

No Maven build profiles (`<profiles>`) are defined in `pom.xml`. The build configuration is flat — a single `spring-boot-maven-plugin` handles packaging. The Dockerfile build stage explicitly calls `mvn clean package -DskipTests` using the `maven:3.9.6-eclipse-temurin-8` image.

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---------|-----------|---------|--------------------------|
| (default — no profiles) | Automatic (always active) | Standard compile/package with Spring Boot fat-JAR | `spring-boot-maven-plugin` (repackage goal) |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides vs Default |
|---------|------------------|-------------|--------------------------|
| (default) | No active profile set; local development | `application.properties` | Baseline — Oracle at `oracle-db:1521/FREEPDB1`; `show-sql=true`; DEBUG logging |
| docker | `SPRING_PROFILES_ACTIVE=docker` env var (Docker Compose) | `application.properties` + `application-docker.properties` | Oracle URL from env var; `show-sql=true`; INFO log for app, WARN for Spring Web, DEBUG for Hibernate SQL |
| test | Applied automatically by Spring Test (`@SpringBootTest` or `@DataJpaTest`) | `application-test.properties` | H2 in-memory DB; `ddl-auto=create-drop`; `show-sql=false`; `upload-path=target/test-uploads` |

## Properties Inventory

### photoalbum-java-app

| Property Key | Default (application.properties) | docker Override | test Override | Source |
|-------------|----------------------------------|-----------------|--------------|--------|
| `server.port` | `8080` | `8080` (explicit) | — | Static config |
| `server.servlet.encoding.charset` | `UTF-8` | `UTF-8` | — | Static config |
| `server.servlet.encoding.enabled` | `true` | `true` | — | Static config |
| `server.servlet.encoding.force` | `true` | `true` | — | Static config |
| `spring.datasource.url` | `jdbc:oracle:thin:@oracle-db:1521/FREEPDB1` | `${SPRING_DATASOURCE_URL}` (env var) | `jdbc:h2:mem:testdb` | Static config / env var override |
| `spring.datasource.username` | `photoalbum` | `${SPRING_DATASOURCE_USERNAME}` (env var) | `sa` | Static config / env var override |
| `spring.datasource.password` | `photoalbum` [MASKED] | `${SPRING_DATASOURCE_PASSWORD}` (env var) [MASKED] | *(empty)* | Static config / env var override |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | `oracle.jdbc.OracleDriver` | `org.h2.Driver` | Static config |
| `spring.jpa.database-platform` | `org.hibernate.dialect.OracleDialect` | `org.hibernate.dialect.OracleDialect` | `org.hibernate.dialect.H2Dialect` | Static config |
| `spring.jpa.hibernate.ddl-auto` | `create` | `create` | `create-drop` | Static config |
| `spring.jpa.show-sql` | `true` | `true` | `false` | Static config |
| `spring.jpa.properties.hibernate.format_sql` | `true` | `true` | — | Static config |
| `spring.servlet.multipart.max-file-size` | `10MB` | `10MB` | — | Static config |
| `spring.servlet.multipart.max-request-size` | `50MB` | `50MB` | — | Static config |
| `app.file-upload.max-file-size-bytes` | `10485760` | `10485760` | `10485760` | Static config |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | `image/jpeg,image/png,image/gif,image/webp` | `image/jpeg,image/png,image/gif,image/webp` | Static config |
| `app.file-upload.max-files-per-upload` | `10` | `10` | `10` | Static config |
| `app.file-upload.upload-path` | — (not declared) | — | `target/test-uploads` | Test config only |
| `logging.level.com.photoalbum` | `DEBUG` | `INFO` | `DEBUG` | Static config |
| `logging.level.org.springframework.web` | `DEBUG` | `WARN` | — | Static config |
| `logging.level.org.hibernate.SQL` | — | `DEBUG` | — | Docker config only |

## Startup Parameters & Resource Requirements

| Service | JVM / Runtime Options | Memory | Instance Count |
|---------|-----------------------|--------|---------------|
| photoalbum-java-app | `JAVA_OPTS=-Xmx512m -Xms256m` (baked into Dockerfile ENV); heap injected via `java $JAVA_OPTS -jar app.jar` entrypoint | No Docker `mem_limit` specified in docker-compose.yml — container may use host memory up to JVM heap limit | 1 (no scaling configured) |
| oracle-db | N/A (third-party container, Oracle-managed JVM) | No `mem_limit` in docker-compose.yml | 1 |

No Kubernetes resource requests/limits, no JVM GC flags, and no additional `-D` system properties are configured beyond the heap settings.

## Startup Dependency Chain

```
oracle-db
  └─► healthcheck: healthcheck.sh (interval: 30s, timeout: 10s, retries: 15, start_period: 180s)
        └─► (condition: service_healthy) ──► photoalbum-java-app starts
```

1. **oracle-db** starts first; Docker Compose waits up to ~(`start_period` 180s + 15 × 30s = 630s) for Oracle's built-in `healthcheck.sh` to return healthy.
2. **photoalbum-java-app** starts only after `oracle-db` passes the health check (`condition: service_healthy`).
3. The app container has `restart: on-failure` — it will restart automatically if it crashes (e.g., if Oracle becomes unavailable after initial startup).

No Spring Boot Actuator health endpoints are configured on the application side. There are no Kubernetes readiness/liveness probes and no `dockerize` wait-for-TCP mechanism at the application level.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Configured In | Storage |
|-----------------|------|--------------|---------|
| `spring.datasource.password` | Oracle DB password | `application.properties` | **Plain text** [MASKED] |
| `spring.datasource.password` | Oracle DB password (docker profile) | `application-docker.properties` | Plain text [MASKED] — overridden at runtime by `SPRING_DATASOURCE_PASSWORD` env var |
| `SPRING_DATASOURCE_PASSWORD` | Oracle DB password | `docker-compose.yml` (env section) | **Plain text** in Compose YAML [MASKED] |
| `ORACLE_PASSWORD` | Oracle sys/system password | `docker-compose.yml` (oracle-db env section) | **Plain text** in Compose YAML [MASKED] |
| `APP_USER_PASSWORD` | Oracle app user password | `docker-compose.yml` (oracle-db env section) | **Plain text** in Compose YAML [MASKED] |

### Secrets Provisioning Workflow

There is **no secrets management system** in use. All credentials are stored as plain text directly in source-committed files (`application.properties`, `application-docker.properties`, `docker-compose.yml`). The provisioning workflow is:

1. Developer/operator provides Oracle DB credentials as literal values in `application.properties` (for local runs) or Docker Compose YAML (for containerised runs).
2. At container startup, Docker Compose injects `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, and `SPRING_DATASOURCE_PASSWORD` as environment variables, which Spring Boot resolves via the `${...}` placeholder in `application-docker.properties`.
3. The Oracle container reads `ORACLE_PASSWORD`, `APP_USER`, and `APP_USER_PASSWORD` from its own environment section to initialise the database user on first launch.

No managed identity, service principal, RBAC, Key Vault binding, GitHub Actions secrets injection, or any encrypted secret mechanism is present. Rotating credentials requires direct file edits and a container rebuild/restart.

## Feature Flags

No feature flag framework (LaunchDarkly, Unleash, Spring Feature Flags, or `@ConditionalOnProperty`) is used. The `app.file-upload.*` properties function as operational limits rather than feature toggles, but they are not wired to any conditional bean registration.

| Property | Default | Controlled By | Effect |
|----------|---------|--------------|--------|
| `app.file-upload.max-file-size-bytes` | `10485760` (10 MB) | Properties file | Maximum allowed upload size per file |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | Properties file | Allowed MIME types for upload |
| `app.file-upload.max-files-per-upload` | `10` | Properties file | Maximum number of files per upload request |

## Framework & Runtime Versions

| Component | Version | Source |
|-----------|---------|--------|
| Spring Boot | 2.7.18 | `pom.xml` parent BOM |
| Java (compile target) | 8 (1.8) | `pom.xml` `<java.version>` |
| Java runtime (container) | Eclipse Temurin 8 JRE | `Dockerfile` (`eclipse-temurin:8-jre`) |
| Maven (build stage) | 3.9.6 | `Dockerfile` (`maven:3.9.6-eclipse-temurin-8`) |
| Hibernate ORM | 5.6.x | Spring Boot 2.7.18 BOM (transitive) |
| Spring Data JPA | 2.7.18 | Spring Boot 2.7.18 BOM (transitive) |
| Thymeleaf | 3.0.x | Spring Boot 2.7.18 BOM (transitive) |
| Embedded Tomcat | 9.0.x | Spring Boot 2.7.18 BOM (transitive) |
| Jackson | 2.13.x | Spring Boot 2.7.18 BOM (transitive) |
| Hibernate Validator | 6.x | Spring Boot 2.7.18 BOM (transitive) |
| commons-io | 2.11.0 | `pom.xml` explicit declaration |
| H2 (test) | 2.1.x | Spring Boot 2.7.18 BOM (transitive, test scope) |
| Oracle JDBC (ojdbc8) | BOM-managed | Spring Boot 2.7.18 BOM (runtime scope) |
| Docker build image | maven:3.9.6-eclipse-temurin-8 | `Dockerfile` (build stage) |
| Docker runtime image | eclipse-temurin:8-jre | `Dockerfile` (final stage) |
| Oracle Database container | gvenzl/oracle-free:latest | `docker-compose.yml` (floating tag — not pinned) |
