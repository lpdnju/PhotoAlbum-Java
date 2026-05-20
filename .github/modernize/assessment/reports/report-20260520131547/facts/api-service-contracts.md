# API & Service Communication Contracts

This application exposes a compact HTTP API surface over a single Spring MVC service, with primarily synchronous request-response communication between browser clients and server controllers.

## Service Catalog

| Service | Port | Category | Purpose |
|---|---:|---|---|
| Photo Album Web App (`photo-album`) | 8080 | API Layer + Business | Serves gallery/detail pages, accepts uploads, and streams photo content |

## API Endpoints Inventory

| Service | Method | Path | Request Type | Response Type |
|---|---|---|---|---|
| Photo Album Web App | GET | `/` | None | Thymeleaf HTML view (`index`) |
| Photo Album Web App | POST | `/upload` | Multipart form with `files` collection | JSON object with `success`, `uploadedPhotos`, `failedUploads` |
| Photo Album Web App | GET | `/detail/{id}` | Path parameter `id` | Thymeleaf HTML view (`detail`) or redirect |
| Photo Album Web App | POST | `/detail/{id}/delete` | Path parameter `id` | Redirect to `/` with flash message |
| Photo Album Web App | GET | `/photo/{id}` | Path parameter `id` | Binary resource (`image/*`) with headers |

## Management & Observability Endpoints

| Service | Endpoint | Custom Metrics (if any) |
|---|---|---|
| Photo Album Web App | None explicitly configured in source | None detected |

## DTOs & Contracts

The API contract is centered on the domain model `Photo` plus upload response wrapper `UploadResult`. `Photo` acts as both persistence model and response payload source for HTML/JSON rendering paths, while `UploadResult` encapsulates per-file upload outcomes before response aggregation in `HomeController`. No dedicated immutable DTO layer, OpenAPI specification, protobuf schema, or GraphQL schema is present. Serialization for JSON responses relies on Spring Boot's default Jackson stack.

## Communication Patterns

All discovered communication is synchronous HTTP between browser clients and controller endpoints, followed by in-process service and repository invocation. There are no asynchronous messaging patterns, service discovery components, API gateway layers, or circuit-breaker/retry libraries configured. Startup dependency concerns are minimal: API availability depends on successful database connectivity at application startup. Security posture at API contract level is open by default—no explicit authentication, authorization, or TLS enforcement is configured in the application code.

## Service Technology Matrix

| Service | Web | Data Access | Discovery | Gateway | Actuator | Cache | Metrics |
|---|---|---|---|---|---|---|---|
| Photo Album Web App | Spring MVC + Thymeleaf | Spring Data JPA (Oracle) | None | No | Not configured | None detected | None detected |

## Service Communication Sequence

```mermaid
sequenceDiagram
    participant User as "Browser User"
    participant Js as "upload.js"
    participant Home as "HomeController"
    participant Detail as "DetailController"
    participant FileCtrl as "PhotoFileController"
    participant Service as "PhotoServiceImpl"
    participant Repo as "PhotoRepository"
    participant DB as "Oracle Database"

    User->>Js: Select or drop image files
    Js->>Home: POST /upload (multipart files)
    Home->>Service: uploadPhoto(file)
    Service->>Repo: save(Photo)
    Repo->>DB: INSERT photo metadata + BLOB
    DB-->>Repo: Persisted row
    Repo-->>Service: Saved Photo
    Service-->>Home: UploadResult(success/photoId)
    Home-->>Js: JSON upload summary

    User->>Detail: GET /detail/id
    Detail->>Service: getPhotoById(id)
    Service->>Repo: findById(id)
    Repo->>DB: SELECT photo
    DB-->>Repo: Photo row
    Repo-->>Service: Optional<Photo>
    Service-->>Detail: Photo details
    Detail-->>User: Render detail page

    User->>FileCtrl: GET /photo/id
    FileCtrl->>Service: getPhotoById(id)
    Service->>Repo: findById(id)
    Repo->>DB: SELECT BLOB content
    DB-->>Repo: Photo bytes
    Repo-->>Service: Photo
    Service-->>FileCtrl: Photo
    FileCtrl-->>User: Binary image response
```
