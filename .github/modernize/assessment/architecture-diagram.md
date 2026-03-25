# Architecture Diagram

A Spring Boot 2.7 photo album application using Thymeleaf for server-side rendering, Spring Data JPA for data access, and Oracle Database for persistent photo (BLOB) storage, deployed via Docker Compose.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Web Browser"]
    end
    subgraph App["Application Layer - Spring Boot 2.7 / Java 8"]
        Web["Spring MVC + Thymeleaf"]
        FileCtrl["Photo File Serving"]
        Service["PhotoService - Business Logic"]
        Validation["Bean Validation + ImageIO"]
    end
    subgraph Data["Data Layer"]
        JPA["Spring Data JPA + Hibernate"]
        DB[("Oracle Database Free")]
    end
    subgraph Container["Container Infrastructure"]
        AppContainer["photoalbum-java-app"]
        DBContainer["photoalbum-oracle"]
        Network["photoalbum-network (bridge)"]
        Volume[("oracle_data volume")]
    end

    Browser -->|"HTTP GET / POST /upload"| Web
    Browser -->|"GET /photo/{id}"| FileCtrl
    Web -->|"delegates"| Service
    FileCtrl -->|"delegates"| Service
    Service -->|"validates + stores BLOB"| Validation
    Service -->|"CRUD operations"| JPA
    JPA -->|"native Oracle SQL"| DB
    DB --- Volume
    AppContainer --- Network
    DBContainer --- Network
```

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation["Presentation Layer"]
        HomeCtrl["HomeController"]
        DetailCtrl["DetailController"]
        PhotoFileCtrl["PhotoFileController"]
    end
    subgraph Business["Business Logic"]
        PhotoSvc["PhotoService (interface)"]
        PhotoSvcImpl["PhotoServiceImpl"]
        MathUtil["MathUtil"]
    end
    subgraph DataAccess["Data Access"]
        PhotoRepo["PhotoRepository"]
        PhotoEntity["Photo (Entity)"]
        UploadResult["UploadResult (DTO)"]
    end

    HomeCtrl -->|"upload / list"| PhotoSvc
    DetailCtrl -->|"detail / delete / navigate"| PhotoSvc
    PhotoFileCtrl -->|"serve BLOB"| PhotoSvc
    PhotoSvc -->|"implemented by"| PhotoSvcImpl
    PhotoSvcImpl -->|"uses"| MathUtil
    PhotoSvcImpl -->|"queries"| PhotoRepo
    PhotoRepo -->|"maps to"| PhotoEntity
    PhotoSvcImpl -->|"returns"| UploadResult
```
