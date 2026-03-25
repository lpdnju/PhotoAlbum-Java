# Project Facts Report

**Project:** Photo Album  
**Generated:** 2026-03-25  
**Assessment Basis:** 36 automated fact skills

---

## Application Identity

| Fact | Finding | Confidence |
|------|---------|------------|
| **Application Name** | Photo Album (`photo-album` artifact, `com.photoalbum` group) | High |
| **Application Type** | Web App with REST API (Spring MVC + Thymeleaf server-side rendering) | High |
| **Application Version** | 1.0.0 (pom.xml), Spring Boot 2.7.18 parent | High |
| **Architecture Pattern** | Layered Monolith with MVC — Controller / Service / Repository / Model layers | High |

---

## Runtime & Language

| Fact | Finding | Confidence |
|------|---------|------------|
| **Runtime Environment** | Java 8 (JRE) via Eclipse Temurin (`eclipse-temurin:8-jre`) | High |
| **Servlet Container** | Embedded Apache Tomcat 9.x (Servlet 4.0 / Java EE 8) via `spring-boot-starter-web` | High |
| **Language Dependencies** | 9 declared Maven dependencies (7 runtime, 2 test-only); fat JAR deployment | High |
| **XML Configs** | None — pure Spring Boot annotation-based configuration | High |
| **Embedded Language Usage** | None — pure Java; `upload.js` is a client-side static asset only | High |
| **Startup Instrumentation** | SLF4J + Logback (Spring Boot default); no APM/telemetry; no AOP | High |
| **Testing Framework** | JUnit 5 (Jupiter) + Spring Boot Test; 1 test file, 1 smoke test (`contextLoads`) | High |

---

## Container & Infrastructure

| Fact | Finding | Confidence |
|------|---------|------------|
| **Container Engine** | Docker with Docker Compose v2 | High |
| **Container Version** | Docker Compose Specification v2 (no `version:` key); requires Docker Engine 20.10+ | Medium |
| **Base Image** | Final: `eclipse-temurin:8-jre`; Build: `maven:3.9.6-eclipse-temurin-8` | High |
| **Multi-stage Build** | Yes — 2 stages: build (Maven) → runtime (JRE only) | High |
| **Image Size** | ~250–300 MB (eclipse-temurin:8-jre base ~220 MB + fat JAR ~50–80 MB) | Medium |
| **Image Layers** | ~8–11 layers in final image (3 Dockerfile instructions + base layers) | High |
| **System Packages** | None installed — final stage uses base image only | High |
| **Orchestration Tool** | Docker Compose only; no Kubernetes, Swarm, or Helm | High |

---

## Service Definition & Networking

| Fact | Finding | Confidence |
|------|---------|------------|
| **Service Definition** | docker-compose.yml with 2 services: `oracle-db`, `photoalbum-java-app` | High |
| **Application Port** | 8080 (HTTP) — consistent across Dockerfile EXPOSE, application.properties, and docker-compose | High |
| **Network Settings** | Custom bridge network `photoalbum-network`; DNS-based inter-service communication | High |
| **Volume Mounts** | `oracle_data` (named volume → `/opt/oracle/oradata`) + `./oracle-init` (bind mount for init scripts) | High |
| **Resource Limits** | No container-level CPU/memory limits; JVM heap capped at 512 MB via `JAVA_OPTS` | High |
| **Health Checks** | docker-compose healthcheck on `oracle-db` only (30s interval, 10s timeout, 15 retries) | High |

---

## Configuration & Profiles

| Fact | Finding | Confidence |
|------|---------|------------|
| **Profile Settings** | 3 Spring profiles: `default`, `docker` (activated via `SPRING_PROFILES_ACTIVE`), `test` (H2 DB) | High |
| **Environment Variables** | 8 env vars: `JAVA_OPTS`, `SPRING_PROFILES_ACTIVE`, datasource URL/user/password, Oracle credentials — hardcoded in docker-compose.yml | High |
| **Communication Protocols** | HTTP (port 8080) for client-server; JDBC/TCP (port 1521) for Oracle DB internally | High |

---

## External Dependencies & Services

| Fact | Finding | Confidence |
|------|---------|------------|
| **External Services** | Oracle Database Free 23ai — sole external service (JDBC on port 1521) | High |
| **External Dependencies** | Oracle Database only; no Redis, message queue, search engine, or object storage | High |

---

## Operating System & Hardware

| Fact | Finding | Confidence |
|------|---------|------------|
| **Operating System** | Linux (Ubuntu-based) via eclipse-temurin base; supports x86_64 and ARM64 | High |
| **Hardware Requirements** | Min 4 GB RAM for Oracle DB container; 2 CPU threads; ~2 GB disk (per README) | High |

---

## Security & Compliance

| Fact | Finding | Confidence |
|------|---------|------------|
| **Security Implementation** | Minimal — HTTP only, no auth/authz, no TLS, no encryption; MIME-type + file-size validation for uploads; credentials hardcoded in docker-compose | High |
| **Compliance Requirements** | None detected — no GDPR, HIPAA, PCI-DSS, or SOX indicators | Medium |
| **Data Classification** | Internal — stores photo BLOBs and technical metadata only; no PII, PHI, or PCI data | High |
| **Licensing Information** | MIT License (Copyright Microsoft Corporation); Spring Boot (Apache 2.0); ojdbc8 (Oracle Technology Network License) | High |

---

## Key Migration Considerations

1. **Java 8 / Spring Boot 2.7.x** — Both are approaching or past end-of-life. Migration to Java 17/21 and Spring Boot 3.x is recommended.
2. **Oracle Database dependency** — Tight coupling to Oracle-specific SQL (ROWNUM, TO_CHAR, analytical functions) and `ojdbc8` driver. Migration to Azure Database or Azure-compatible DB requires query rewriting.
3. **No security controls** — Lacks authentication, HTTPS, and secrets management. Credentials hardcoded in docker-compose; must be externalized before cloud deployment.
4. **Single container / no Kubernetes** — Currently Docker Compose only; containerized and ready for AKS or Azure Container Apps with minimal changes.
5. **BLOB storage in Oracle** — Photos stored as database BLOBs; consider migrating to Azure Blob Storage for scalability and cost efficiency.
6. **No resource limits** — Container has no CPU/memory bounds; must be set for production cloud deployment.
7. **Minimal test coverage** — Only 1 smoke test; no unit or integration test suite.
