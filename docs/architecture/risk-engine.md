# Risk Engine Architecture

This document defines the context-aware risk engine for the AI-Based Offline Examination Monitoring System.

## Purpose

The Risk Engine evaluates multiple events and their context to produce a risk score for an observed student or tracked person.

The system must not directly label a student as cheating based on a single detected event.

## Processing Flow

AI/CV Events
    
Event Collection
    
Context Analysis
    
Rule and Weight Evaluation
    
Risk Score
    
Attention Priority
    
Faculty Dashboard

## Risk Score

The risk engine produces a normalized risk score from 0 to 100.

Risk levels:

- LOW - 0 to 39
- MEDIUM - 40 to 69
- HIGH - 70 to 100

The score represents the priority for invigilator attention and must not be treated as a final determination of cheating.

## Contextual Event Evaluation

The risk engine evaluates combinations of events occurring within a relevant time window.

Example:

HEAD_TURN + REPEATED_INTERACTION + PHONE_DETECTED

This combination produces a higher risk than any one of these events considered independently.

Another example:

A single HEAD_TURN event may produce little or no risk increase.

Repeated HEAD_TURN events combined with LOOKING_NEARBY may increase the risk score.

## Event Weighting

Each supported event may have a configurable weight.

Initial event categories include:

- Looking-related behaviour
- Interaction-related behaviour
- Seat-related behaviour
- Prohibited-object detection
- Unusual activity

Event weights must be configurable rather than hard-coded into the dashboard.

## Repeated Events

Repeated occurrences of the same event may increase risk when they occur within the configured context window.

The system must avoid continuously increasing risk without limits.

Risk scoring must apply configured limits and normalization.

## Context Window

Events must be evaluated using their timestamps and related event identifiers.

The context window allows the engine to determine whether multiple events are related to the same situation.

Events outside the relevant context window should have reduced or no influence on the current risk assessment.

## Student and Track Association

When a tracked person is associated with a student, the risk assessment should be associated with that student.

When student_id is unavailable, the assessment may remain associated with the active track_id until identification is available.

## Risk Assessment

Each risk assessment should contain:

- Assessment identifier
- Student identifier when available
- Track identifier
- Risk score
- Risk level
- Contributing event identifiers
- Assessment timestamp
- Context information

## Attention Queue

Risk assessments are converted into an invigilator attention priority.

Priority levels:

- HIGH - Immediate faculty attention recommended
- MEDIUM - Faculty should review when possible
- LOW - Monitor without immediate intervention

The attention queue should prioritize higher-risk assessments while preserving the supporting event context.

## Explainability

Every risk assessment must retain the events that contributed to the score.

The faculty dashboard should be able to show why an assessment received its risk level.

Example:

Risk Score: 82
Risk Level: HIGH

Contributing Events:

- Repeated looking toward nearby student
- Repeated interaction
- Phone detected

## Ownership

The AI/CV Engine detects and publishes AI/CV events.

The Backend API Service owns the Context-Aware Risk Engine and calculates risk assessments.

The Frontend displays risk assessments and attention priorities but does not calculate risk.

## Configuration

Risk weights, thresholds, context windows, and limits must be configurable.

Configuration changes must not require changes to frontend components.

## Safety Rule

The Risk Engine provides decision support for faculty invigilators.

A risk score must never be presented as definitive proof that a student cheated.
