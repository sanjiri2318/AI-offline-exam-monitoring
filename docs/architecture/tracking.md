\# Tracking Architecture



\## 1. Purpose



The tracking module maintains the identity and continuity of detected people across consecutive video frames.



Detection answers:



> "What objects are visible in this frame?"



Tracking answers:



> "Is this the same person that was detected in previous frames?"



Tracking provides stable track IDs that are used by behaviour analysis, event generation, risk evaluation, and cross-camera tracking.



The tracking module is part of the AI/CV Engine.



The AI/CV Engine performs tracking and publishes tracking-related information to the Backend through the defined internal event interface.



The Backend remains the system source of truth.



\---



\## 2. Scope



The tracking architecture covers:



\- Person tracking within a camera

\- Assignment of temporary track IDs

\- Association between detections and existing tracks

\- Track creation

\- Track updates

\- Track loss handling

\- Track termination

\- Temporary occlusion handling

\- Multiple-person tracking

\- Track metadata

\- Relationship with detection

\- Relationship with behaviour analysis

\- Relationship with event generation

\- Relationship with cross-camera tracking

\- Tracking configuration

\- Tracking confidence

\- Tracking lifecycle

\- AI/CV and Backend ownership



This document does not define the complete cross-camera identity matching implementation.



Cross-camera continuity is defined separately in:



`docs/architecture/cross-camera-tracking.md`



\---



\## 3. Tracking Position in the AI Pipeline



The tracking module operates after object detection.



The processing flow is:



Camera Ingestion

→ Frame Selection

→ Preprocessing

→ Object Detection

→ Confidence Filtering

→ Detection Results

→ Tracking

→ Behaviour Analysis

→ Event Generation

→ Backend



Tracking therefore consumes detection results rather than directly performing object detection.



\---



\## 4. Tracking Input



The tracking module receives detection results from the detection module.



For person tracking, each detection should provide sufficient information for association.



A person detection contains information such as:



\- Detection identifier

\- Object class

\- Bounding box

\- Detection confidence

\- Frame timestamp

\- Camera identifier

\- Frame information



Conceptually:



```text

PersonDetection {

&#x20;   detection\_id

&#x20;   class

&#x20;   bounding\_box

&#x20;   confidence

&#x20;   timestamp

&#x20;   camera\_id

}

