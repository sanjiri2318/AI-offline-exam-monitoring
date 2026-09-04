Backend Foundation Architecture

1\. Purpose



The Backend Foundation provides the core application service for the AI-Based Offline Examination Monitoring System.



It establishes the base structure required for:



REST APIs

Authentication

Authorization

Database access

Event processing

Risk evaluation

Incident management

Evidence metadata

WebSocket communication

Configuration

Logging

Testing



The Backend is the single source of truth for application state.



2\. Backend Responsibilities



The Backend is responsible for:



Receiving requests from the Frontend

Receiving events from the AI/CV Engine

Validating requests and events

Applying business rules

Managing authentication and authorization

Reading and writing application data

Coordinating the Risk Engine

Managing incidents

Managing evidence metadata

Maintaining monitoring state

Publishing real-time updates

Providing analytics

Recording audit information



The Backend must not perform CPU/GPU-heavy computer-vision processing.



3\. Backend Technology Boundary



The proposed Backend technology is:



Python

&#x20;   ↓

FastAPI

&#x20;   ↓

Uvicorn

&#x20;   ↓

SQLAlchemy

&#x20;   ↓

SQLite / PostgreSQL



Database migrations should use:



Alembic



The exact production database configuration may differ from local development.



4\. Backend Position



The Backend sits between the AI/CV Engine, database, and Frontend.



&#x20;                   ┌──────────────────┐

&#x20;                   │    AI/CV Engine  │

&#x20;                   └────────┬─────────┘

&#x20;                            │

&#x20;                    Internal Events

&#x20;                            ↓

&#x20;                   ┌──────────────────┐

&#x20;                   │     Backend      │

&#x20;                   │     FastAPI      │

&#x20;                   └───────┬───┬──────┘

&#x20;                           │   │

&#x20;                        DB │   │ WebSocket

&#x20;                           ↓   ↓

&#x20;                   ┌──────────┐ ┌──────────────┐

&#x20;                   │ Database │ │   Frontend   │

&#x20;                   └──────────┘ └──────────────┘

5\. Backend Application Structure



The Backend follows a modular structure.



Conceptually:



backend/

├── pyproject.toml

├── alembic/

├── app/

│   ├── main.py

│   ├── api/

│   ├── domain/

│   ├── infrastructure/

│   ├── schemas/

│   ├── core/

│   └── shared\_events/

└── tests/



The structure should keep API handling, business logic, infrastructure, and data schemas separated.



6\. Application Entry Point



The Backend application starts from:



backend/app/main.py



The entry point is responsible for:



Creating the FastAPI application

Registering API routers

Registering middleware

Registering exception handlers

Configuring application startup/shutdown

Registering WebSocket routes where applicable



Heavy AI/CV processing must not run directly inside the API startup process.



7\. API Layer



The API layer handles external requests and responses.



The proposed structure is:



app/api/

├── auth/

├── cameras/

├── exams/

├── students/

├── seats/

├── monitoring/

├── events/

├── incidents/

├── risk/

├── evidence/

├── analytics/

└── internal/



Each module should contain only the API-specific logic required for that module.



8\. Domain Layer



The domain layer contains application business logic.



Proposed areas include:



app/domain/

├── risk\_engine/

├── incidents/

├── students/

└── evidence/



Domain logic should not depend directly on HTTP request handling.



This allows the same business logic to be used by REST APIs, internal event processing, or other Backend processes.



9\. Infrastructure Layer



The infrastructure layer handles technical implementations.



Proposed structure:



app/infrastructure/

├── db/

│   ├── models/

│   └── repositories/

├── storage/

└── websocket/



Responsibilities include:



Database connection

Database models

Repository implementations

Evidence storage

WebSocket connection management

10\. Schema Layer



The schema layer defines structured data exchanged through the Backend.



Conceptually:



Request

&#x20;  ↓

API Schema

&#x20;  ↓

Domain Logic

&#x20;  ↓

Database Model



and:



Database Model

&#x20;  ↓

Domain Logic

&#x20;  ↓

Response Schema

&#x20;  ↓

Frontend



API schemas should not automatically expose internal database models.



11\. Core Layer



The core layer contains shared Backend configuration and foundational functionality.



Potential responsibilities include:



Application settings

Security

Authentication helpers

Logging

Dependency configuration

Common exceptions

Shared utilities



Secrets must be provided through environment/configuration mechanisms rather than hardcoded into source files.



12\. Shared Event Contract



The Backend must follow the event contract defined in:



docs/architecture/event-model.md



The AI/CV Engine publishes standardized events.



The Backend receives and validates them.



The Backend must not create a second incompatible event structure.



13\. Internal Event Processing



The AI/CV Engine communicates with the Backend through the internal event interface.



Conceptually:



AI/CV Engine

&#x20;     ↓

Event Generation

&#x20;     ↓

Internal Event Interface

&#x20;     ↓

Backend

&#x20;     ↓

Validation

&#x20;     ↓

Event Processing



The Backend becomes responsible for persistence and downstream processing.



14\. Database Ownership



The Backend exclusively owns application database access.



AI/CV Engine ──X──> Database

Frontend     ──X──> Database



Backend ──────────> Database



The AI/CV Engine must never directly write monitoring events into the database.



The Frontend must never directly query or modify the database.



15\. Database Layer



The proposed database architecture uses:



SQLAlchemy

&#x20;   ↓

Repository Layer

&#x20;   ↓

Database



The database may use SQLite during development and PostgreSQL for demonstration or production deployment.



The database implementation must follow the documented database schema architecture.



16\. Repository Boundary



Repositories isolate database operations from business logic.



Conceptually:



Domain Logic

&#x20;    ↓

Repository Interface

&#x20;    ↓

Repository Implementation

&#x20;    ↓

SQLAlchemy

&#x20;    ↓

Database



Business logic should not contain scattered raw database queries.



17\. Database Migrations



Database schema changes must be managed through migrations.



The proposed migration tool is:



Alembic



Schema changes should be reproducible across development environments.



Database changes must remain aligned with:



docs/architecture/database-schema.md

18\. Authentication



Authentication is handled by the Backend.



The Backend verifies user credentials and establishes an authenticated session/token according to the selected implementation.



Conceptually:



Frontend

&#x20;   ↓

Login Request

&#x20;   ↓

Backend

&#x20;   ↓

Credential Validation

&#x20;   ↓

Authentication Token/Session



Authentication secrets must never be stored in Frontend source code.



19\. Authorization



Authorization is enforced by the Backend.



The Backend determines whether an authenticated user can access a resource or operation.



Conceptually:



Request

&#x20;  ↓

Authentication

&#x20;  ↓

Authorization

&#x20;  ↓

API Operation



Frontend visibility controls are not a replacement for Backend authorization.



20\. Camera Management



The Backend stores camera configuration and monitoring state.



The Camera Ingestion component remains responsible for actual camera frame acquisition.



Boundary:



Backend

&#x20;   ↓

Camera Configuration



AI/CV Engine

&#x20;   ↓

Camera Connection / Frame Acquisition



The Backend does not directly process camera frames.



21\. Examination Management



The Backend manages examination context.



This may include:



Examination

Examination room

Examination timing

Student assignments

Seat assignments

Camera associations



This context is used by monitoring and risk processing.



22\. Student Management



The Backend maintains student information required by the monitoring system.



The Backend may associate:



Student

&#x20;  ↓

Seat Assignment

&#x20;  ↓

Track

&#x20;  ↓

Event

&#x20;  ↓

Risk Assessment



Student identity must not be guessed by the AI/CV Engine.



23\. Monitoring State



The Backend coordinates the current monitoring state.



This may include:



Active examination

Camera states

Student/seat status

Active tracks

Recent events

Current risk assessments

Active incidents

Attention queue



The Backend provides this information to the Frontend.



24\. Risk Engine Integration



The Risk Engine is part of the Backend.



The processing flow is:



AI/CV Event

&#x20;     ↓

Backend Event Processing

&#x20;     ↓

Risk Engine

&#x20;     ↓

Risk Assessment



The Risk Engine combines contextual information rather than treating a single event as proof of cheating.



25\. Adaptive Baseline Integration



The Backend may consume baseline-related information produced by the monitoring pipeline.



The baseline architecture remains separate from the API foundation.



The Backend should use baseline results as contextual information for risk evaluation.



26\. Incident Management



The Backend owns incident creation and management.



Conceptually:



Events

&#x20;  ↓

Risk Engine

&#x20;  ↓

Incident Evaluation

&#x20;  ↓

Incident



The Backend stores incident information and makes it available to the Frontend.



An incident represents a monitoring record requiring review, not an automatic declaration of cheating.



27\. Evidence Management



The Backend manages evidence metadata.



Actual evidence files are stored separately according to the Evidence Storage architecture.



Conceptually:



Incident

&#x20;  ↓

Evidence Metadata

&#x20;  ↓

Evidence Storage



The Backend controls access to evidence according to authorization rules.



28\. WebSocket Layer



The Backend provides real-time updates to the Frontend.



Conceptually:



Backend

&#x20;  ↓

WebSocket Manager

&#x20;  ↓

Faculty Dashboard



WebSocket updates may include:



camera\_status

detection\_event

risk\_updated

incident\_created

attention\_queue\_updated

student\_track\_update

29\. WebSocket Connection Management



The Backend should maintain active WebSocket connections and distribute relevant updates.



Conceptually:



Faculty Client 1 ─┐

Faculty Client 2 ─┼──> Backend WebSocket Manager

Faculty Client 3 ─┘



A disconnected client should not cause the Backend monitoring process to fail.



30\. Attention Queue Integration



The Backend owns the Attention Queue.



The flow is:



Events

&#x20;  ↓

Risk Engine

&#x20;  ↓

Attention Queue

&#x20;  ↓

WebSocket

&#x20;  ↓

Faculty Dashboard



Priority levels remain:



HIGH

MEDIUM

LOW



The Frontend displays the queue provided by the Backend.



31\. Analytics



The Backend provides monitoring analytics.



Analytics may include:



Event counts

Incident counts

Risk distribution

Camera activity

Behaviour statistics

Examination statistics



Analytics must be generated from Backend-owned data.



32\. Error Handling



The Backend should provide consistent error handling.



Conceptually:



Request

&#x20;  ↓

Validation / Processing

&#x20;  ↓

Success

&#x20;  OR

Error Handler

&#x20;  ↓

Structured Error Response



Errors should not expose internal stack traces, credentials, database information, or other sensitive implementation details.



33\. Logging



The Backend should maintain structured application logs.



Important categories include:



Authentication failures

API errors

Database failures

Internal event failures

WebSocket failures

Risk processing failures

Incident processing failures

Camera status processing

Application startup/shutdown



Logs should avoid unnecessary sensitive student information.



34\. Configuration



Backend configuration should be externalized.



Potential configuration areas include:



Database connection

API host/port

Authentication settings

Internal service settings

Evidence storage location

WebSocket settings

Logging configuration



Environment variables and configuration files should be used instead of hardcoding environment-specific values.



35\. Health Check



The Backend should provide a basic health mechanism for service monitoring.



Conceptually:



GET /health



The health response should indicate whether the Backend service is operating.



Additional dependency checks may be implemented when required.



36\. Startup and Shutdown



The Backend should have controlled startup and shutdown behavior.



Startup may initialize:



Database connection

Application configuration

WebSocket manager

Required service dependencies



Shutdown should release:



Database connections

WebSocket resources

Background resources



The Backend must not terminate unexpectedly because one external component becomes unavailable.



37\. Performance Boundary



The Backend should remain responsive while AI/CV processing is running.



CPU/GPU-heavy tasks such as:



Object detection

Pose processing

Tracking

Re-identification

Frame processing



belong to the AI/CV Engine.



The Backend focuses on coordination, persistence, APIs, and business logic.



38\. Concurrency



The Backend must support concurrent requests from:



Faculty dashboard

AI/CV Engine

Multiple monitoring clients

Background processing



Long-running operations should not block normal API request processing unnecessarily.



39\. Security Boundary



The Backend is a trusted application boundary.



It must enforce:



Authentication

Authorization

Input validation

Internal API protection

Secure configuration

Safe error handling

Evidence access control



Internal endpoints must not be treated as public endpoints.



40\. Privacy Boundary



The Backend should minimize unnecessary student information exposure.



API responses should contain only information required by the consuming feature.



Sensitive information must not be unnecessarily included in:



Logs

WebSocket messages

Event metadata

Error messages

Analytics responses

41\. Testing Structure



Backend tests should be organized under:



backend/tests/



Testing should cover:



API

Authentication

Authorization

Database

Repositories

Event Processing

Risk Engine

Incidents

Evidence

WebSocket

Analytics

Configuration



Integration tests should verify interactions between major Backend modules.



42\. API Contract Alignment



Backend implementation must follow:



docs/architecture/api-contract.md



The API contract is the integration reference for the Frontend and AI/CV Engine.



The Backend must not silently introduce undocumented endpoints or incompatible request/response structures.



43\. Architecture Documentation Alignment



The Backend must remain aligned with:



docs/architecture/

├── event-model.md

├── database-schema.md

├── risk-engine.md

├── baseline-architecture.md

├── attention-queue.md

├── cross-camera-tracking.md

├── camera-ingestion.md

├── detection.md

├── tracking.md

├── behaviour-analysis.md

├── event-generation.md

└── api-contract.md



If implementation requires a change to an established architecture decision, the corresponding documentation must be updated through the project workflow.



44\. Architecture Rules



The following rules are mandatory:



Backend is the system source of truth.

Backend owns database access.

Frontend must not access the database directly.

AI/CV must not access the database directly.

Backend owns application APIs.

AI/CV communicates through the defined internal event interface.

Frontend communicates through documented Backend APIs.

Backend owns final risk evaluation.

Backend owns incident creation.

Backend owns the attention queue.

WebSocket updates originate from the Backend.

CPU/GPU-heavy CV processing belongs to the AI/CV Engine.

API contracts must remain aligned with api-contract.md.

Event structures must remain aligned with event-model.md.

Database implementation must remain aligned with database-schema.md.

Authentication and authorization must be enforced by the Backend.

Sensitive information must not be unnecessarily exposed.

Architecture changes must be documented and reviewed.

Backend failures must be handled without unnecessarily stopping unrelated monitoring operations.

Backend implementation must not invent undocumented APIs or bypass established architecture boundaries.

45\. Overall Backend Flow

&#x20;                    ┌──────────────────┐

&#x20;                    │    AI/CV Engine  │

&#x20;                    └────────┬─────────┘

&#x20;                             │

&#x20;                      Events │

&#x20;                             ↓

&#x20;                    ┌──────────────────┐

&#x20;                    │     Backend      │

&#x20;                    │                  │

&#x20;                    │  API Layer       │

&#x20;                    │  Domain Layer    │

&#x20;                    │  Event Processor │

&#x20;                    │  Risk Engine     │

&#x20;                    │  Incident Logic  │

&#x20;                    │  WebSocket       │

&#x20;                    └──────┬─────┬─────┘

&#x20;                           │     │

&#x20;                         DB│     │WebSocket

&#x20;                           ↓     ↓

&#x20;                    ┌────────┐ ┌──────────┐

&#x20;                    │Database│ │ Frontend │

&#x20;                    └────────┘ └──────────┘

46\. Responsibility Summary

Component	Responsibility

FastAPI	Backend HTTP API

API Layer	Request/response handling

Domain Layer	Business logic

Infrastructure	Database, storage, WebSocket

Event Processor	Receive and process AI/CV events

Risk Engine	Contextual risk scoring

Incident Module	Incident lifecycle

Database	Persistent application state

WebSocket	Real-time dashboard updates

Frontend	Dashboard presentation

AI/CV Engine	Computer-vision processing

47\. Source of Truth



The Backend follows this ownership model:



AI/CV Engine

&#x20;   ↓

Observe + Analyze + Generate Events

&#x20;   ↓

Backend

&#x20;   ↓

Validate + Store + Evaluate + Coordinate

&#x20;   ↓

Database

&#x20;   ↓

Persistent State



Backend

&#x20;   ↓

WebSocket / REST

&#x20;   ↓

Frontend



No Frontend or AI/CV component should bypass the Backend ownership boundary without an explicit architecture decision.
