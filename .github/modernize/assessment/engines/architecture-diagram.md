# Architecture Diagram

This application is a single-service Spring Boot photo gallery with server-rendered pages and file upload endpoints. It persists photo metadata and binary data in Oracle using JPA.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Web Browser"]
    end
    subgraph App["Application Layer - Spring Boot 2.7"]
        MVC["Spring MVC Controllers"]
        Service["PhotoService"]
        View["Thymeleaf Views"]
    end
    subgraph Data["Data Layer"]
        Repo["Spring Data JPA Repository"]
        Oracle[("Oracle Database")]
    end
    subgraph External["External Dependencies"]
        DockerOracle["Oracle container in docker-compose"]
    end

    Browser -->|"HTTP requests"| MVC
    MVC -->|"model + view"| View
    MVC -->|"invoke business operations"| Service
    Service -->|"CRUD + custom queries"| Repo
    Repo -->|"SQL/native queries"| Oracle
    DockerOracle -->|"provides DB runtime"| Oracle
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Presentation | Spring MVC + Thymeleaf | Spring Boot 2.7.18 | Render gallery and detail pages |
| Business | Spring Service + Transactions | Spring Framework 5.x (via Boot 2.7.18) | Upload validation and photo lifecycle operations |
| Data Access | Spring Data JPA + Hibernate | Spring Data JPA (via Boot 2.7.18) | Persistence and native SQL querying |
| Data Store | Oracle | Oracle Free (docker image) / ojdbc8 runtime | Store photo metadata and BLOB content |

### Data Storage & External Services

The application uses a single Oracle database as the system of record. The `photos` table stores both metadata and binary image data (`@Lob`). No external API or message broker integration is present; the only external runtime dependency is the Oracle database service in Docker Compose.

### Key Architectural Decisions

- Uses a layered monolith (controller → service → repository) with constructor-based dependency injection.
- Stores image bytes directly in Oracle BLOB columns instead of object storage or filesystem paths.
- Uses native Oracle SQL in repository methods for ordering, pagination, and Oracle-specific functions.

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
        Tx["Spring Transaction Manager"]
        Logger["SLF4J Logging"]
    end

    HomeCtrl -->|"upload/list requests"| PhotoSvc
    DetailCtrl -->|"detail/delete requests"| PhotoSvc
    FileCtrl -->|"binary read request"| PhotoSvc
    PhotoSvc -->|"persist/query"| PhotoRepo
    PhotoRepo -->|"maps rows"| PhotoEntity
    Tx -.->|"transaction boundaries"| PhotoSvc
    Logger -.->|"cross-cutting logs"| Presentation
    Logger -.->|"cross-cutting logs"| Business
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| HomeController | Presentation | MVC Controller | Gallery page rendering and multi-file upload endpoint |
| DetailController | Presentation | MVC Controller | Single-photo view and delete flow |
| PhotoFileController | Presentation | MVC Controller | Serves stored photo bytes with media type headers |
| PhotoServiceImpl | Business | Spring Service | File validation, metadata extraction, transactional orchestration |
| PhotoRepository | Data Access | Spring Data JPA Repository | Native SQL/JPA CRUD for photo retrieval and navigation |
| Photo | Data Access | JPA Entity | Photo metadata + BLOB payload mapping |
```
