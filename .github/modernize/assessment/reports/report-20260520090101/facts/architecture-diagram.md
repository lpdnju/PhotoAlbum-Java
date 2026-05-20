# Architecture Diagram

This application is a monolithic Spring Boot web app that serves a photo gallery UI and manages photo lifecycle operations backed by Oracle.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Web Browser"]
    end
    subgraph App["Application Layer - Spring Boot 2.7"]
        MVC["Spring MVC Controllers"]
        Svc["PhotoService"]
        View["Thymeleaf Templates"]
    end
    subgraph Data["Data Layer"]
        Repo["Spring Data JPA Repository"]
        Oracle[("Oracle Database")]
    end

    Browser -->|"HTTP requests"| MVC
    MVC -->|"render views"| View
    MVC -->|"invoke business logic"| Svc
    Svc -->|"CRUD and queries"| Repo
    Repo -->|"SQL and BLOB storage"| Oracle
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Presentation | Spring MVC + Thymeleaf | Spring Boot 2.7.18 | Handles page rendering and request routing |
| Business | PhotoService | App code | Upload validation, navigation, deletion |
| Data Access | Spring Data JPA + Hibernate | Spring Boot managed | Persistence abstraction for photo entities |
| Database | Oracle | JDBC ojdbc8 runtime | Stores photo metadata and binary image content |

### Data Storage & External Services

The application stores all photo metadata and image binary data (BLOB) in an Oracle database. No external API integrations or message brokers were detected.

### Key Architectural Decisions

- Uses a layered monolith (controller → service → repository).
- Persists image binary payloads directly in Oracle instead of filesystem/object storage.
- Uses Thymeleaf server-side rendering for the gallery and detail pages.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation
        HomeCtrl["HomeController"]
        DetailCtrl["DetailController"]
        FileCtrl["PhotoFileController"]
    end
    subgraph Business["Business Logic"]
        PhotoSvc["PhotoServiceImpl"]
    end
    subgraph DataAccess["Data Access"]
        PhotoRepo["PhotoRepository"]
        PhotoEntity["Photo Entity"]
    end
    subgraph Infra["Infrastructure"]
        Tx["Spring Transaction Management"]
        Log["SLF4J Logging"]
    end

    HomeCtrl -->|"list and upload"| PhotoSvc
    DetailCtrl -->|"detail, next/prev, delete"| PhotoSvc
    FileCtrl -->|"binary photo fetch"| PhotoSvc
    PhotoSvc -->|"query and persist"| PhotoRepo
    PhotoRepo -->|"map rows"| PhotoEntity
    Tx -.->|"transaction boundary"| PhotoSvc
    Log -.->|"cross-cutting logging"| Presentation
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| HomeController | Presentation | MVC Controller | Gallery page and multi-file upload endpoint |
| DetailController | Presentation | MVC Controller | Photo detail view and delete action |
| PhotoFileController | Presentation | MVC Controller | Serves photo bytes by id |
| PhotoServiceImpl | Business Logic | Service | Validation, metadata extraction, photo lifecycle logic |
| PhotoRepository | Data Access | JpaRepository | Oracle-backed photo queries and persistence |
| Photo | Data Access | JPA Entity | Photo metadata plus BLOB payload model |
