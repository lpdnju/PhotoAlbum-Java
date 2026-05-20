# Architecture Diagram

This application is a single Spring Boot web service that provides photo upload, gallery viewing, and image serving capabilities backed by Oracle Database storage.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Web Browser"]
    end

    subgraph App["Application Layer - Spring Boot 2.7"]
        MVC["Spring MVC Controllers"]
        Thymeleaf["Thymeleaf Views"]
        Service["PhotoService"]
    end

    subgraph Data["Data Layer"]
        JPA["Spring Data JPA"]
        Oracle[("Oracle Database")]
    end

    subgraph External["External Infrastructure"]
        Docker["Docker Compose Oracle Container"]
    end

    Browser -->|"HTTP requests"| MVC
    MVC -->|"render pages"| Thymeleaf
    MVC -->|"delegate operations"| Service
    Service -->|"CRUD and queries"| JPA
    JPA -->|"SQL via JDBC"| Oracle
    Docker -->|"hosts database service"| Oracle
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Presentation | Spring MVC + Thymeleaf | Spring Boot 2.7.18 | Serve gallery/detail pages and upload interactions |
| Business | PhotoService / PhotoServiceImpl | Project code | Validate uploads and orchestrate persistence |
| Data Access | Spring Data JPA + Hibernate | Via Spring Boot 2.7.18 | Repository abstraction and ORM mapping |
| Database | Oracle Database + ojdbc8 | Runtime driver from Maven BOM | Persist photo metadata and BLOB image data |

### Data Storage & External Services

The application stores image binaries and metadata in a single Oracle table (`photos`) through JPA. No external APIs, message brokers, or cache services are integrated.

### Key Architectural Decisions

- Uses a layered MVC pattern (controllers → service → repository) to isolate web and persistence concerns.
- Stores uploaded image bytes directly in Oracle BLOB columns instead of filesystem/object storage.
- Uses profile-based database configuration (`default` and `docker`) for local/runtime environment switching.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation["Presentation"]
        HomeCtrl["HomeController"]
        DetailCtrl["DetailController"]
        FileCtrl["PhotoFileController"]
    end

    subgraph Business["Business Logic"]
        PhotoSvc["PhotoService"]
        PhotoSvcImpl["PhotoServiceImpl"]
    end

    subgraph DataAccess["Data Access"]
        Repo["PhotoRepository"]
        Entity["Photo Entity"]
    end

    subgraph Infrastructure["Infrastructure"]
        OracleDB["Oracle Database"]
        Logging["SLF4J Logging"]
    end

    HomeCtrl -->|"upload/list"| PhotoSvc
    DetailCtrl -->|"view/delete"| PhotoSvc
    FileCtrl -->|"serve binary"| PhotoSvc
    PhotoSvc -->|"implemented by"| PhotoSvcImpl
    PhotoSvcImpl -->|"query/save/delete"| Repo
    Repo -->|"maps records"| Entity
    Repo -->|"native SQL and JPA"| OracleDB
    HomeCtrl -.->|"logs failures"| Logging
    DetailCtrl -.->|"logs actions"| Logging
    FileCtrl -.->|"logs serving"| Logging
    PhotoSvcImpl -.->|"logs validation and DB events"| Logging
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| HomeController | Presentation | MVC Controller | Gallery page rendering and multi-file upload endpoint |
| DetailController | Presentation | MVC Controller | Single-photo view navigation and delete action |
| PhotoFileController | Presentation | MVC Controller | Stream image bytes by photo ID |
| PhotoService | Business | Service Interface | Defines upload/query/delete/navigation contracts |
| PhotoServiceImpl | Business | Service Implementation | Enforces upload rules and orchestrates persistence |
| PhotoRepository | Data Access | Spring Data Repository | Executes JPA and native Oracle photo queries |
| Photo | Data Access | JPA Entity | Photo metadata and BLOB payload persistence model |
