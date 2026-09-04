API Contract Architecture

1\. Purpose



The API Contract defines the communication interfaces between the Frontend, Backend, and AI/CV Engine.



It establishes consistent request and response structures so that independently developed modules can integrate without missing endpoints, incorrect parameters, incompatible data structures, or duplicated implementations.



The Backend remains the primary API provider and system source of truth.



2\. Scope



The API Contract covers:



Frontend-to-Backend communication

AI/CV-to-Backend communication

REST API conventions

Internal AI/CV event interface

Authentication

Camera management

Examination management

Student management

Seat management

Monitoring

Events

Incidents

Risk assessments

Evidence

Analytics

WebSocket communication

Request validation

Response structures

Error handling

API versioning

Ownership boundaries

3\. Architecture Boundary



The system contains three major communication boundaries:



Frontend

&#x20;   ↓

Backend REST API

&#x20;   ↓

Backend



and:



AI/CV Engine

&#x20;   ↓

Internal Event Interface

&#x20;   ↓

Backend



and real-time updates:



Backend

&#x20;   ↓

WebSocket

&#x20;   ↓

Frontend



The AI/CV Engine does not communicate directly with the Frontend.



The Frontend does not communicate directly with the AI/CV Engine.



4\. Backend API Ownership



The Backend owns the public application API.



The Backend is responsible for:



Authentication

Authorization

Business logic

Database access

Event persistence

Risk assessment coordination

Incident management

Evidence metadata

Monitoring state

Analytics

WebSocket updates



The Frontend consumes Backend APIs rather than implementing independent business logic.



5\. AI/CV Internal API Ownership



The AI/CV Engine communicates with the Backend through an internal interface.



The primary purpose of this interface is event delivery.



Conceptually:



AI/CV Engine

&#x20;     ↓

POST /internal/events

&#x20;     ↓

Backend



The exact implementation endpoint must be finalized when Backend API implementation begins.



The AI/CV Engine must not access the database directly.



6\. API Design Principles



The APIs should follow these principles:



Clear ownership

Consistent naming

Consistent request structures

Consistent response structures

Explicit validation

Predictable error responses

Authentication where required

Authorization based on user role

No direct database access from Frontend

No direct database access from AI/CV

No duplicated business logic across services

Backend as the source of truth

7\. API Base Structure



The Backend API should use a consistent versioned structure.



Conceptually:



/api/v1/



Functional modules may follow:



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



The exact routes must be finalized before implementation and documented before Frontend integration.



8\. Authentication API



Authentication provides secure access to the faculty monitoring system.



Conceptual operations:



POST /api/v1/auth/login

POST /api/v1/auth/logout

GET  /api/v1/auth/me



Login should return an authenticated session/token according to the selected authentication implementation.



The Backend validates credentials.



The Frontend must not implement authentication logic independently.



9\. Authorization



API access must respect the authenticated user's role.



Potential roles include:



ADMIN

FACULTY



The Backend is responsible for authorization.



The Frontend may hide unavailable functionality for usability, but the Backend remains responsible for enforcing permissions.



10\. Camera API



The Camera API manages configured CCTV/IP cameras.



Conceptual operations:



GET    /api/v1/cameras

GET    /api/v1/cameras/{camera\_id}

POST   /api/v1/cameras

PUT    /api/v1/cameras/{camera\_id}

DELETE /api/v1/cameras/{camera\_id}



Camera information may include:



Camera ID

Camera name

Room

RTSP configuration reference

Status

Resolution

Frame rate

Configuration state



Sensitive camera credentials must not be returned through normal API responses.



11\. Examination API



The Examination API manages examination sessions.



Conceptual operations:



GET  /api/v1/exams

GET  /api/v1/exams/{exam\_id}

POST /api/v1/exams

PUT  /api/v1/exams/{exam\_id}



Exam context may include:



Examination name

Date

Start time

End time

Examination room

Status

12\. Student API



The Student API provides student information required for examination monitoring.



Conceptual operations:



GET /api/v1/students

GET /api/v1/students/{student\_id}



Student information may include:



Student ID

Name

Registration information

Department

Examination assignment



Only information required by the monitoring system should be exposed.



13\. Seat API



The Seat API manages examination seating information.



Conceptual operations:



GET  /api/v1/seats

GET  /api/v1/seats/{seat\_id}

POST /api/v1/seats

PUT  /api/v1/seats/{seat\_id}



Seat information may include:



Seat ID

Room

Position

Assigned student

Camera context where applicable

14\. Monitoring API



The Monitoring API provides the current monitoring state.



Conceptual operations:



GET /api/v1/monitoring/status

GET /api/v1/monitoring/cameras

GET /api/v1/monitoring/students



Monitoring responses may contain:



Camera status

Active examination

Student status

Current tracks

Current risk state

Active incidents

Attention queue summary



The Backend constructs this state from system data.



15\. Event API



Events represent observations generated by the AI/CV Engine.



Conceptual operations:



GET /api/v1/events

GET /api/v1/events/{event\_id}



The Frontend should consume event information from the Backend rather than directly from the AI/CV Engine.



16\. Internal Event API



The AI/CV Engine publishes events to the Backend using the internal event interface.



Conceptually:



POST /internal/events



Request body follows the shared event contract:



{

&#x20;   event\_id,

&#x20;   event\_type,

&#x20;   timestamp,

&#x20;   camera\_id,

&#x20;   track\_id,

&#x20;   student\_id,

&#x20;   confidence,

&#x20;   metadata,

&#x20;   related\_event\_ids

}



The Backend validates the request before processing it.



17\. Event Validation



The Backend must validate internally received events.



Validation includes:



Valid event type

Valid event ID

Timestamp

Camera existence

Track context where applicable

Student context where applicable

Confidence range

Metadata structure

Related event references



Invalid events should return an appropriate error response.



18\. Event Idempotency



The event\_id provides an idempotency mechanism.



If an AI/CV event is retried, the Backend should recognize an already processed event rather than creating an unintended duplicate.



Conceptually:



AI/CV

&#x20; ↓

Event E123

&#x20; ↓

Backend



Retry

&#x20; ↓

Event E123

&#x20; ↓

Backend recognizes E123



This is important for reliable internal event delivery.



19\. Risk API



Risk assessments are generated by the Backend Risk Engine.



Conceptual operations:



GET /api/v1/risk

GET /api/v1/risk/{student\_id}



Risk information may include:



Risk score

Risk level

Timestamp

Student/track context

Contributing events

Explanation/context



The Frontend displays risk information but does not calculate the final risk score.



20\. Incident API



Incidents are created by Backend logic based on events and risk context.



Conceptual operations:



GET   /api/v1/incidents

GET   /api/v1/incidents/{incident\_id}

PATCH /api/v1/incidents/{incident\_id}



Incident information may include:



Incident ID

Student

Camera

Time

Risk level

Related events

Evidence references

Review status



The system must treat incidents as reviewable monitoring records rather than automatic declarations of cheating.



21\. Evidence API



Evidence metadata is managed by the Backend.



Conceptual operations:



GET /api/v1/evidence/{evidence\_id}



Evidence may reference:



Incident

Camera

Timestamp

File path/storage reference

Evidence type

Creation time



Actual evidence media remains in the configured evidence storage.



22\. Analytics API



Analytics provides summarized examination monitoring information.



Conceptual operations:



GET /api/v1/analytics/overview

GET /api/v1/analytics/events

GET /api/v1/analytics/incidents

GET /api/v1/analytics/risk



Analytics may include:



Event counts

Incident counts

Risk distribution

Camera statistics

Examination statistics

Behaviour statistics

23\. WebSocket Architecture



The Backend provides real-time updates to the Frontend.



Conceptually:



AI/CV

&#x20; ↓

Backend

&#x20; ↓

WebSocket

&#x20; ↓

Frontend Dashboard



The Frontend should not continuously poll every monitoring value when a real-time WebSocket update is appropriate.



24\. WebSocket Message Types



The architecture defines the following message categories:



camera\_status

detection\_event

risk\_updated

incident\_created

attention\_queue\_updated

student\_track\_update



Each message should contain a predictable structure.



Conceptually:



{

&#x20;   "type": "risk\_updated",

&#x20;   "timestamp": "...",

&#x20;   "data": { ... }

}

25\. Camera Status Updates



Camera status changes may be sent through WebSocket.



Example:



{

&#x20;   "type": "camera\_status",

&#x20;   "data": {

&#x20;       "camera\_id": "CAM-01",

&#x20;       "status": "ONLINE"

&#x20;   }

}



Possible states should follow the Camera Ingestion architecture.



26\. Detection Event Updates



Relevant detection events may be forwarded to the Frontend through Backend WebSocket updates.



Example:



{

&#x20;   "type": "detection\_event",

&#x20;   "data": {

&#x20;       "event\_id": "...",

&#x20;       "event\_type": "PHONE\_DETECTED",

&#x20;       "camera\_id": "CAM-01",

&#x20;       "track\_id": "105"

&#x20;   }

}



The Backend remains responsible for determining what information is exposed to the Frontend.



27\. Risk Updates



Risk changes may be pushed to the dashboard.



Example:



{

&#x20;   "type": "risk\_updated",

&#x20;   "data": {

&#x20;       "student\_id": "...",

&#x20;       "risk\_score": 78,

&#x20;       "risk\_level": "HIGH"

&#x20;   }

}



The Frontend displays the risk assessment provided by the Backend.



28\. Incident Updates



When the Backend creates an incident, it may publish:



{

&#x20;   "type": "incident\_created",

&#x20;   "data": {

&#x20;       "incident\_id": "...",

&#x20;       "student\_id": "...",

&#x20;       "risk\_level": "HIGH"

&#x20;   }

}



This allows the faculty dashboard to update without manual refresh.



29\. Attention Queue Updates



Changes to the invigilator attention queue may be published through WebSocket.



Example:



{

&#x20;   "type": "attention\_queue\_updated",

&#x20;   "data": {

&#x20;       "queue": \[ ... ]

&#x20;   }

}



The Backend determines queue priority.



The Frontend only displays the resulting queue.



30\. Student Track Updates



Tracking updates may be sent to the dashboard.



Example:



{

&#x20;   "type": "student\_track\_update",

&#x20;   "data": {

&#x20;       "camera\_id": "CAM-01",

&#x20;       "track\_id": "105",

&#x20;       "student\_id": "...",

&#x20;       "position": { ... }

&#x20;   }

}



Track identity must follow the Tracking and Cross-Camera Tracking architectures.



31\. Request Structure



Requests should use predictable structures.



Example:



POST /api/v1/exams

Content-Type: application/json



Request:



{

&#x20;   "name": "Semester Examination",

&#x20;   "date": "...",

&#x20;   "start\_time": "...",

&#x20;   "end\_time": "..."

}



The Backend validates the request before processing.



32\. Response Structure



Successful responses should follow a consistent structure where appropriate.



Conceptually:



{

&#x20;   "success": true,

&#x20;   "data": { ... }

}



Collection responses may include:



{

&#x20;   "success": true,

&#x20;   "data": \[ ... ],

&#x20;   "pagination": { ... }

}



The exact response wrapper must be finalized before implementation and used consistently.



33\. Error Structure



API errors should follow a consistent structure.



Conceptually:



{

&#x20;   "success": false,

&#x20;   "error": {

&#x20;       "code": "INVALID\_REQUEST",

&#x20;       "message": "Request validation failed",

&#x20;       "details": { ... }

&#x20;   }

}



Errors should provide enough information for the Frontend to handle them correctly without exposing internal implementation details.



34\. HTTP Status Codes



The Backend should use appropriate HTTP status codes.



Examples:



200 OK

201 Created

204 No Content

400 Bad Request

401 Unauthorized

403 Forbidden

404 Not Found

409 Conflict

422 Validation Error

500 Internal Server Error



The exact usage should remain consistent across API modules.



35\. Validation Boundary



Validation should occur at the API boundary before business logic executes.



Request

&#x20;  ↓

Authentication

&#x20;  ↓

Authorization

&#x20;  ↓

Validation

&#x20;  ↓

Business Logic

&#x20;  ↓

Database / Processing

&#x20;  ↓

Response



Invalid requests should not reach deeper application layers unnecessarily.



36\. Frontend Integration Boundary



The Frontend should use a centralized API layer.



Conceptually:



React Page

&#x20;   ↓

Frontend API Client

&#x20;   ↓

Backend REST API



Pages and components should not independently construct inconsistent API requests.



The API client should follow the documented API contract.



37\. AI/CV Integration Boundary



The AI/CV Engine should use a dedicated internal client for Backend event delivery.



Conceptually:



AI/CV Module

&#x20;   ↓

Event Generator

&#x20;   ↓

Internal API Client

&#x20;   ↓

Backend



This prevents individual AI/CV modules from implementing different Backend communication methods.



38\. API Contract as Integration Source



The API contract serves as the shared reference for Frontend and Backend development.



Before implementing an API-dependent Frontend feature:



API Contract

&#x20;    ↓

Backend Endpoint

&#x20;    ↓

Frontend API Client

&#x20;    ↓

Frontend Page



The Frontend must not call undocumented endpoints.



The Backend must not silently change a documented contract without updating the contract and dependent implementations.



39\. Contract Change Management



Changes to API contracts must be reviewed before implementation.



A contract change may include:



New endpoint

Removed endpoint

Changed request field

Changed response field

Changed event structure

Changed WebSocket message

Changed authentication requirement

Changed error structure



Dependent Frontend and AI/CV modules must be checked before merging the change.



40\. Backward Compatibility



Existing API consumers should not be broken unnecessarily.



Breaking changes should require an explicit architecture decision.



Where necessary, API versioning may be used:



/api/v1/



followed by a future:



/api/v2/



The project should avoid unnecessary version duplication.



41\. Security Requirements



API communication must enforce:



Authentication

Authorization

Input validation

Secure credential handling

Restricted internal endpoints

Appropriate CORS configuration

Safe error responses



Camera credentials and authentication secrets must never be exposed through normal API responses.



42\. Privacy Requirements



API responses should expose only information required by the consuming component.



The system should avoid unnecessarily exposing:



Sensitive student information

Camera credentials

Internal system configuration

Private filesystem paths

Debug information



Evidence access should follow authorization rules.



43\. Performance Requirements



API design should support multiple cameras and real-time monitoring.



The Backend should avoid unnecessary database queries and excessive WebSocket messages.



High-frequency tracking updates may require controlled update rates.



High-priority risk and incident updates should receive appropriate real-time handling.



44\. Testing Requirements



API tests should cover:



Authentication

Authorization

Request validation

Response structures

Error structures

Camera endpoints

Exam endpoints

Student endpoints

Seat endpoints

Monitoring endpoints

Event endpoints

Risk endpoints

Incident endpoints

Evidence endpoints

Analytics endpoints

Internal event endpoint

WebSocket messages

Duplicate event handling

Invalid requests

Unauthorized access



Integration tests should verify compatibility between:



Frontend ↔ Backend

AI/CV ↔ Backend

Backend ↔ Database

Backend ↔ WebSocket

45\. Architecture Rules



The following rules are mandatory:



Backend is the source of truth for application APIs.

Frontend communicates with the Backend only through defined interfaces.

AI/CV communicates with the Backend through the internal event interface.

AI/CV must not access the database directly.

Frontend must not access the database directly.

Frontend must not communicate directly with the AI/CV Engine.

All API endpoints must be documented before dependent implementation.

Request and response structures must remain consistent.

API changes must be reviewed before implementation.

Authentication and authorization are enforced by the Backend.

Event structures must follow the shared event contract.

Risk calculation belongs to the Backend Risk Engine.

Incident creation belongs to the Backend.

WebSocket updates originate from the Backend.

API errors must use a consistent structure.

Student identity must not be guessed or fabricated.

Sensitive configuration and credentials must not be exposed.

API contracts must not be silently changed to fix Frontend integration issues.

Dependent modules must be checked whenever a contract changes.

The API contract is the integration reference for the entire system.

46\. Overall API Communication Flow

&#x20;                   ┌──────────────────┐

&#x20;                   │   CCTV / RTSP    │

&#x20;                   └────────┬─────────┘

&#x20;                            ↓

&#x20;                   ┌──────────────────┐

&#x20;                   │    AI/CV Engine  │

&#x20;                   └────────┬─────────┘

&#x20;                            ↓

&#x20;                   ┌──────────────────┐

&#x20;                   │ Event Generation│

&#x20;                   └────────┬─────────┘

&#x20;                            ↓

&#x20;                   Internal Event API

&#x20;                            ↓

┌──────────────┐      ┌──────────────────┐

│   Frontend   │ ←──→ │     Backend      │

│ React/Vite   │ REST │ FastAPI          │

└──────────────┘      └────────┬─────────┘

&#x20;        ↑                     ↓

&#x20;        │              ┌──────────────┐

&#x20;        │              │   Database   │

&#x20;        │              └──────────────┘

&#x20;        │

&#x20;     WebSocket

&#x20;        │

&#x20;        └───────────────────────┘

47\. Responsibility Summary

Component	API Responsibility

AI/CV Engine	Generate and publish standardized events

Event Generation	Prepare events according to shared contract

Backend	Own and process application APIs

Risk Engine	Produce contextual risk assessments

Frontend API Client	Consume documented Backend APIs

WebSocket Layer	Deliver real-time Backend updates

Database	Persist Backend-owned system state

48\. Source of Truth



The project follows:



API Contract

&#x20;     ↓

Backend API

&#x20;     ↓

Database / Business Logic

&#x20;     ↓

WebSocket Updates

&#x20;     ↓

Frontend Dashboard



For AI/CV:



AI/CV

&#x20; ↓

Shared Event Contract

&#x20; ↓

Internal Backend API

&#x20; ↓

Backend



No component should bypass these defined boundaries without an explicit architecture decision.
