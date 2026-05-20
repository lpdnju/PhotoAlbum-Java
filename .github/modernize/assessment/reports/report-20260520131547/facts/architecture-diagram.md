# Architecture Diagram

This document summarizes the current Photo Album application structure and the key component interactions discovered from the codebase.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Web Browser"]
    end

    subgraph App["Application Layer - Spring Boot 2.7"]
        ThymeleafUI["Thymeleaf Views"]
        Controllers["MVC Controllers"]
        PhotoService["PhotoService"]
    end

    subgraph Data["Data Layer"]
        SpringData["Spring Data JPA"]
        Oracle[("Oracle Database")]
        Blob[("PHOTOS table BLOB data")]
    end

    subgraph External["External Dependencies"]
        BootstrapCDN["Bootstrap CDN"]
    end

    Browser -->|"HTTP requests"| Controllers
    Controllers -->|"Render views"| ThymeleafUI
    Controllers -->|"Business operations"| PhotoService
    PhotoService -->|"Repository access"| SpringData
    SpringData -->|"SQL queries"| Oracle
    Oracle -->|"Stores metadata and image bytes"| Blob
    Browser -->|"Loads UI assets"| BootstrapCDN
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Presentation | Spring MVC + Thymeleaf | Spring Boot 2.7.18 | Serves gallery/detail pages and handles form uploads |
| Business | Spring Service Layer | Spring Framework 5.3.x (via Boot 2.7.18) | Validates uploads, coordinates persistence, navigation, and deletion |
| Data Access | Spring Data JPA + Hibernate | Hibernate 5.6.x (via Boot 2.7.18) | Maps `Photo` entity and executes repository queries |
| Database | Oracle | Configured via JDBC driver `ojdbc8` | Stores photo metadata and binary image content |
| Client Assets | Bootstrap | 5.3.0 (CDN) | UI styling and layout |

### Data Storage & External Services

The application persists all photo metadata and file bytes in an Oracle database table (`PHOTOS`) through JPA/Hibernate. There is no separate cache, message broker, or external API integration in the current implementation; the primary external runtime dependency is the Bootstrap CDN used by the web UI.

### Key Architectural Decisions

- Uses a classic layered Spring MVC design with controllers delegating to a single photo-focused service.
- Stores image binaries directly in database BLOB columns instead of external object storage.
- Uses Thymeleaf server-side rendering for both gallery and detail experiences.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation["Presentation"]
        HomeController["HomeController"]
        DetailController["DetailController"]
        PhotoFileController["PhotoFileController"]
        UploadJs["upload.js"]
    end

    subgraph Business["Business Logic"]
        PhotoServiceApi["PhotoService"]
        PhotoServiceImpl["PhotoServiceImpl"]
    end

    subgraph DataAccess["Data Access"]
        PhotoRepository["PhotoRepository"]
        PhotoEntity["Photo Entity"]
    end

    subgraph Infra["Infrastructure"]
        OracleDB["Oracle DB"]
        Logger["SLF4J Logging"]
    end

    UploadJs -->|"POST /upload"| HomeController
    HomeController -->|"delegates"| PhotoServiceApi
    DetailController -->|"delegates"| PhotoServiceApi
    PhotoFileController -->|"delegates"| PhotoServiceApi
    PhotoServiceApi -->|"implemented by"| PhotoServiceImpl
    PhotoServiceImpl -->|"queries and saves"| PhotoRepository
    PhotoRepository -->|"maps rows to"| PhotoEntity
    PhotoRepository -->|"native SQL"| OracleDB
    HomeController -.->|"logs"| Logger
    DetailController -.->|"logs"| Logger
    PhotoFileController -.->|"logs"| Logger
    PhotoServiceImpl -.->|"logs"| Logger
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| `HomeController` | Presentation | MVC Controller | Renders gallery and handles multi-file upload API |
| `DetailController` | Presentation | MVC Controller | Displays photo detail, navigation, and deletion |
| `PhotoFileController` | Presentation | MVC Controller | Streams image bytes by photo ID |
| `upload.js` | Presentation | Frontend Script | Handles drag-drop selection, client validation, and async upload |
| `PhotoService` | Business | Service Interface | Defines photo upload/query/delete/navigation operations |
| `PhotoServiceImpl` | Business | Service Implementation | Executes validation, image dimension extraction, and persistence orchestration |
| `PhotoRepository` | Data Access | Spring Data Repository | Executes CRUD and custom native Oracle queries |
| `Photo` | Data Access | JPA Entity | Represents persisted photo metadata and BLOB content |
| Oracle DB | Infrastructure | Database | Stores all application photo data |
| SLF4J Logger | Infrastructure | Cross-cutting | Captures operational and error logs across layers |
