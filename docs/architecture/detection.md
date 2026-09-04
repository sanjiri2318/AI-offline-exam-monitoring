\# Detection Architecture



This document defines the detection architecture for the AI-Based Offline Examination Monitoring System.



\## 1. Purpose



The Detection subsystem processes frames received from the Camera Ingestion subsystem and identifies relevant objects and persons in the examination environment.



The detection subsystem provides structured detection results to the Tracking and Behaviour Analysis components.



Detection must not directly determine whether a student is cheating.



\## 2. Detection Scope



The initial detection scope includes:



\- Person detection

\- Mobile phone detection

\- Book or notes detection

\- Unauthorized visible device detection



The system should be designed so additional object classes can be added later without changing the overall pipeline architecture.



\## 3. Processing Flow



Camera Ingestion

&#x20;   ↓

Frame Selection

&#x20;   ↓

Preprocessing

&#x20;   ↓

Object Detection

&#x20;   ↓

Confidence Filtering

&#x20;   ↓

Detection Results

&#x20;   ↓

Tracking

&#x20;   ↓

Behaviour Analysis

&#x20;   ↓

Event Generation



\## 4. Person Detection



Person detection identifies people visible within the camera frame.



Each detected person should provide:



\- Detection identifier

\- Bounding box

\- Detection confidence

\- Frame timestamp

\- Camera identifier



Person detections are passed to the Tracking subsystem so that observations can be associated with persistent tracks.



The Detection subsystem does not assign a student identity.



\## 5. Prohibited Object Detection



The system detects prohibited or unauthorized objects visible within the examination environment.



\### 5.1 Mobile Phones



The detector identifies visible mobile phones.



A phone detection should include:



\- Object class

\- Bounding box

\- Confidence

\- Timestamp

\- Camera identifier

\- Associated track when available



A single low-confidence detection should not automatically create a high-priority incident.



\### 5.2 Books and Notes



The detector identifies visible books, notebooks, papers, or notes that are configured as prohibited examination objects.



The result should contain the same basic detection metadata used by other object detections.



\### 5.3 Unauthorized Devices



The detection architecture supports additional unauthorized visible devices.



Examples may include:



\- Tablets

\- Laptops

\- Other configured electronic devices



The exact object classes are configuration-driven and should not require changes to the dashboard architecture.



\## 6. Detection Model Boundary



The AI/CV Engine owns model inference.



The Backend API Service does not execute computer-vision inference.



The Frontend does not execute detection inference.



The architecture therefore follows:



AI/CV Engine

&#x20;   ↓

Detection Results

&#x20;   ↓

Shared Event Contract

&#x20;   ↓

Backend API Service



\## 7. Model Abstraction



The detection subsystem should provide an abstraction between the pipeline and the selected detection model.



The rest of the AI/CV pipeline should not depend directly on model-specific implementation details.



This allows the project to evaluate or replace detection models without redesigning the complete system.



The initial implementation may use an object-detection model suitable for person and prohibited-object detection.



The exact model and version are implementation decisions and must be documented when implementation begins.



\## 8. Confidence Handling



Every detection must include a confidence value.



Confidence thresholds should be configurable rather than hard-coded into dashboard logic.



The detection pipeline should distinguish between:



\- Accepted detections

\- Low-confidence detections

\- Rejected detections



Low-confidence observations should not automatically produce suspicious-behaviour events.



Confidence filtering should occur before detection results are passed to downstream processing where appropriate.



\## 9. Frame Processing



The detection subsystem does not necessarily need to process every camera frame.



Frame-processing frequency may be configured according to:



\- Camera frame rate

\- Available CPU/GPU resources

\- Number of active cameras

\- Required monitoring responsiveness



Frame selection must preserve timestamps so that downstream tracking and event processing can maintain temporal context.



\## 10. Multi-Camera Detection



Each detection must retain its originating camera identifier.



This ensures that detections from different CCTV feeds can be associated with the correct camera and examination area.



The detection subsystem does not perform final cross-camera identity association.



Cross-camera continuity is handled by the Cross-Camera Tracking architecture.



\## 11. Detection Result Structure



A detection result should contain, at minimum:



\- Detection identifier

\- Camera identifier

\- Timestamp

\- Object/person class

\- Bounding box

\- Confidence

\- Track identifier when available

\- Additional model metadata when required



Detection results should use consistent field naming across the AI/CV pipeline.



\## 12. Event Generation



Detection results may produce events according to the shared event contract.



Examples include:



\- PERSON\_DETECTED

\- PHONE\_DETECTED

\- BOOK\_DETECTED



Detection alone does not necessarily create a suspicious-behaviour event.



Behaviour-related events require contextual analysis from downstream components.



\## 13. Relationship With Tracking



Detection identifies objects in individual frames.



Tracking associates detections across multiple frames.



The responsibility boundary is:



Detection:

\- What is visible in the current frame?



Tracking:

\- Which observations belong to the same continuing track?



Detection must therefore provide sufficient information for the Tracking subsystem to operate reliably.



\## 14. Relationship With Behaviour Analysis



Behaviour Analysis consumes tracked observations and temporal information to identify patterns such as:



\- Looking toward nearby students

\- Repeated interaction

\- Leaving an assigned seat

\- Unusual movement or activity



The Detection subsystem must not independently classify these behaviours.



Behaviour analysis uses detection and tracking information together with temporal context.



\## 15. False Positive Handling



Detection errors are expected in real CCTV environments.



Potential causes include:



\- Occlusion

\- Poor lighting

\- Camera angle

\- Motion blur

\- Small objects

\- Overlapping students

\- Partial visibility

\- Low image quality



The system should avoid treating a single uncertain detection as definitive evidence of misconduct.



Repeated or corroborated observations may be considered by downstream Behaviour Analysis and the Context-Aware Risk Engine.



\## 16. Occlusion Handling



Students and objects may temporarily become partially or completely occluded.



A temporary loss of detection should not immediately terminate the corresponding track.



Tracking is responsible for maintaining continuity according to its configured tracking strategy.



Detection should resume association when the object becomes visible again.



\## 17. Camera Quality Considerations



Detection quality depends on the quality of incoming CCTV frames.



The system should monitor conditions such as:



\- Resolution

\- Frame rate

\- Connection stability

\- Lighting

\- Motion blur

\- Excessive compression



Camera health information belongs to the Camera Ingestion subsystem, while detection quality may be included as processing metadata.



\## 18. Configuration



Detection-related configuration should be externalized where practical.



Configuration may include:



\- Enabled object classes

\- Confidence thresholds

\- Frame-processing frequency

\- Model configuration

\- Camera-specific detection settings



Configuration changes should not require frontend changes.



\## 19. Evidence Boundary



The Detection subsystem produces structured detection information.



Evidence capture and storage are handled by the Evidence subsystem.



Detection must not directly manage the database or evidence-storage system.



\## 20. Database Boundary



The AI/CV Engine must not directly write detection records to the application database.



The flow is:



AI/CV Detection

&#x20;   ↓

Event Creation

&#x20;   ↓

Backend Internal Event API

&#x20;   ↓

Backend Validation

&#x20;   ↓

Database



The Backend remains the single source of truth.



\## 21. Risk Engine Boundary



Detection does not calculate final risk.



The flow is:



Detection

&#x20;   ↓

Tracking

&#x20;   ↓

Behaviour Analysis

&#x20;   ↓

Events

&#x20;   ↓

Backend Risk Engine

&#x20;   ↓

Risk Assessment

&#x20;   ↓

Attention Queue



A phone detection, for example, may contribute to risk, but the Detection subsystem must not decide the student's final risk level.



\## 22. Explainability



Detection results contributing to a risk assessment should remain traceable through event identifiers.



This allows the dashboard to explain which observations contributed to an attention item.



Example:



Phone detected

&#x20;   +

Repeated interaction

&#x20;   +

Repeated looking

&#x20;   ↓

Higher contextual risk



The final explanation is generated from the events and risk assessment rather than from an opaque detection label.



\## 23. Ownership



\### AI/CV Engine



Responsible for:



\- Frame processing

\- Detection inference

\- Confidence filtering

\- Detection result generation

\- Publishing detection-related events



\### Backend API Service



Responsible for:



\- Event validation

\- Event persistence

\- Risk evaluation

\- Incident creation

\- WebSocket updates



\### Frontend



Responsible for:



\- Displaying detections

\- Displaying alerts

\- Displaying risk and attention information

\- Providing faculty interaction



The Frontend must not calculate detection or risk.



\## 24. Safety Rule



The Detection subsystem identifies visual observations.



A detection must never be presented as definitive proof that a student cheated.



All suspicious-behaviour and risk decisions must remain contextual and subject to faculty review.



\## 25. Future Extension



The architecture may later support additional detection categories without changing the core event and backend architecture.



Possible future classes can include other prohibited objects or examination-specific visual objects.



Such additions must follow the same detection result and event contract defined by the project architecture.
