\# Event Generation Architecture



\## 1. Purpose



The Event Generation module converts meaningful AI/CV observations into standardized events that can be consumed by the Backend.



The module provides the interface between the AI/CV analysis pipeline and the Backend event-processing system.



The Event Generation module does not calculate final risk scores, create final incidents, or directly modify the database.



The Backend remains the system source of truth.



\---



\## 2. Scope



The Event Generation module covers:



\- Converting AI/CV observations into events

\- Event type selection

\- Event identifiers

\- Timestamps

\- Camera context

\- Track context

\- Student context when available

\- Confidence values

\- Event metadata

\- Related event references

\- Event validation before publication

\- Duplicate event prevention

\- Event ordering

\- Event batching where required

\- Event delivery to the Backend

\- Event failure handling

\- Event observability

\- Relationship with detection

\- Relationship with tracking

\- Relationship with behaviour analysis

\- Relationship with prohibited-object detection

\- Relationship with the Backend Risk Engine



\---



\## 3. Position in the Architecture



Event Generation operates after detection, tracking, and behaviour analysis.



Conceptually:



```text

Camera Ingestion

&#x20;     ↓

Frame Processing

&#x20;     ↓

Detection

&#x20;     ↓

Tracking

&#x20;     ↓

Behaviour Analysis

&#x20;     ↓

Event Generation

&#x20;     ↓

Backend Internal Event Interface

&#x20;     ↓

Backend Event Processing

&#x20;     ↓

Risk Engine

&#x20;     ↓

Attention Queue

&#x20;     ↓

Faculty Dashboard

4\. Event Generation Responsibility



The AI/CV Engine is responsible for generating events from observations produced by its processing modules.



Examples include:



PERSON\_DETECTED

PERSON\_MOVED

HEAD\_TURN

LOOKING\_NEARBY

SEAT\_LEFT

PHONE\_DETECTED

BOOK\_DETECTED

REPEATED\_INTERACTION

UNUSUAL\_ACTIVITY

CAMERA\_OFFLINE



Backend-originated events include:



RISK\_UPDATED

INCIDENT\_CREATED



The AI/CV Engine must not generate Backend-owned events such as final risk assessments or incidents.



5\. Shared Event Contract



All generated events must follow the shared event contract defined in:



docs/architecture/event-model.md



The common event structure is:



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



The exact implementation may use typed Python models or another internal representation, but the published event structure must remain compatible with the shared contract.



6\. Event Identifier



Every event must have a unique event\_id.



The identifier allows the Backend to distinguish events from one another.



Conceptually:



Observation

&#x20;   ↓

Event Generation

&#x20;   ↓

Unique Event ID

&#x20;   ↓

Backend



The event ID should not be reused for unrelated observations.



7\. Event Type



Each event must have a valid event type defined by the shared event model.



AI/CV event types include:



PERSON\_DETECTED

PERSON\_MOVED

HEAD\_TURN

LOOKING\_NEARBY

SEAT\_LEFT

PHONE\_DETECTED

BOOK\_DETECTED

REPEATED\_INTERACTION

UNUSUAL\_ACTIVITY

CAMERA\_OFFLINE



Backend event types include:



RISK\_UPDATED

INCIDENT\_CREATED



The AI/CV Engine must not invent event types during runtime without updating the shared architecture and event contract.



8\. Timestamp



Every event must contain the timestamp associated with the observation.



The timestamp should represent when the relevant observation occurred rather than only when the Backend received the event.



Conceptually:



Camera Observation

&#x20;     ↓

Observation Timestamp

&#x20;     ↓

Event

&#x20;     ↓

Backend Receipt Time



This distinction is important when processing delayed frames or network/internal communication delays.



9\. Camera Context



Events generated from camera observations must contain the relevant camera\_id.



Example:



camera\_id = CAM-01



Camera context allows the Backend and faculty dashboard to determine where the event originated.



For multi-camera systems, camera context must never be omitted from camera-derived events.



10\. Track Context



Person-related events should include the relevant track\_id.



Example:



Track 105

&#x20;   ↓

PERSON\_MOVED



The track ID provides continuity between related observations.



Track IDs are temporary computer-vision identities and must not automatically be treated as student IDs.



11\. Student Context



An event may contain student\_id when the student identity is reliably available from examination context.



If the identity is unavailable, the value may remain null.



Conceptually:



track\_id = 105

student\_id = null



is valid when the system cannot reliably associate the track with a student.



The AI/CV Engine must not invent or guess student identities.



12\. Confidence



Events should contain an appropriate confidence value when confidence is available.



Confidence represents the reliability of the underlying AI/CV observation.



For example:



PHONE\_DETECTED

confidence = 0.91



Confidence should not be interpreted as a risk score.



Confidence

&#x20;   ↓

Reliability of observation



Risk Score

&#x20;   ↓

Backend contextual evaluation

13\. Metadata



The metadata field contains additional information required to explain or process the event.



Examples may include:



Bounding box

Position

Movement information

Object class

Observation duration

Number of repeated observations

Nearby track IDs

Seat context

Behaviour state

Detection information



Metadata should contain useful contextual information without unnecessarily exposing personal information.



14\. Related Events



The related\_event\_ids field connects events that contribute to a larger behavioural context.



Example:



HEAD\_TURN

&#x20;   ↓

LOOKING\_NEARBY

&#x20;   ↓

REPEATED\_INTERACTION



The final behavioural event may reference earlier events.



This allows the Backend Risk Engine to understand the context behind an observation.



15\. Detection Event Generation



Detection events may be generated when relevant detection conditions are satisfied.



Examples:



Person detected

&#x20;   ↓

PERSON\_DETECTED



Phone detected

&#x20;   ↓

PHONE\_DETECTED



Book detected

&#x20;   ↓

BOOK\_DETECTED



Detection confidence and relevant object information should be preserved in the event.



The Event Generation module does not perform object detection itself.



16\. Tracking Event Generation



Tracking observations may generate tracking-related events.



Examples:



New person track

&#x20;   ↓

PERSON\_DETECTED



Significant tracked movement

&#x20;   ↓

PERSON\_MOVED



The Tracking module provides the observation.



Event Generation converts the observation into the standardized event representation.



17\. Behaviour Event Generation



Confirmed behavioural observations may generate events such as:



HEAD\_TURN

LOOKING\_NEARBY

SEAT\_LEFT

REPEATED\_INTERACTION

UNUSUAL\_ACTIVITY



Behaviour Analysis determines whether the observation satisfies the configured behavioural conditions.



Event Generation packages the result according to the shared event contract.



18\. Camera Offline Event



Camera connectivity is handled by the Camera Ingestion architecture.



When the camera state indicates that a configured camera is unavailable, the system may generate:



CAMERA\_OFFLINE



The event must retain the affected camera context.



A camera-offline event is an infrastructure/monitoring event and must not be interpreted as suspicious student behaviour.



19\. Event Validation



Before publication, generated events should be validated against the shared event contract.



Validation should check required information such as:



Event ID

Event type

Timestamp

Camera ID where applicable

Track ID where applicable

Confidence where applicable

Metadata structure

Related event references



Invalid events should not be silently published to the Backend.



20\. Event Deduplication



The Event Generation module should prevent uncontrolled duplicate events.



Example:



Frame 100 → PHONE\_DETECTED

Frame 101 → PHONE\_DETECTED

Frame 102 → PHONE\_DETECTED

Frame 103 → PHONE\_DETECTED



The system should avoid generating a separate meaningful alert for every frame when the same observation is continuously present.



Deduplication rules should be configurable according to event type.



The Backend may also perform additional event/incident deduplication.



21\. Event Frequency



Event generation should be controlled according to the meaning of the event.



Some events may be generated once when an observation begins.



Others may be generated when a meaningful state change occurs.



Conceptually:



Observation begins

&#x20;     ↓

Event



Observation continues

&#x20;     ↓

No uncontrolled duplicate events



Observation changes meaningfully

&#x20;     ↓

New event if required



This prevents unnecessary Backend traffic and alert flooding.



22\. Event Ordering



Events should preserve their observation timestamps.



Events may arrive at the Backend slightly out of order because of processing delays.



The Backend should use timestamps and event relationships when determining event context.



The AI/CV Engine should generate events in a predictable processing order whenever possible.



23\. Event Delivery



The AI/CV Engine publishes events to the Backend through the defined internal event interface.



Conceptually:



AI/CV Engine

&#x20;     ↓

Event Generation

&#x20;     ↓

Internal Event Interface

&#x20;     ↓

Backend



The AI/CV Engine must not directly access the Backend database.



24\. Internal REST Boundary



The project architecture recommends an internal REST interface for AI-to-Backend communication.



Conceptually:



AI/CV Engine

&#x20;     ↓

POST /internal/events

&#x20;     ↓

Backend



The exact endpoint implementation must follow the backend API architecture when implemented.



The Event Generation module should depend on the defined internal event interface rather than directly depending on database models.



25\. Event Delivery Reliability



Event delivery should account for temporary failures.



Possible failures include:



Backend unavailable

Internal request failure

Timeout

Temporary processing failure

Invalid response



The AI/CV Engine should avoid silently losing important generated events where reliable delivery is required.



The architecture may use a local outbox or equivalent reliability mechanism as defined by the system implementation.



26\. Local Outbox Boundary



If the internal event delivery mechanism uses an outbox, generated events can first be placed into a local durable queue.



Conceptually:



Event Generated

&#x20;     ↓

Local Outbox

&#x20;     ↓

Backend Available?

&#x20;  ↙        ↘

&#x20;Yes         No

&#x20;↓            ↓

Send       Keep Pending

&#x20;↓

Backend



This prevents temporary Backend availability problems from immediately discarding events.



The exact storage mechanism is an implementation decision.



27\. Event Retry



Failed event delivery may be retried according to configurable retry rules.



The system should avoid unlimited rapid retries.



Potential controls include:



Retry count

Retry delay

Maximum retry duration

Failure logging



Events that cannot be delivered after configured retry limits should be handled according to the system's failure policy.



28\. Event Idempotency



The Backend should be able to recognize the same event if delivery is retried.



The unique event\_id supports this requirement.



Conceptually:



Event ID: E123



First attempt

&#x20;   ↓

Backend receives E123



Retry

&#x20;   ↓

Backend receives E123 again



Backend recognizes existing event



This prevents duplicate persistence caused by retry operations.



29\. Event Context Preservation



Event generation must preserve information required for downstream reasoning.



For example:



LOOKING\_NEARBY

&#x20;   ↓

Track ID

Camera ID

Timestamp

Confidence

Nearby track

Observation duration

Related events



The Backend Risk Engine can then combine this event with other events.



30\. Risk Engine Boundary



Event Generation does not calculate risk.



The flow is:



AI/CV Observation

&#x20;     ↓

Event Generation

&#x20;     ↓

Behaviour/Event

&#x20;     ↓

Backend

&#x20;     ↓

Risk Engine

&#x20;     ↓

Risk Assessment



The Risk Engine may combine multiple events.



Example:



LOOKING\_NEARBY

&#x20;      +

REPEATED\_INTERACTION

&#x20;      +

PHONE\_DETECTED

&#x20;      ↓

Contextual Risk Evaluation



The resulting risk score belongs to the Backend.



31\. Incident Boundary



Event Generation does not create final incidents.



The Backend determines whether events and risk assessments meet the configured incident criteria.



Conceptually:



Event

&#x20; ↓

Risk Assessment

&#x20; ↓

Incident Evaluation

&#x20; ↓

INCIDENT\_CREATED



INCIDENT\_CREATED is therefore a Backend-owned event.



32\. Attention Queue Boundary



Event Generation does not determine invigilator priority.



The flow is:



Event

&#x20;  ↓

Backend Risk Engine

&#x20;  ↓

Risk Assessment

&#x20;  ↓

Attention Queue



The Attention Queue determines the appropriate priority:



HIGH

MEDIUM

LOW

33\. Adaptive Baseline Boundary



Event Generation does not calculate the adaptive baseline.



The baseline may use generated events and behavioural observations as inputs.



Conceptually:



Observations

&#x20;   ↓

Behaviour Analysis

&#x20;   ↓

Events

&#x20;   ↓

Baseline/Risk Context



The baseline remains a separate architecture component.



34\. Cross-Camera Context



Events generated from local camera tracking must preserve camera and track context.



Cross-camera tracking may associate:



CAM-01 / Track-105

&#x20;       ↓

CAM-02 / Track-42



The cross-camera architecture is responsible for determining broader continuity.



Event Generation should not independently implement cross-camera identity matching.



35\. Multi-Camera Event Processing



Multiple cameras may generate events concurrently.



Example:



Camera 1 → Event Generation ─┐

Camera 2 → Event Generation ─┤

Camera 3 → Event Generation ─┤

Camera 4 → Event Generation ─┘

&#x20;                            ↓

&#x20;                         Backend



Every event must retain its originating camera context.



36\. Event Batching



Where required for performance, events may be transmitted in batches.



Conceptually:



Event 1 ─┐

Event 2 ─┤

Event 3 ─┤

Event 4 ─┘

&#x20;   ↓

Batch

&#x20;   ↓

Backend



Batching must not introduce unacceptable latency for high-priority monitoring events.



The final batching strategy is an implementation decision.



37\. Real-Time Requirement



Event generation should operate with sufficiently low latency for real-time examination monitoring.



Important events should reach the Backend quickly enough to support:



Risk evaluation

Attention queue updates

Faculty dashboard updates

Incident review



The system should balance event reliability with processing performance.



38\. Event Logging



The AI/CV Engine should provide operational logs for event-generation failures and important processing states.



Logs may include:



Event creation failures

Validation failures

Delivery failures

Retry attempts

Backend communication failures

Event processing statistics



Logs should not unnecessarily contain sensitive student information.



39\. Security



AI-to-Backend event communication should be restricted to the trusted local system boundary.



The implementation should prevent unauthorized processes from injecting arbitrary examination events.



Event validation and authentication mechanisms belong to the Backend/internal API implementation.



40\. Privacy



Events should contain only the information required for examination monitoring.



Student identity information should only be included when reliably available and necessary.



Sensitive information should not be unnecessarily copied into event metadata.



Evidence media remains separately managed through the Evidence architecture.



41\. Explainability



Generated events should contain enough context to explain why they were produced.



Example:



Event:

REPEATED\_INTERACTION



Context:

Camera: CAM-01

Track A: 105

Track B: 108

Observation window: configured period

Repeated observations: configured count

Confidence: available value

Related events: previous observations



This context supports later review by the Backend and faculty dashboard.



42\. Failure Isolation



A failure while generating or delivering one event should not stop event generation for all cameras.



Conceptually:



Camera 1 → Event failure

Camera 2 → Continue

Camera 3 → Continue

Camera 4 → Continue



The system should isolate recoverable failures wherever practical.



43\. Performance Considerations



Event generation should consider:



Number of cameras

Number of active tracks

Event frequency

Duplicate prevention

Backend communication latency

Retry volume

Memory usage

Batch size

Processing load



The implementation should avoid generating excessive events that do not provide useful information.



44\. Testing Requirements



Event Generation should have tests covering:



Event creation

Event type validation

Required field validation

Unique event IDs

Timestamp handling

Camera context

Track context

Student context

Confidence handling

Metadata

Related events

Duplicate prevention

Retry handling

Idempotency

Backend delivery

Invalid event rejection

Multi-camera event generation

Failure isolation



Integration testing should verify compatibility with:



Detection

Tracking

Behaviour Analysis

Backend internal event interface

Risk Engine

Attention Queue

45\. Architecture Rules



The following rules are mandatory:



Event Generation converts meaningful AI/CV observations into standardized events.

All events must follow the shared event contract.

Every event must have a unique event ID.

Camera-derived events must preserve camera context.

Person-related events should preserve track context.

Student IDs must not be guessed.

Confidence must not be confused with risk score.

Event Generation must not calculate final risk.

Event Generation must not create final incidents.

Event Generation must not directly write to the database.

Backend remains the system source of truth.

Backend-owned events must not be generated by the AI/CV Engine.

Duplicate event generation must be controlled.

Event retries must support idempotent processing.

Related events should preserve contextual relationships where required.

Cross-camera identity matching belongs to the cross-camera tracking architecture.

Event generation must not replace human invigilator judgement.

Event failures must not unnecessarily stop unrelated camera processing.

46\. Overall Event Generation Flow



The complete conceptual flow is:



RTSP/IP Camera

&#x20;     ↓

Camera Ingestion

&#x20;     ↓

Frame Processing

&#x20;     ↓

Object Detection

&#x20;     ↓

Tracking

&#x20;     ↓

Behaviour Analysis

&#x20;     ↓

AI/CV Observation

&#x20;     ↓

Event Generation

&#x20;     ↓

Event Validation

&#x20;     ↓

Duplicate / Frequency Control

&#x20;     ↓

Internal Event Interface

&#x20;     ↓

Backend Event Processing

&#x20;     ↓

Persistent Storage

&#x20;     ↓

Risk Engine

&#x20;     ↓

Attention Queue

&#x20;     ↓

Faculty Dashboard

47\. Responsibility Summary

Component	Responsibility

Camera Ingestion	Acquire camera frames

Detection	Detect people and prohibited objects

Tracking	Maintain person continuity

Behaviour Analysis	Interpret suspicious behaviour

Event Generation	Convert observations into shared events

Cross-Camera Tracking	Maintain continuity across cameras

Backend	Validate, store and coordinate events

Risk Engine	Calculate contextual risk

Attention Queue	Prioritize invigilator attention

Frontend	Display backend state

48\. Source of Truth



The project follows the established ownership model:



AI/CV Engine

&#x20;   ↓

Observe, analyze and generate events



Backend

&#x20;   ↓

Validate, store, evaluate and coordinate



Frontend

&#x20;   ↓

Display backend state



Event Generation is therefore the controlled boundary between AI/CV observations and the Backend event-processing system.
