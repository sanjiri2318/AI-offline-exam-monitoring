\# AI Invigilator Attention Queue



This document defines the attention queue used by the AI-Based Offline Examination Monitoring System to prioritize situations that require faculty or invigilator attention.



\## Purpose



The Attention Queue converts contextual risk assessments into an ordered list of situations requiring human attention.



The queue must help invigilators:

\- Focus on higher-priority situations first.

\- Avoid continuously watching every camera equally.

\- Understand why an item was prioritized.

\- Review multiple alerts in an organized manner.



The queue is a decision-support mechanism and must not automatically declare cheating.



\## Processing Flow



Detection Events

&#x20;   ↓

Context-Aware Risk Engine

&#x20;   ↓

Risk Assessment

&#x20;   ↓

Attention Priority

&#x20;   ↓

Attention Queue

&#x20;   ↓

Faculty Dashboard

&#x20;   ↓

Human Review



\## Priority Levels



The queue uses three priority levels:



\### HIGH



Immediate faculty attention is recommended.



Typical causes may include:

\- High contextual risk score.

\- Multiple related suspicious events.

\- Prohibited-object detection combined with other relevant events.

\- Repeated interaction or unusual activity with supporting context.



HIGH priority does not mean cheating has been confirmed.



\### MEDIUM



Faculty review should be performed when possible.



Typical causes may include:

\- Moderate contextual risk.

\- Repeated behavioural deviations.

\- Multiple related events without enough evidence for HIGH priority.



\### LOW



Continue monitoring with limited immediate attention.



Typical causes may include:

\- Low risk score.

\- Minor isolated behavioural deviations.

\- Events that do not form a significant contextual pattern.



\## Queue Ordering



Queue items should be ordered primarily by:

1\. Priority level.

2\. Risk score.

3\. Recency.

4\. Relevant event context.



Higher-priority and higher-risk situations should appear before lower-priority situations.



Older items may remain available for review according to configured retention rules.



\## Attention Queue Item



Each queue item should retain enough information for faculty review.



A queue item may contain:

\- Attention item identifier.

\- Student identifier when available.

\- Track identifier.

\- Room identifier.

\- Camera identifier.

\- Risk assessment identifier.

\- Risk score.

\- Priority level.

\- Timestamp.

\- Contributing event identifiers.

\- Brief explanation of the contributing context.

\- Current review status.



\## Context Preservation



The queue must preserve the context that caused the priority.



Example:



HEAD\_TURN

&#x20;   +

REPEATED\_INTERACTION

&#x20;   +

PHONE\_DETECTED

&#x20;   ↓

HIGH priority



The queue should not display only "HIGH".



It should allow the faculty member to understand the contributing events.



\## Duplicate and Repeated Alerts



Repeated events should not create unlimited duplicate queue items for the same ongoing situation.



The system may group related assessments within a configured context period.



Grouping should preserve the underlying event identifiers and timestamps.



A new queue item may be created when a materially different situation occurs after the previous context has ended.



\## Queue State



An attention item may have states such as:

\- NEW

\- ACKNOWLEDGED

\- UNDER\_REVIEW

\- RESOLVED



The exact persistence model belongs to the backend implementation.



Faculty actions must update the backend state so that all connected dashboard clients receive consistent information.



\## Human Review



The faculty member remains responsible for interpreting the alert.



The queue should provide:

\- What happened.

\- Where it happened.

\- When it happened.

\- Which student or track was involved when known.

\- Which events contributed to the risk.

\- Current risk level and score.



Faculty should be able to review supporting evidence when available.



\## Real-Time Updates



The queue is expected to receive updates when:

\- A new high-priority assessment is created.

\- A medium or low assessment is added.

\- An existing assessment changes priority.

\- An attention item is acknowledged.

\- An attention item is resolved.

\- A relevant risk assessment is updated.



Backend remains the source of truth for queue state.



The frontend receives queue updates through the backend WebSocket layer.



\## Multi-Camera Context



When the same student is associated across multiple camera views, related risk information should remain connected to the same student or cross-camera tracking context when available.



The queue should avoid treating the same ongoing situation as unrelated solely because the camera changed.



\## Interaction With Risk Engine



The Attention Queue does not calculate the risk score.



The Risk Engine:

\- Evaluates contextual events.

\- Produces the risk score.

\- Produces the risk level.

\- Identifies contributing events.



The Attention Queue:

\- Converts the assessment into a faculty priority.

\- Orders attention items.

\- Maintains review state.

\- Presents the context to the dashboard.



\## Configuration



Queue behaviour should be configurable where appropriate.



Configurable parameters may include:

\- Priority thresholds.

\- Queue retention period.

\- Grouping/context window.

\- Maximum visible queue size.

\- Priority ordering rules.



Configuration must remain independent of frontend presentation logic.



\## Ownership



AI/CV Engine:

\- Detects and publishes supported events.



Backend:

\- Owns the Risk Engine.

\- Creates and maintains attention queue items.

\- Manages queue state.

\- Provides real-time queue updates.



Frontend:

\- Displays the queue.

\- Displays priority and context.

\- Allows faculty review actions.

\- Does not calculate risk or independently reorder the authoritative queue.



\## Safety Rule



The AI Invigilator Attention Queue prioritizes situations for human review.



A HIGH priority item is not proof of cheating.



Faculty members must remain the final decision-makers for interpreting incidents and taking action.
