# Configuration & Externalized Settings Inventory

This project keeps configuration in a small set of Spring property files with environment-specific overrides, and directly defines several sensitive runtime values in local configuration.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Spring application properties | Static file | `src/main/resources/application.properties` | Default runtime settings including datasource and upload limits |
| Spring docker profile properties | Static file | `src/main/resources/application-docker.properties` | Docker-oriented Oracle connection and logging overrides |
| Maven build descriptor | Build configuration | `pom.xml` | Defines dependency set, Java version, packaging, and plugin config |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| default Maven build | Standard Maven lifecycle | Build Spring Boot jar | `spring-boot-maven-plugin` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | Implicit when no Spring profile set | `application.properties` | Oracle datasource, SQL logging, upload validation settings |
| docker | `spring.profiles.active=docker` (expected runtime activation) | `application-docker.properties` (+ defaults) | Docker Oracle URL variant and reduced logging noise |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `server.port` | `8080` | default, docker | application properties files |
| `server.servlet.encoding.charset` | `UTF-8` | default, docker | application properties files |
| `server.servlet.encoding.enabled` | `true` | default, docker | application properties files |
| `server.servlet.encoding.force` | `true` | default, docker | application properties files |
| `spring.datasource.url` | Oracle JDBC URL | default, docker override | application properties files |
| `spring.datasource.username` | `photoalbum` | default, docker | application properties files |
| `spring.datasource.password` | `[MASKED]` | default, docker | application properties files |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | default, docker | application properties files |
| `spring.jpa.database-platform` | `org.hibernate.dialect.OracleDialect` | default, docker | application properties files |
| `spring.jpa.hibernate.ddl-auto` | `create` | default, docker | application properties files |
| `spring.jpa.show-sql` | `true` | default, docker | application properties files |
| `spring.jpa.properties.hibernate.format_sql` | `true` | default, docker | application properties files |
| `spring.servlet.multipart.max-file-size` | `10MB` | default, docker | application properties files |
| `spring.servlet.multipart.max-request-size` | `50MB` | default, docker | application properties files |
| `app.file-upload.max-file-size-bytes` | `10485760` | default, docker | application properties files |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default, docker | application properties files |
| `app.file-upload.max-files-per-upload` | `10` | default, docker | application properties files |
| `logging.level.com.photoalbum` | `DEBUG` (default) / `INFO` (docker) | default, docker | application properties files |
| `logging.level.org.springframework.web` | `DEBUG` (default) / `WARN` (docker) | default, docker | application properties files |
| `logging.level.org.hibernate.SQL` | `DEBUG` | docker | `application-docker.properties` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| photo-album | No explicit JVM args declared in repository | Not specified in repo | Not specified in repo |

## Startup Dependency Chain

1. `photo-album` service starts and initializes Spring context.
2. The application requires Oracle database availability before datasource initialization completes.
3. Once datasource and JPA startup succeed, web endpoints become available on configured server port.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `spring.datasource.password` | Database credential | Plain property in config file (`[MASKED]`) |
| `spring.datasource.username` | Database credential | Plain property in config file (`[MASKED]`) |

### Secrets Provisioning Workflow

Secrets are currently sourced from static property files committed with environment settings. No external secret store, managed identity, or vault-based retrieval path is configured in this repository. The application reads datasource credentials at startup through Spring configuration binding.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Java | 1.8 | `pom.xml` properties |
| Spring Boot | 2.7.18 | `pom.xml` parent |
| Spring Framework | 5.3.x (managed by Boot 2.7.18) | Spring Boot dependency management |
| Hibernate | 5.6.x (managed by Boot 2.7.18) | Spring Boot dependency management |
| Maven | Project uses Maven (wrapper not present) | `pom.xml` |
| Bootstrap (frontend) | 5.3.0 | CDN reference in templates |
