\# Cross-Camera Student Tracking Architecture



This document defines how the system maintains tracking continuity when the same student appears across multiple CCTV camera views.



\## Purpose



The Cross-Camera Tracking component maintains continuity of a tracked person across different cameras.



The system should:

\- Maintain a consistent identity across camera views when possible.

\- Associate observations from multiple cameras.

\- Support movement continuity between assigned seating areas.

\- Provide cross-camera context to the backend.

\- Avoid treating the same student as unrelated individuals solely because the camera changed.



\## Processing Flow



Camera Feed A

&#x20;   ↓

Person Detection

&#x20;   ↓

Single-Camera Tracking

&#x20;   ↓

Track Representation

&#x20;   ↓

Cross-Camera Matching

&#x20;   ↓

Cross-Camera Track

&#x20;   ↓

Backend Context



Camera Feed B

&#x20;   ↓

Person Detection

&#x20;   ↓

Single-Camera Tracking

&#x20;   ↓

Track Representation

&#x20;   ↓

Cross-Camera Matching

&#x20;   ────────────────┘



\## Single-Camera Tracking



Each camera maintains its own local tracking identifiers.



A local track may contain:

\- Camera identifier

\- Local track identifier

\- Detection timestamps

\- Bounding box information

\- Movement information

\- Appearance representation when available



Local track identifiers are camera-specific and must not be treated as global student identifiers.



\## Cross-Camera Identity



A CrossCameraTrack represents the system's best-known continuity of a person across cameras.



It may associate:

\- Multiple local camera tracks

\- Camera transitions

\- Observation timestamps

\- Student identifier when reliably available



The cross-camera identity must remain distinct from a confirmed student identity.



\## Matching



Cross-camera matching may use available tracking information such as:

\- Appearance features

\- Spatial and temporal consistency

\- Camera layout

\- Expected movement paths

\- Track timing

\- Other validated matching signals



A match should only be accepted when sufficient evidence exists.



Uncertain matches must not automatically be treated as confirmed student identities.



\## Camera Transition



When a person disappears from one camera and appears in another, the system may attempt to associate the new local track with an existing CrossCameraTrack.



Example:



Camera A

Student observation

&#x20;   ↓

Track A-17

&#x20;   ↓

Leaves camera view

&#x20;   ↓

Expected transition interval

&#x20;   ↓

Camera B

&#x20;   ↓

Track B-04

&#x20;   ↓

Cross-camera matching

&#x20;   ↓

Same CrossCameraTrack



The transition must respect configured timing and camera-layout constraints.



\## Student Association



A CrossCameraTrack may be associated with a student when reliable identification is available.



Before confirmation:

\- Track identity remains independent of student identity.

\- Risk assessments may remain associated with the active track.



After reliable association:

\- Related camera observations may be connected to the corresponding student.



The system must avoid assigning a student identity based only on an uncertain cross-camera match.



\## Multi-Camera Context



Cross-camera tracking allows the backend to understand that observations across different cameras may belong to the same ongoing situation.



This is important for:

\- Behaviour continuity

\- Seat movement

\- Suspicious activity context

\- Incident history

\- Risk evaluation



The frontend should be able to receive the relevant student/track context without independently performing identity matching.



\## Interaction With Event Model



AI/CV events must retain their originating camera and track information.



Cross-camera association may connect related events through:

\- Cross-camera track identity

\- Related event identifiers

\- Student identifier when known



The existing shared event contract remains the source of event structure.



Cross-camera tracking must not create a separate incompatible event format.



\## Interaction With Risk Engine



The Cross-Camera Tracking component does not calculate risk.



It provides continuity and context.



The backend Risk Engine may use cross-camera context when evaluating:

\- Repeated interactions

\- Movement between areas

\- Seat leaving

\- Behaviour occurring across multiple camera views

\- Related observations involving the same track/student



The backend remains responsible for the final risk assessment.



\## Uncertainty Handling



Cross-camera identity is probabilistic.



The system should preserve:

\- Matching confidence

\- Source camera

\- Source track

\- Matching timestamp

\- Relevant evidence



Low-confidence matches should not silently become permanent identity assignments.



\## Failure Cases



The system must handle:

\- Person visible in only one camera.

\- Camera temporarily offline.

\- Partial camera overlap.

\- Multiple visually similar students.

\- Occlusion.

\- Poor lighting.

\- Track loss.

\- Simultaneous camera transitions.



When matching is unreliable, the system should preserve separate local tracks rather than forcing an incorrect association.



\## Data Ownership



AI/CV Engine:

\- Maintains camera-local tracking.

\- Produces cross-camera matching information.

\- Provides confidence and tracking context.

\- Publishes relevant tracking events.



Backend:

\- Stores cross-camera tracking state.

\- Associates tracking information with students when appropriate.

\- Uses cross-camera context for risk and incident evaluation.

\- Remains the system source of truth.



Frontend:

\- Displays relevant tracking continuity.

\- Displays camera and student context.

\- Does not independently perform cross-camera identity matching.



\## Privacy and Data Minimization



Cross-camera tracking should retain only the information required for examination monitoring and system operation.



Appearance representations and tracking information should follow the project's configured retention and storage policies.



Tracking information must not be used for unrelated purposes.



\## Configuration



Cross-camera behaviour should be configurable where required.



Configurable parameters may include:

\- Maximum transition interval.

\- Matching confidence threshold.

\- Camera transition relationships.

\- Appearance matching parameters.

\- Track retention duration.

\- Association timeout.



Configuration should remain independent from frontend implementation.



\## Safety Rule



Cross-camera tracking is an identity-continuity mechanism, not definitive proof of student identity.



When confidence is insufficient, the system must preserve uncertainty rather than making an unsupported identity claim.
