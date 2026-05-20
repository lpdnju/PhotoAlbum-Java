# Configuration & Externalized Settings Inventory

Configuration is file-based and profile-driven, primarily through Spring Boot property files with environment-specific Oracle connection settings.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Spring application config | Properties | `src/main/resources/application.properties` | Default runtime settings |
| Spring profile config | Properties | `src/main/resources/application-docker.properties` | Docker profile overrides |
| Build config | Maven POM | `pom.xml` | Dependencies and build plugin config |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| default Maven build | `mvn ...` execution | Compile/package Spring Boot app | `spring-boot-maven-plugin` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | implicit | `application.properties` | Oracle URL `FREEPDB1`, DEBUG logging |
| docker | `spring.profiles.active=docker` | `application-docker.properties` | Oracle URL `XE`, adjusted logging levels |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `server.port` | `8080` | default,docker | property file |
| `spring.datasource.url` | Oracle JDBC URL | default,docker (different values) | property file |
| `spring.datasource.username` | `photoalbum` | default,docker | property file |
| `spring.datasource.password` | `[MASKED]` | default,docker | property file |
| `spring.datasource.driver-class-name` | `oracle.jdbc.OracleDriver` | default,docker | property file |
| `spring.jpa.hibernate.ddl-auto` | `create` | default,docker | property file |
| `spring.servlet.multipart.max-file-size` | `10MB` | default,docker | property file |
| `spring.servlet.multipart.max-request-size` | `50MB` | default,docker | property file |
| `app.file-upload.allowed-mime-types` | `image/jpeg,image/png,image/gif,image/webp` | default,docker | property file |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| photo-album | None explicitly configured in repo | Not specified | Not specified |

## Startup Dependency Chain

1. photo-album application starts and initializes Spring context.
2. Data source initialization requires reachable Oracle database endpoint (`oracle-db`).
3. Repository and web layers become available after datasource/JPA initialization succeeds.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `spring.datasource.password` | Database credential | Application property file `[MASKED]` |

### Secrets Provisioning Workflow

Secrets are currently configured as static property values in local configuration files. No managed secret store integration (for example Key Vault) or managed identity flow was detected.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Java | 1.8 | `pom.xml` properties |
| Spring Boot | 2.7.18 | parent POM |
| Spring Framework | 5.x (via Boot BOM) | dependency management |
| Maven | project-managed build tool | `pom.xml` |
