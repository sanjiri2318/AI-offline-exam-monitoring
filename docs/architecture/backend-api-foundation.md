Backend API Foundation Architecture

1\. Purpose



The Backend API Foundation provides the initial REST API structure for the AI-Based Offline Examination Monitoring System.



It establishes the common API conventions, router organization, request/response handling, validation, authentication boundaries, error handling, and module structure required for the Backend implementation.



The API Foundation must remain aligned with:



docs/architecture/api-contract.md

docs/architecture/backend-foundation.md

docs/architecture/event-model.md

docs/architecture/database-schema.md



The Backend remains the single source of truth for application APIs and application state.



2\. API Architecture



The Backend API follows:



Client

&#x20; ↓

FastAPI Application

&#x20; ↓

API Router

&#x20; ↓

Request Validation

&#x20; ↓

Domain Logic

&#x20; ↓

Repository

&#x20; ↓

Database



For AI/CV events:



AI/CV Engine

&#x20;     ↓

Event Generation

&#x20;     ↓

Internal API

&#x20;     ↓

Backend

&#x20;     ↓

Event Processing

&#x20;     ↓

Database / Risk Engine



For real-time monitoring:



Backend

&#x20;   ↓

WebSocket

&#x20;   ↓

Frontend Dashboard

3\. API Technology



The proposed API stack is:



Python 3.11+

&#x20;   ↓

FastAPI

&#x20;   ↓

Pydantic

&#x20;   ↓

Uvicorn



FastAPI provides:



REST API routing

Request validation

Response validation

Dependency injection

OpenAPI documentation

WebSocket support

4\. API Base Path



Application APIs should use a versioned base path:



/api/v1



Module routes should follow the structure:



/api/v1/{module}



Examples:



/api/v1/auth

/api/v1/cameras

/api/v1/exams

/api/v1/students

/api/v1/seats

/api/v1/monitoring

/api/v1/events

/api/v1/incidents

/api/v1/risk

/api/v1/evidence

/api/v1/analytics

5\. Router Organization



The API layer should be modular.



Conceptually:



app/

└── api/

&#x20;   ├── auth/

&#x20;   ├── cameras/

&#x20;   ├── exams/

&#x20;   ├── students/

&#x20;   ├── seats/

&#x20;   ├── monitoring/

&#x20;   ├── events/

&#x20;   ├── incidents/

&#x20;   ├── risk/

&#x20;   ├── evidence/

&#x20;   ├── analytics/

&#x20;   └── internal/



Each module owns only its related endpoints.



6\. Application Router Registration



The main application should register API routers centrally.



Conceptually:



FastAPI Application

&#x20;      ↓

API Router Registration

&#x20;      ↓

Authentication Router

Camera Router

Exam Router

Student Router

Seat Router

Monitoring Router

Event Router

Incident Router

Risk Router

Evidence Router

Analytics Router

Internal Router



Routers should not be scattered throughout unrelated application modules.



7\. Authentication API



Authentication provides access to the monitoring system.



Initial conceptual endpoints:



POST /api/v1/auth/login

POST /api/v1/auth/logout

GET  /api/v1/auth/me



Responsibilities:



Authenticate users

Establish authenticated access

Return authenticated user information

Validate authentication credentials



The Backend owns authentication.



8\. Authentication Request



The login request should contain the required credentials.



Conceptually:



{

&#x20;   "username": "...",

&#x20;   "password": "..."

}



The exact field names must be finalized during implementation and must remain consistent with the API contract.



9\. Authentication Response



A successful authentication response may contain:



{

&#x20;   "success": true,

&#x20;   "data": {

&#x20;       "access\_token": "...",

&#x20;       "token\_type": "bearer",

&#x20;       "user": {

&#x20;           "user\_id": "...",

&#x20;           "username": "...",

&#x20;           "role": "FACULTY"

&#x20;       }

&#x20;   }

}



Sensitive credentials must never be returned.



10\. Authorization



Protected endpoints must verify:



Authentication

&#x20;     ↓

Authorization

&#x20;     ↓

Endpoint



The Backend determines whether the authenticated user has permission to perform an operation.



Possible roles include:



ADMIN

FACULTY



The exact role model must remain aligned with the database architecture.



11\. Camera API



The Camera API manages configured CCTV/IP cameras.



Conceptual endpoints:



GET    /api/v1/cameras

GET    /api/v1/cameras/{camera\_id}

POST   /api/v1/cameras

PUT    /api/v1/cameras/{camera\_id}

DELETE /api/v1/cameras/{camera\_id}



The API manages camera configuration and state.



Actual frame acquisition belongs to the AI/CV Camera Ingestion component.



12\. Camera Request



Camera creation/update information may contain:



{

&#x20;   "name": "...",

&#x20;   "room\_id": "...",

&#x20;   "stream\_url": "...",

&#x20;   "enabled": true

}



Credentials must be handled securely.



Sensitive RTSP credentials must not be exposed through normal responses.



13\. Camera Response



Camera responses may contain:



{

&#x20;   "camera\_id": "...",

&#x20;   "name": "...",

&#x20;   "room\_id": "...",

&#x20;   "status": "ONLINE",

&#x20;   "enabled": true

}



The response should not expose sensitive stream credentials.



14\. Examination API



The Examination API manages examination sessions.



Conceptual endpoints:



GET  /api/v1/exams

GET  /api/v1/exams/{exam\_id}

POST /api/v1/exams

PUT  /api/v1/exams/{exam\_id}



Exam information may include:



Exam name

Date

Start time

End time

Examination room

Status

15\. Student API



The Student API provides student information required by the monitoring system.



Conceptual endpoints:



GET /api/v1/students

GET /api/v1/students/{student\_id}



Student information may include:



student\_id

name

department

registration information

exam assignment



Only required student information should be exposed.



16\. Seat API



The Seat API manages examination seating.



Conceptual endpoints:



GET  /api/v1/seats

GET  /api/v1/seats/{seat\_id}

POST /api/v1/seats

PUT  /api/v1/seats/{seat\_id}



Seat information may include:



Seat ID

Room

Position

Assigned student

Assignment status

17\. Monitoring API



The Monitoring API provides current examination monitoring state.



Conceptual endpoints:



GET /api/v1/monitoring/status

GET /api/v1/monitoring/cameras

GET /api/v1/monitoring/students



Monitoring responses may include:



Active examination

Camera states

Student status

Active tracks

Risk state

Active incidents

Attention queue information

18\. Event API



The Event API provides access to stored monitoring events.



Conceptual endpoints:



GET /api/v1/events

GET /api/v1/events/{event\_id}



Events must follow the shared event model.



The Frontend receives events from the Backend rather than directly communicating with the AI/CV Engine.



19\. Internal Event API



The Internal API receives events from the AI/CV Engine.



Conceptual endpoint:



POST /internal/events



The endpoint is intended for trusted internal communication.



The request follows:



Event {

&#x20;   event\_id

&#x20;   event\_type

&#x20;   timestamp

&#x20;   camera\_id

&#x20;   track\_id

&#x20;   student\_id

&#x20;   confidence

&#x20;   metadata

&#x20;   related\_event\_ids

}

20\. Internal API Security



The internal event endpoint must not be treated as a normal public API.



The implementation should restrict access to trusted local/internal communication.



Possible mechanisms include:



Local network restriction

Internal authentication

Service credentials

Request validation



The final mechanism must be selected during implementation.



21\. Internal Event Processing



The Backend processes an incoming event through:



POST /internal/events

&#x20;       ↓

Authentication / Internal Validation

&#x20;       ↓

Schema Validation

&#x20;       ↓

Event Validation

&#x20;       ↓

Duplicate Check

&#x20;       ↓

Persistence

&#x20;       ↓

Risk Processing

&#x20;       ↓

WebSocket Update



Not every event necessarily results in an incident.



22\. Event Idempotency



The event\_id is used to prevent duplicate processing.



Example:



AI/CV

&#x20; ↓

E123

&#x20; ↓

Backend

&#x20; ↓

Stored



Retry

&#x20; ↓

E123

&#x20; ↓

Backend recognizes E123



Repeated delivery of the same event must not unintentionally create duplicate records.



23\. Risk API



The Risk API exposes Backend-generated risk assessments.



Conceptual endpoints:



GET /api/v1/risk

GET /api/v1/risk/{student\_id}



Risk information may include:



risk\_score

risk\_level

timestamp

student\_id

related\_events

explanation



The Frontend does not calculate the final risk score.



24\. Incident API



The Incident API manages reviewable monitoring incidents.



Conceptual endpoints:



GET   /api/v1/incidents

GET   /api/v1/incidents/{incident\_id}

PATCH /api/v1/incidents/{incident\_id}



Incident information may include:



Incident ID

Student

Camera

Timestamp

Risk level

Related events

Evidence

Review status



An incident is a monitoring record requiring human review.



25\. Evidence API



The Evidence API provides access to evidence metadata.



Conceptual endpoint:



GET /api/v1/evidence/{evidence\_id}



Evidence metadata may include:



evidence\_id

incident\_id

camera\_id

timestamp

evidence\_type

storage\_reference

created\_at



Actual evidence storage remains outside the API database layer.



26\. Analytics API



The Analytics API provides monitoring statistics.



Conceptual endpoints:



GET /api/v1/analytics/overview

GET /api/v1/analytics/events

GET /api/v1/analytics/incidents

GET /api/v1/analytics/risk



Analytics may include:



Event totals

Incident totals

Risk distribution

Camera statistics

Behaviour statistics

Examination statistics

27\. Request Validation



All incoming API requests must be validated before business logic executes.



Flow:



Request

&#x20;  ↓

Pydantic Schema

&#x20;  ↓

Validation

&#x20;  ↓

Domain Logic



Invalid requests should return structured validation errors.



28\. Response Validation



API responses should use defined response schemas.



Flow:



Domain Data

&#x20;  ↓

Response Schema

&#x20;  ↓

JSON Response



Internal database models should not automatically be returned directly to clients.



29\. Standard Success Response



Where a response wrapper is used, the standard structure should be:



{

&#x20;   "success": true,

&#x20;   "data": {}

}



For collections:



{

&#x20;   "success": true,

&#x20;   "data": \[],

&#x20;   "pagination": {}

}



The exact implementation must remain consistent across modules.



30\. Standard Error Response



API errors should use:



{

&#x20;   "success": false,

&#x20;   "error": {

&#x20;       "code": "INVALID\_REQUEST",

&#x20;       "message": "Request validation failed",

&#x20;       "details": {}

&#x20;   }

}



The structure should remain consistent across endpoints.



31\. HTTP Status Codes



The API should use appropriate HTTP status codes.



200 OK

201 Created

204 No Content

400 Bad Request

401 Unauthorized

403 Forbidden

404 Not Found

409 Conflict

422 Unprocessable Entity

500 Internal Server Error



Status codes must be used consistently.



32\. Not Found Handling



When a requested resource does not exist:



GET /api/v1/students/unknown



The Backend should return:



404 Not Found



with a structured error response.



33\. Authentication Failure



Invalid or missing authentication should return:



401 Unauthorized



The response must not expose sensitive authentication details.



34\. Authorization Failure



An authenticated user without sufficient permission should receive:



403 Forbidden



The Frontend may hide unavailable actions, but the Backend must still enforce authorization.



35\. Conflict Handling



Conflicting operations may return:



409 Conflict



Examples include:



Duplicate resource

Duplicate event

Conflicting seat assignment

Conflicting configuration

36\. Dependency Injection



FastAPI dependency injection should be used for shared Backend services where appropriate.



Potential dependencies include:



Database session

Authenticated user

Authorization checks

Repository access

Configuration

Internal service validation



Dependencies should remain reusable and centralized.



37\. Database Session Boundary



Database sessions should be managed by the Backend infrastructure layer.



Conceptually:



API Request

&#x20;   ↓

Database Session

&#x20;   ↓

Repository

&#x20;   ↓

Database

&#x20;   ↓

Session Cleanup



API routers should not create unmanaged database connections.



38\. Repository Boundary



API routers should not contain extensive database queries.



Preferred flow:



API Router

&#x20;   ↓

Domain / Service Logic

&#x20;   ↓

Repository

&#x20;   ↓

Database



This keeps business logic separate from HTTP handling.



39\. Business Logic Boundary



Business rules belong to the domain/service layer.



For example:



Event Received

&#x20;   ↓

Event Processing Logic

&#x20;   ↓

Risk Engine



The API router should coordinate the request rather than contain the entire risk-processing implementation.



40\. WebSocket API



The Backend provides a real-time WebSocket interface for the Frontend.



Conceptually:



WebSocket

&#x20;   ↓

/ws/monitoring



The exact WebSocket path must be finalized during implementation.



41\. WebSocket Message Structure



Messages should follow a predictable structure:



{

&#x20;   "type": "risk\_updated",

&#x20;   "timestamp": "...",

&#x20;   "data": {}

}



Supported message categories include:



camera\_status

detection\_event

risk\_updated

incident\_created

attention\_queue\_updated

student\_track\_update

42\. Camera Status WebSocket Message



Example:



{

&#x20;   "type": "camera\_status",

&#x20;   "timestamp": "...",

&#x20;   "data": {

&#x20;       "camera\_id": "CAM-01",

&#x20;       "status": "ONLINE"

&#x20;   }

}



Camera status originates from the Backend monitoring state.



43\. Detection Event WebSocket Message



Example:



{

&#x20;   "type": "detection\_event",

&#x20;   "timestamp": "...",

&#x20;   "data": {

&#x20;       "event\_id": "...",

&#x20;       "event\_type": "PHONE\_DETECTED",

&#x20;       "camera\_id": "CAM-01",

&#x20;       "track\_id": "105"

&#x20;   }

}



The Backend controls what event information is exposed.



44\. Risk Update WebSocket Message



Example:



{

&#x20;   "type": "risk\_updated",

&#x20;   "timestamp": "...",

&#x20;   "data": {

&#x20;       "student\_id": "...",

&#x20;       "risk\_score": 78,

&#x20;       "risk\_level": "HIGH"

&#x20;   }

}



Risk information originates from the Backend Risk Engine.



45\. Incident WebSocket Message



Example:



{

&#x20;   "type": "incident\_created",

&#x20;   "timestamp": "...",

&#x20;   "data": {

&#x20;       "incident\_id": "...",

&#x20;       "student\_id": "...",

&#x20;       "risk\_level": "HIGH"

&#x20;   }

}



The Backend generates the incident.



46\. Attention Queue WebSocket Message



Example:



{

&#x20;   "type": "attention\_queue\_updated",

&#x20;   "timestamp": "...",

&#x20;   "data": {

&#x20;       "queue": \[]

&#x20;   }

}



Queue priority is determined by the Backend Attention Queue.



47\. Student Track WebSocket Message



Example:



{

&#x20;   "type": "student\_track\_update",

&#x20;   "timestamp": "...",

&#x20;   "data": {

&#x20;       "camera\_id": "CAM-01",

&#x20;       "track\_id": "105",

&#x20;       "student\_id": "...",

&#x20;       "position": {}

&#x20;   }

}



Track information must remain aligned with the Tracking architecture.



48\. API Pagination



Collection endpoints should support pagination where datasets can become large.



Conceptually:



GET /api/v1/events?page=1\&page\_size=50



Response:



{

&#x20;   "success": true,

&#x20;   "data": \[],

&#x20;   "pagination": {

&#x20;       "page": 1,

&#x20;       "page\_size": 50,

&#x20;       "total": 100

&#x20;   }

}



The exact pagination implementation should be standardized across applicable endpoints.



49\. Filtering



Monitoring data may require filtering.



Examples:



GET /api/v1/events?camera\_id=CAM-01

GET /api/v1/incidents?risk\_level=HIGH

GET /api/v1/events?event\_type=PHONE\_DETECTED



Filter parameters must be documented before Frontend implementation.



50\. Date and Time Filtering



Event and incident APIs may support time-based filtering.



Conceptually:



GET /api/v1/events?start\_time=...\&end\_time=...



The Backend should use a consistent timestamp format.



All stored monitoring timestamps should be handled consistently across services.



51\. Sorting



Collection APIs may support controlled sorting.



Conceptually:



GET /api/v1/events?sort=timestamp\&order=desc



Only documented sortable fields should be accepted.



52\. API Documentation



FastAPI automatically provides OpenAPI documentation.



The API documentation should remain aligned with:



docs/architecture/api-contract.md



The generated API documentation must not become a replacement for the architecture contract.



53\. Health Endpoint



The Backend should provide a health endpoint.



Conceptually:



GET /health



Example:



{

&#x20;   "status": "healthy"

}



The endpoint should provide a simple indication that the Backend service is operational.



54\. Readiness Boundary



If dependency checks are implemented, readiness should distinguish between:



Backend process available



and:



Backend fully ready for monitoring



Database or required service failures should be reported clearly.



55\. CORS



The Backend should use controlled CORS configuration for the Frontend application.



Allowed origins should be configured rather than blindly allowing all origins in deployment environments.



56\. Configuration



API configuration should be externalized.



Potential configuration includes:



API host

API port

Database URL

Authentication settings

CORS origins

Internal API settings

WebSocket settings

Evidence storage

Logging



Environment-specific values must not be hardcoded.



57\. Secrets



The following must not be committed to Git:



Passwords

JWT secrets

Database credentials

Camera credentials

API keys

Private tokens



These values should be provided through environment/configuration mechanisms.



58\. Logging



API operations should produce structured logs where useful.



Important information includes:



Request processing failures

Authentication failures

Authorization failures

Database errors

Internal event failures

WebSocket failures

Unexpected application errors



Sensitive information must not be unnecessarily logged.



59\. Exception Handling



Unhandled application exceptions should be converted into controlled API responses.



The API must not expose:



Stack traces

Database credentials

Internal file paths

Secret configuration



to normal clients.



60\. Internal Event Error Handling



If an internal event is invalid:



AI/CV

&#x20; ↓

Invalid Event

&#x20; ↓

Backend Validation

&#x20; ↓

Structured Error



The event must not be silently accepted as valid.



61\. API Rate and Load Considerations



The Backend must support multiple concurrent cameras and monitoring clients.



High-frequency operations should be controlled where necessary.



Potential controls include:



Pagination

Filtering

Controlled WebSocket update frequency

Event batching

Efficient database queries



The API should not become a bottleneck for the AI/CV Engine.



62\. AI/CV Separation



The API Foundation must preserve the AI/CV separation.



The Backend does not directly perform:



Object Detection

Pose Estimation

Tracking

Re-identification

Frame Processing



These belong to the AI/CV Engine.



The Backend consumes their meaningful results through defined interfaces.



63\. Risk Separation



The Backend API does not allow the Frontend to submit an arbitrary final risk score.



Risk is generated by the Backend Risk Engine.



The Frontend receives:



Risk Assessment



rather than submitting the final assessment.



64\. Incident Separation



The Frontend should not directly create a system-generated incident by assigning a cheating label.



Incident creation is controlled by Backend business logic.



Faculty review actions may update incident status where permitted.



65\. Evidence Separation



The API provides controlled access to evidence metadata and authorized evidence resources.



The Frontend must not access arbitrary filesystem paths.



The Backend controls evidence access.



66\. Cross-Camera Separation



Cross-camera identity continuity belongs to the Cross-Camera Tracking architecture.



The API layer exposes resulting track information where required.



The API itself must not implement cross-camera identity matching.



67\. Baseline Separation



Adaptive baseline calculation remains separate from API request handling.



The Backend may consume baseline information for contextual risk processing.



The API layer should not contain baseline calculation logic.



68\. Attention Queue Separation



The Attention Queue is generated by Backend logic.



The API exposes the resulting queue to the Frontend.



The Frontend does not calculate queue priority.



69\. Frontend Integration



Frontend features should consume the API through a centralized API client.



Conceptually:



React Page

&#x20;   ↓

API Client

&#x20;   ↓

HTTP Request

&#x20;   ↓

Backend API



Frontend components should not independently construct incompatible requests.



70\. API Contract Change Process



Before changing an API:



Identify Change

&#x20;     ↓

Update API Contract

&#x20;     ↓

Check Backend Dependencies

&#x20;     ↓

Check Frontend Dependencies

&#x20;     ↓

Check AI/CV Dependencies

&#x20;     ↓

Implement

&#x20;     ↓

Test



Breaking changes require explicit review.



71\. Testing Requirements



Backend API tests should cover:



Authentication

Authorization

Request Validation

Response Validation

Error Handling

Camera API

Exam API

Student API

Seat API

Monitoring API

Event API

Internal Event API

Risk API

Incident API

Evidence API

Analytics API

WebSocket

Health Check

72\. Integration Testing



Integration tests should verify:



Frontend

&#x20;   ↕

Backend API



AI/CV Engine

&#x20;   ↓

Internal Event API

&#x20;   ↓

Backend



Backend

&#x20;   ↕

Database



Backend

&#x20;   ↓

WebSocket

&#x20;   ↓

Frontend

73\. API Contract Testing



API contract tests should verify that:



Request fields match the contract

Response fields match the contract

Event structures match the shared event model

WebSocket messages match documented structures

Error responses remain consistent



This helps prevent Frontend/Backend integration discrepancies.



74\. Architecture Rules



The following rules are mandatory:



Backend owns the application API.

All public APIs use the documented API contract.

API routes must be organized by module.

Requests must be validated.

Responses must use defined schemas.

Errors must use a consistent structure.

Authentication must be enforced by the Backend.

Authorization must be enforced by the Backend.

Frontend must not access the database directly.

AI/CV must not access the database directly.

AI/CV must communicate through the internal event interface.

Internal event structures must follow the shared event contract.

Event IDs must support idempotent processing.

Backend owns risk calculation.

Backend owns incident creation.

Backend owns the attention queue.

Backend owns WebSocket updates.

CPU/GPU-heavy processing belongs to the AI/CV Engine.

Sensitive credentials must not be exposed.

API changes must be reflected in the API contract.

Frontend must not call undocumented endpoints.

AI/CV modules must not invent undocumented Backend interfaces.

Database models must not automatically become public API responses.

API failures must not expose internal implementation details.

API implementation must remain aligned with the established architecture documentation.

75\. Overall API Flow

&#x20;                        ┌──────────────────┐

&#x20;                        │    Frontend      │

&#x20;                        │   React/Vite     │

&#x20;                        └────────┬─────────┘

&#x20;                                 │

&#x20;                             REST API

&#x20;                                 ↓

&#x20;                        ┌──────────────────┐

&#x20;                        │     FastAPI      │

&#x20;                        │   API Layer      │

&#x20;                        └────────┬─────────┘

&#x20;                                 ↓

&#x20;                        ┌──────────────────┐

&#x20;                        │ Request Schemas  │

&#x20;                        │  Validation      │

&#x20;                        └────────┬─────────┘

&#x20;                                 ↓

&#x20;                        ┌──────────────────┐

&#x20;                        │ Domain / Service │

&#x20;                        │     Logic        │

&#x20;                        └────────┬─────────┘

&#x20;                                 ↓

&#x20;                        ┌──────────────────┐

&#x20;                        │   Repository     │

&#x20;                        └────────┬─────────┘

&#x20;                                 ↓

&#x20;                        ┌──────────────────┐

&#x20;                        │    Database      │

&#x20;                        └──────────────────┘





┌──────────────────┐

│   AI/CV Engine   │

└────────┬─────────┘

&#x20;        ↓

&#x20;Event Generation

&#x20;        ↓

&#x20;Internal Event API

&#x20;        ↓

┌──────────────────┐

│     Backend      │

└────────┬─────────┘

&#x20;        ↓

&#x20;Risk / Incident / Storage

&#x20;        ↓

&#x20;WebSocket

&#x20;        ↓

┌──────────────────┐

│    Frontend      │

└──────────────────┘

76\. Responsibility Summary

Component	Responsibility

FastAPI	HTTP API framework

API Router	Endpoint handling

Pydantic	Request/response validation

Domain Layer	Business rules

Repository	Database operations

Authentication	User authentication

Authorization	Permission enforcement

Internal API	AI/CV event reception

Risk Engine	Contextual risk calculation

Incident Module	Incident lifecycle

WebSocket	Real-time updates

Database	Persistent application state

Frontend	API consumer and dashboard

AI/CV Engine	Computer-vision processing

77\. Source of Truth



The Backend API Foundation follows:



API Contract

&#x20;     ↓

FastAPI Routes

&#x20;     ↓

Validation

&#x20;     ↓

Domain Logic

&#x20;     ↓

Repositories

&#x20;     ↓

Database



AI/CV communication follows:



AI/CV Engine

&#x20;     ↓

Event Generation

&#x20;     ↓

Shared Event Contract

&#x20;     ↓

Internal Backend API

&#x20;     ↓

Backend



Frontend communication follows:



Frontend

&#x20;     ↓

Documented REST API

&#x20;     ↓

Backend



Real-time communication follows:



Backend

&#x20;     ↓

WebSocket

&#x20;     ↓

Frontend



No component should bypass these boundaries without an explicit architecture decision.

