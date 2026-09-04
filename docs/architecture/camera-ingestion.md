\# CCTV / RTSP Camera Ingestion Architecture



This document defines how the AI-Based Offline Examination Monitoring System connects to classroom CCTV/IP cameras and provides live frames to the AI/CV processing pipeline.



\## Purpose



The Camera Ingestion component is responsible for acquiring live video from supported CCTV/IP camera sources.



The system should:

\- Support RTSP/IP camera streams.

\- Support multiple classroom cameras.

\- Maintain camera connection state.

\- Deliver frames to downstream AI/CV processing.

\- Handle temporary camera failures and reconnection.

\- Keep camera-specific configuration separate from processing logic.



\## Processing Flow



RTSP / IP Camera

&#x20;   ↓

Camera Ingestion

&#x20;   ↓

Frame Acquisition

&#x20;   ↓

Frame Preprocessing

&#x20;   ↓

AI/CV Pipeline



\## Camera Configuration



Each camera should have configuration such as:

\- Camera identifier.

\- Camera name.

\- Room identifier.

\- RTSP endpoint.

\- Enabled/disabled state.

\- Frame processing settings.

\- Reconnection settings.

\- Optional camera position information.



Camera configuration should be stored outside application code.



\## RTSP Support



The system is designed to support standard RTSP camera streams.



The ingestion layer should:

\- Open the configured RTSP stream.

\- Read frames continuously.

\- Detect read failures.

\- Release resources when the camera is stopped.

\- Reconnect when a temporary connection failure occurs.



The exact RTSP transport and backend library may be selected during implementation.



\## Multiple Cameras



Multiple camera streams must operate independently.



Example:



Camera A

&#x20;   ↓

Ingestion A

&#x20;   ↓

AI/CV Processing



Camera B

&#x20;   ↓

Ingestion B

&#x20;   ↓

AI/CV Processing



Camera C

&#x20;   ↓

Ingestion C

&#x20;   ↓

AI/CV Processing



A failure in one camera should not unnecessarily stop unrelated camera streams.



\## Frame Acquisition



The ingestion layer provides frames to downstream processing.



The system should define:

\- Capture frame rate.

\- Processing frame rate.

\- Frame timestamp.

\- Camera identifier.

\- Frame sequence information when required.



The AI/CV pipeline may process fewer frames than the camera's native capture rate when required for performance.



\## Frame Metadata



Each processed frame should retain enough metadata to identify its source.



Relevant metadata may include:

\- Camera identifier.

\- Room identifier.

\- Capture timestamp.

\- Frame sequence number.

\- Processing timestamp.



Frame metadata must remain consistent with the shared event model when events are later generated.



\## Connection State



Each camera should have an observable connection state.



Possible states include:

\- CONNECTING

\- ONLINE

\- DEGRADED

\- OFFLINE

\- RECONNECTING



The exact persistent representation belongs to backend implementation.



Camera state changes should be made available to the backend so the faculty dashboard can display camera availability.



\## Reconnection Strategy



Temporary camera failures should trigger controlled reconnection attempts.



The implementation should:

\- Detect loss of frame acquisition.

\- Release the failed connection.

\- Wait for a configured retry interval.

\- Attempt reconnection.

\- Limit excessive retry activity.

\- Restore normal processing after successful reconnection.



Repeated failures should eventually produce a camera health event.



\## Camera Offline Event



A camera becoming unavailable should generate a camera status event that can be consumed by the backend.



Example:



Camera A

&#x20;   ↓

Frame acquisition failure

&#x20;   ↓

Connection state → OFFLINE

&#x20;   ↓

CAMERA\_OFFLINE event

&#x20;   ↓

Backend

&#x20;   ↓

Faculty Dashboard



Camera recovery should also be represented so the dashboard can reflect the restored state.



\## Performance and Backpressure



Camera ingestion must not allow a slow AI/CV pipeline to cause unlimited frame buffering.



The implementation should use controlled buffering.



When processing falls behind, the system may drop outdated frames rather than allowing unbounded memory growth.



The latest relevant frame should generally be preferred for real-time monitoring.



\## Camera-to-Room Association



Every configured camera should belong to a known examination room.



Example:



Room: Hall A

&#x20;   ├── Camera A

&#x20;   ├── Camera B

&#x20;   └── Camera C



This association is required for:

\- Classroom layout.

\- Student seat mapping.

\- Multi-camera context.

\- Risk evaluation.

\- Faculty monitoring.



\## Camera-to-Processing Association



A camera stream must be associated with the corresponding AI/CV processing pipeline.



The AI/CV Engine receives frames together with camera identity and timing information.



The AI/CV Engine must not directly write camera state into the database.



Backend remains the system source of truth for persisted camera information.



\## Interaction With AI/CV Engine



Camera Ingestion provides video frames.



The AI/CV Engine is responsible for:

\- Detection.

\- Tracking.

\- Behaviour analysis.

\- Prohibited-object detection.

\- Event generation.



Camera ingestion must not contain detection or risk logic.



\## Interaction With Backend



The backend is responsible for persistent camera configuration and system state.



The AI/CV Engine may communicate camera-health information to the backend through the internal event mechanism.



The backend stores camera status and exposes it to the frontend.



\## Interaction With Frontend



The faculty dashboard should receive camera status through the backend.



The frontend may display:

\- Camera online/offline state.

\- Room association.

\- Available live feed.

\- Camera errors.

\- Last known status timestamp.



The frontend should not independently determine camera health.



\## Error Handling



The ingestion layer should handle:

\- Invalid RTSP endpoint.

\- Authentication failure.

\- Network interruption.

\- Camera shutdown.

\- Corrupted frame.

\- Repeated read failure.

\- Processing slowdown.



Errors should be logged with sufficient context to identify the camera involved.



One camera failure should not terminate the complete monitoring system.



\## Logging and Observability



Camera ingestion should provide operational information such as:

\- Connection attempts.

\- Successful connections.

\- Disconnections.

\- Reconnection attempts.

\- Frame acquisition failures.

\- Processing delays.

\- Camera state changes.



Logs should identify the camera involved.



\## Configuration



Camera behaviour should be configurable.



Possible parameters include:

\- RTSP endpoint.

\- Enabled state.

\- Capture settings.

\- Processing frame rate.

\- Reconnection interval.

\- Maximum retry attempts.

\- Buffer size.

\- Timeout settings.



Configuration should remain separate from source code where practical.



\## Security



RTSP credentials must not be hard-coded into source files.



Sensitive camera credentials should be supplied through secure configuration or environment-based mechanisms.



Credentials must not be committed to Git.



\## Resource Management



The ingestion component must release camera resources correctly when:

\- A camera is disabled.

\- The service stops.

\- A stream fails permanently.

\- A reconnection cycle replaces a failed connection.



The system should avoid abandoned camera connections and uncontrolled worker processes.



\## Ownership



Camera Ingestion:

\- Connects to RTSP/IP cameras.

\- Acquires frames.

\- Maintains stream connection state.

\- Handles reconnection.

\- Supplies frames to AI/CV.



AI/CV Engine:

\- Processes acquired frames.

\- Performs detection, tracking, behaviour analysis, and event generation.



Backend:

\- Owns persistent camera configuration and status.

\- Receives camera health events.

\- Provides authoritative camera information to the dashboard.



Frontend:

\- Displays camera feeds and camera health information.

\- Does not manage RTSP connections directly.



\## Safety Rule



Camera ingestion is a video acquisition layer.



It must not independently determine whether a student is cheating, calculate risk scores, or make disciplinary decisions.
