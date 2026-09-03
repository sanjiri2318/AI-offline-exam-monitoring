# AI-Based Offline Examination Monitoring System Using CCTV

## Project Scope

This document defines the fixed scope for the initial implementation. The project will first complete the following eight requirements before considering additional features.

## 1. Real-Time CCTV Monitoring

- Connect to existing classroom CCTV/IP cameras using RTSP.
- Provide live monitoring of the examination hall.
- Support multiple camera feeds.

## 2. Suspicious Behaviour Detection

- Detect repeated interaction between students.
- Detect looking toward nearby students.
- Detect leaving the assigned seat.
- Detect unusual movements or activity.

## 3. Prohibited Object Detection

- Detect mobile phones.
- Detect books or notes.
- Detect unauthorized visible devices.

## 4. Faculty Monitoring Dashboard

- Display live camera feeds.
- Display classroom layout.
- Display alerts.
- Display student and seat status.
- Display incident history.

## 5. Adaptive Normal-Behaviour Baseline

- Learn the normal activity pattern of the examination hall.
- Avoid flagging every small or normal movement.
- Identify significant deviations from normal behaviour.

## 6. Context-Aware Risk Engine

- Combine multiple events instead of reacting to a single event.
- Consider contextual combinations of detected events.
- Generate a risk score instead of directly labeling an event as cheating.

Example:
Head turn + repeated interaction + phone detection = higher risk.

## 7. AI Invigilator Attention Queue

- Rank incidents as High, Medium, or Low priority.
- Direct the invigilator's attention to the most important areas first.
- Reduce the need to continuously watch every camera.

## 8. Cross-Camera Student Tracking

- Track students when they move between different CCTV camera views.
- Maintain continuity of student tracking across cameras.
- Help identify movement between assigned seating areas.

## Scope Rule

The implementation will prioritize completing and integrating these eight requirements. Additional features will only be considered after the core scope is completed and validated.
