# Architecture Diagram

This application is a single Spring Boot web service that renders Thymeleaf pages and persists photo metadata plus image blobs into Oracle. The architecture is layered around controllers, a photo service, and a JPA repository.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Web Browser"]
    end

    subgraph App["Application Layer - Spring Boot 2.7"]
        MVC["Spring MVC Controllers"]
        View["Thymeleaf Views"]
        Service["PhotoService"]
        Upload["Multipart Upload Validation"]
    end

    subgraph Data["Data Layer"]
        JPA["Spring Data JPA"]
        Oracle[("Oracle Database PHOTOS table")]
    end

    subgraph External["External Services"]
        OracleContainer["Oracle Free Container"]
    end

    Browser -->|"HTTP requests"| MVC
    MVC -->|"render pages"| View
    MVC -->|"upload, read, delete actions"| Service
    Service -->|"validate MIME and size"| Upload
    Service -->|"CRUD and native SQL"| JPA
    JPA -->|"SQL and BLOB operations"| Oracle
    Oracle -->|"provisioned by"| OracleContainer
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Presentation | Spring MVC + Thymeleaf | Spring Boot 2.7.18 | Serves gallery/detail views and handles upload/delete requests |
| Business Logic | Spring Service + Transaction Management | Spring Framework 5.3.x (via Boot 2.7.18) | Validates uploads and orchestrates persistence operations |
| Data Access | Spring Data JPA + Hibernate | Hibernate 5.6.x (via Boot 2.7.18) | Maps `Photo` entity and executes repository queries |
| Storage | Oracle Database | Oracle Free image (docker-compose) | Stores photo metadata and binary photo data |

### Data Storage & External Services

The application uses one relational data store (Oracle) with a single `Photo` aggregate persisted through JPA. No cache, message broker, or third-party API integration is present; the only external dependency is the Oracle database container/runtime.

### Key Architectural Decisions

- Uses a simple monolith pattern with server-side rendered views instead of separate frontend and backend services.
- Stores photo binary content directly in the database as BLOB (`photo_data`) rather than file-system storage.
- Keeps business logic in a dedicated service layer with `@Transactional` boundaries around repository operations.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation["Presentation"]
        HomeCtrl["HomeController"]
        DetailCtrl["DetailController"]
        FileCtrl["PhotoFileController"]
    end

    subgraph Business["Business Logic"]
        PhotoSvc["PhotoServiceImpl"]
        UploadResult["UploadResult DTO"]
    end

    subgraph DataAccess["Data Access"]
        PhotoRepo["PhotoRepository"]
        PhotoEntity["Photo Entity"]
    end

    subgraph Infra["Infrastructure"]
        Tx["Spring Transaction Interceptor"]
        Log["SLF4J Logging"]
    end

    HomeCtrl -->|"list photos"| PhotoSvc
    HomeCtrl -->|"upload files"| PhotoSvc
    DetailCtrl -->|"load and delete photo"| PhotoSvc
    FileCtrl -->|"serve photo bytes"| PhotoSvc
    PhotoSvc -->|"returns status"| UploadResult
    PhotoSvc -->|"save/find/delete"| PhotoRepo
    PhotoRepo -->|"maps records"| PhotoEntity
    Tx -.->|"transaction boundary"| PhotoSvc
    Log -.->|"application logging"| Presentation
    Log -.->|"service logging"| Business
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| HomeController | Presentation | Spring MVC Controller | Serves gallery page and handles multi-file upload response payload |
| DetailController | Presentation | Spring MVC Controller | Shows single-photo detail and deletes selected photo |
| PhotoFileController | Presentation | Spring MVC Controller | Streams photo bytes from database to HTTP response |
| PhotoServiceImpl | Business Logic | Spring Service | Enforces file validation and orchestrates repository operations |
| UploadResult | Business Logic | DTO/Model | Represents upload outcome and error/success metadata |
| PhotoRepository | Data Access | Spring Data JPA Repository | Executes CRUD plus custom native SQL queries |
| Photo | Data Access | JPA Entity | Represents persisted photo metadata and BLOB content |
```
