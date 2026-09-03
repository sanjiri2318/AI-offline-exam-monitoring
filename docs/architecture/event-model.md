# Event Model

This document defines the shared event contract between the AI/CV Engine and Backend API Service.

## Event Structure

Each event must contain the following fields:

- `event_id` - Unique identifier for the event.
- `event_type` - Type of event detected or generated.
- `timestamp` - Date and time when the event occurred.
- `camera_id` - Identifier of the camera associated with the event.
- `track_id` - Identifier of the tracked person associated with the event.
- `student_id` - Student identifier when the tracked person is associated with a student; otherwise null.
- `confidence` - Confidence score associated with the detection or behaviour event.
- `metadata` - Additional event-specific information.
- `related_event_ids` - Identifiers of related events that provide context.

## Event Types

### AI/CV Engine Events

- `PERSON_DETECTED`
- `PERSON_MOVED`
- `HEAD_TURN`
- `LOOKING_NEARBY`
- `SEAT_LEFT`
- `PHONE_DETECTED`
- `BOOK_DETECTED`
- `REPEATED_INTERACTION`
- `UNUSUAL_ACTIVITY`
- `CAMERA_OFFLINE`

### Backend Events

- `RISK_UPDATED`
- `INCIDENT_CREATED`

## Ownership

- The AI/CV Engine detects and publishes AI/CV events.
- The Backend API Service receives and stores events.
- The Backend Risk Engine evaluates events and generates risk updates.
- The Backend creates incidents when configured risk conditions are met.
- The Frontend consumes backend-provided event, risk, and incident updates.

## Event Rules

1. Event field names and meanings must remain consistent across services.
2. The AI/CV Engine must not directly write to the database.
3. The AI/CV Engine must not calculate the final contextual risk score.
4. The Backend is the single source of truth for stored events and risk state.
5. Multiple related events may be used together by the Backend Risk Engine for contextual evaluation.
