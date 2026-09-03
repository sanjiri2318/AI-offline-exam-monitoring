\# Adaptive Normal-Behaviour Baseline



This document defines the adaptive baseline used by the AI-Based Offline Examination Monitoring System to model normal examination-hall activity and reduce false alerts caused by ordinary student movements.



\## Purpose



The baseline system learns normal activity patterns in an examination environment.



The system should distinguish between:

\- Normal small movements

\- Expected examination activity

\- Significant deviations from normal behaviour



The baseline must support the Risk Engine by providing behavioural context rather than directly declaring suspicious behaviour.



\## Processing Flow



Observed Activity

&#x20;   ↓

Behaviour Measurements

&#x20;   ↓

Baseline Comparison

&#x20;   ↓

Deviation Detection

&#x20;   ↓

AI/CV Event

&#x20;   ↓

Backend Risk Engine



\## Baseline Scope



Version 1 uses a room-level baseline.



The baseline may consider:

\- Movement frequency

\- Movement duration

\- Head movement frequency

\- General activity level

\- Seat occupancy patterns

\- Other supported behavioural measurements



Student-specific baselines may be considered as a future enhancement after the room-level baseline is validated.



\## Calibration



The system requires an initial calibration period before relying heavily on baseline deviations.



During calibration:

\- Normal activity measurements are collected.

\- Initial statistical values are calculated.

\- Temporary unusual activity should not immediately produce strong risk.

\- The baseline should represent the current examination environment.



\## Statistical Model



Version 1 uses a rolling statistical model.



For supported activity measurements, the system may maintain:

\- Rolling mean

\- Rolling standard deviation

\- Recent observation window



A deviation can be calculated by comparing the current measurement with the expected baseline range.



The exact thresholds must remain configurable.



\## Rolling Window



The baseline continuously updates using recent observations.



The rolling window prevents the system from relying permanently on old activity patterns.



Recent normal behaviour can gradually influence the expected baseline while significant deviations can remain visible for risk evaluation.



\## Deviation Detection



A deviation occurs when observed behaviour differs significantly from the current baseline.



Examples:

\- A student briefly adjusts their sitting position → likely normal.

\- A student repeatedly makes unusual movements over a sustained period → potential deviation.

\- Normal head movement during writing → should not automatically create high risk.

\- Repeated unusual activity combined with other events → may contribute to increased risk.



Baseline deviation alone must not be treated as proof of cheating.



\## False-Positive Reduction



The baseline exists partly to reduce false positives.



Small and common movements should have limited influence on the system.



The system should consider:

\- Duration

\- Frequency

\- Repetition

\- Context

\- Other related events



before producing a significant risk contribution.



\## Context Integration



Baseline information is provided as behavioural context to the backend Risk Engine.



The Risk Engine may combine baseline deviation with other events.



Example:



Normal movement

&#x20;   ↓

Low or no risk contribution



Significant movement deviation

&#x20;   +

Repeated interaction

&#x20;   +

Looking toward nearby student

&#x20;   ↓

Higher contextual risk



The baseline does not independently determine the final risk level.



\## Configuration



Baseline configuration should be externalized from application code.



Configurable parameters may include:

\- Calibration duration

\- Rolling window size

\- Deviation thresholds

\- Minimum observation count

\- Update rate

\- Measurement-specific limits



Configuration changes should not require frontend changes.



\## Student and Room Context



Version 1 uses room-level behavioural statistics.



If reliable student-level tracking is available, student-specific baseline information may be associated with the corresponding track or student in a later version.



Student-specific behaviour must not be used to create unfair or permanent assumptions about a student.



\## Ownership



AI/CV Engine:

\- Collects behavioural measurements.

\- Maintains baseline-related processing.

\- Detects baseline deviations.

\- Publishes supported events and metadata.



Backend:

\- Receives baseline-related events.

\- Uses baseline information as context for risk evaluation.

\- Owns the final contextual risk assessment.



Frontend:

\- Displays relevant baseline/deviation information when required.

\- Does not calculate baseline or risk scores.



\## Safety Rule



The adaptive baseline is a decision-support mechanism.



A baseline deviation must never be presented as definitive proof that a student cheated.



The system should preserve the underlying observations and context so that faculty can review the reason for an alert.
