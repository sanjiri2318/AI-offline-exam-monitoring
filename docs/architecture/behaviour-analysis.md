\# Behaviour Analysis Architecture



\## 1. Purpose



The Behaviour Analysis module interprets tracked student observations over time and identifies behaviour patterns relevant to examination monitoring.



Detection identifies objects in individual frames.



Tracking maintains continuity of people across frames.



Behaviour Analysis interprets observations across time and context.



The module is part of the AI/CV Engine.



The module generates behavioural observations/events but does not make final cheating decisions or calculate the final risk score.



The Backend Risk Engine remains responsible for contextual risk evaluation.



\---



\## 2. Scope



The Behaviour Analysis module covers the following project-defined suspicious behaviours:



\- Repeated interaction between students

\- Looking toward nearby students

\- Leaving the assigned seat

\- Unusual movement or activity



The module also handles:



\- Temporal observation windows

\- Track-based behaviour analysis

\- Position history

\- Movement analysis

\- Nearby-student context

\- Seat context

\- Repeated behaviour detection

\- Behaviour confidence

\- Behaviour event generation

\- False-positive reduction

\- Temporary uncertainty

\- Relationship with adaptive baseline

\- Relationship with risk engine

\- Relationship with attention queue



The module does not define prohibited-object detection.



Mobile phones, books/notes, and unauthorized visible devices are detected by the Detection module.



\---



\## 3. Position in the AI/CV Pipeline



Behaviour Analysis operates after tracking.



The conceptual processing flow is:



```text

Camera Ingestion

&#x20;     ↓

Frame Selection

&#x20;     ↓

Preprocessing

&#x20;     ↓

Object Detection

&#x20;     ↓

Confidence Filtering

&#x20;     ↓

Detection Results

&#x20;     ↓

Tracking

&#x20;     ↓

Track History

&#x20;     ↓

Behaviour Analysis

&#x20;     ↓

Behaviour Events

&#x20;     ↓

Backend Internal Event Interface

&#x20;     ↓

Risk Engine

