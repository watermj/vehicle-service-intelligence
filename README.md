# Vehicle Service Intelligence

**Jason Waterman**

*Context-aware vehicle health intelligence from telemetry to service action | 2026*

---

## Executive Summary

Modern vehicles continuously generate signals about their own health, but turning that telemetry into a trustworthy service decision requires more than detecting an anomaly.

**Vehicle Service Intelligence** explores an end-to-end decision architecture for converting vehicle telemetry into evidence-backed diagnostic and service actions. The project asks a practical question:

> **When can vehicle telemetry be trusted to make a service decision?**

Using production-vehicle telemetry from a 2024 Fisker Ocean, the reference use case — **Parked Energy & Wake Intelligence** — develops a workflow that:

- validates whether vehicle signals are fresh and trustworthy,
- determines the vehicle's expected operating context,
- characterizes known-normal behavior,
- detects behavior inconsistent with that context,
- separates behavioral confidence from root-cause confidence,
- suppresses premature customer or technician notifications,
- identifies what additional diagnostic evidence should be collected, and
- translates accumulated evidence into the appropriate next service action.

The core design principle is:

> **Anomaly ≠ Notification**

A statistically unusual vehicle state is evidence, not a diagnosis. When evidence is insufficient, the system should gather the minimum additional information required to make a better decision rather than prematurely declaring a fault.

The resulting architecture connects:

```text
Vehicle Signals
      ↓
Signal Validity & Operating Context
      ↓
Expected Behavior
      ↓
Behavioral Deviation
      ↓
Confidence Engine
      ↓
Targeted Evidence Collection
      ↓
Service Intelligence
      ↓
Verified Outcome
      ↓
Learning
```

The current implementation uses interpretable statistical and machine-learning methods including PCA, regularized Mahalanobis distance, and Isolation Forest. The objective is not to demonstrate a production-ready predictive-maintenance system, but to demonstrate how **vehicle intelligence can become trustworthy Service Intelligence**.

### Agentic Diagnostic Assistant — implementation framework

A public implementation framework is now available in [`agent/README.md`](./agent/README.md). It defines a Python + LLM API tool-calling architecture for multi-step diagnostic workflows, targeted evidence collection, explicit uncertainty, and human-in-the-loop technician approval.

**Current status:** architecture and implementation scaffold published; working agent prototype in development.

### Relevance to AI-Powered Vehicle Diagnostics

This architecture provides a foundation for AI-powered diagnostic assistants and agentic service workflows.

Before an AI assistant can reason reliably about a fault, it needs trustworthy vehicle state, operating context, diagnostic evidence, and an explicit representation of uncertainty. Vehicle Service Intelligence provides that evidence layer.

A future diagnostic intelligence layer could combine these vehicle-health findings with service documentation, diagnostic procedures, repair history, and technician feedback to support:

```text
Validated Vehicle Evidence
      ↓
Diagnostic Context
      ↓
AI / LLM Diagnostic Assistant
      ↓
Multi-Step Diagnostic Agent
      ↓
Recommended Next Action
      ↓
Technician Review / Execution
      ↓
Verified Repair Outcome
      ↓
Learning
```

In this model, AI does not replace deterministic vehicle diagnostics or technician judgment. It helps **reason across evidence, orchestrate diagnostic steps, identify missing information, and recommend the next-best action**.

The longer-term opportunity is a reusable intelligence layer spanning vehicle firmware, cloud diagnostics, AI-assisted reasoning, and service operations — while maintaining traceability from the original vehicle signal through the eventual repair outcome.

---

## Overview

Modern vehicles generate enormous amounts of telemetry describing vehicle
behavior, component state, network activity, energy use, software state,
and operating conditions.

The challenge is no longer simply collecting vehicle data.

The challenge is determining:

> **When can vehicle telemetry be trusted to make a service decision?**

This project extends the earlier
[Vehicle State of Health using AI/ML](https://github.com/watermj/lv-battery-soh)
project from **vehicle-health inference** toward **Vehicle Service Intelligence**.

The initial reference use case is:

# Parked Energy & Wake Intelligence

> **Can the vehicle recognize whether its energy behavior is appropriate
> for its operating context?**

The case study uses telemetry collected from a **2024 Fisker Ocean
production vehicle** to characterize known-normal parked-energy behavior,
identify behavior inconsistent with an expected Operating Mode, suppress
false positives, and determine what additional evidence should be
collected before a service action is taken.

The objective is not simply better anomaly detection.

The objective is to transform vehicle telemetry into **trustworthy,
context-aware, evidence-backed service decisions**.

---

## Problem Statement

Modern vehicles are **data-rich but insight-poor**.

Vehicle networks expose thousands of CAN, LIN, Ethernet, diagnostic,
sensor, and control signals that reflect vehicle behavior and system
health. Yet traditional vehicle diagnostics still depend heavily on
Diagnostic Trouble Codes (DTCs), fixed thresholds, customer-reported
symptoms, and technician investigation after a problem has become
observable.

The original Vehicle State of Health project explored whether this
existing telemetry could provide richer real-time insight into vehicle
health.

This project asks the harder operational question:

> **How can vehicle intelligence become trustworthy Service Intelligence?**

A vehicle-health system must address several problems before an unusual
signal pattern should become a service event:

- A numerical signal value may not represent a **fresh physical observation**.
- Behavior that is unusual in one Operating Mode may be completely normal
  in another.
- Statistically unusual behavior does not necessarily represent a vehicle
  failure.
- A sensitive anomaly detector can create excessive false positives.
- High confidence that behavior is unusual does not imply high confidence
  about the underlying root cause.
- Insufficient evidence should trigger **additional evidence collection**,
  not necessarily a warning or service visit.

Therefore:

> **Anomaly ≠ Notification**

A useful Service Intelligence system should be sensitive enough to notice
potentially meaningful behavior while maintaining a much higher threshold
for interrupting a customer or technician.

---

## From Vehicle State of Health to Service Intelligence

This project is a direct evolution of:

### [Vehicle State of Health (SoH) using AI/ML](https://github.com/watermj/lv-battery-soh)

The original UC Berkeley AI/ML Professional Certificate capstone used
production-vehicle network telemetry to explore 12V battery State of
Health and broader vehicle-health intelligence.

### Original Question

> **Can vehicle-network telemetry provide richer insight into vehicle health?**

The project explored IBS signals, derived charge-cycle features, broader
vehicle-network features, and multiple machine-learning approaches.

### New Question

> **When can vehicle-network telemetry be trusted to make a service decision?**

This changes the problem substantially.

The objective is no longer simply to produce a better health prediction
or anomaly score.

Vehicle Service Intelligence must understand:

1. **What was measured?**
2. **Is the measurement trustworthy and fresh?**
3. **What was the vehicle expected to be doing?**
4. **Is the observed behavior appropriate for that context?**
5. **Does the deviation persist or recur?**
6. **Do independent signals corroborate the concern?**
7. **Is the available evidence sufficient to identify a likely cause?**
8. **What is the appropriate next action?**
9. **What can be learned from the eventual outcome?**

---

## Jupyter Notebooks

Two Jupyter notebooks form the executable core of this project and should
be run in sequence.

### 1. Generate Full Charge / Discharge Cycle Metrics

[`cs_gen_full-charge-discharge-cycle-metrics.ipynb`](./cs_gen_full-charge-discharge-cycle-metrics.ipynb)

This notebook creates the cycle-level analytical dataset used by the
Vehicle Service Intelligence analysis.

Starting from the cleaned Intelligent Battery Sensor (IBS) vehicle logs,
it identifies LV recharge cycles and characterizes both the recharge event
and, where observable, the preceding discharge interval.

Derived metrics include:

- Recharge duration
- Charge returned (`Ah_in`)
- Mean bulk charging current
- Peak, mean, and tail charging voltage
- State-of-charge behavior
- Apparent preceding discharge (`Ah_out_prev`)
- Discharge duration
- Apparent discharge rate
- Discharge voltage start / end
- Voltage drop and decay rate
- Mean and high-percentile discharge current

The notebook combines these derived metrics with available IBS features
to create the cycle-level feature set used by the primary Service
Intelligence notebook.

An important outcome of this work is the recognition that some apparent
discharge metrics become unreliable during deep vehicle sleep because the
downsampled dataset may carry the last known IBS value forward after fresh
network observations stop.

This notebook therefore supports both **feature generation** and the later
**signal-observability analysis**.

---

### 2. Parked Energy Service Intelligence

[`cs_parked_energy_service_intelligence.ipynb`](./cs_parked_energy_service_intelligence.ipynb)

This is the main analytical notebook for the project.

It uses the generated cycle-level dataset to develop the Parked Energy &
Wake Intelligence case study from exploratory analysis through the
Vehicle Service Intelligence architecture.

The notebook includes:

- Dataset and Operating Mode validation
- Signal-observability analysis
- Physical-consistency testing
- Healthy parked-energy characterization
- Feature Trust Matrix development
- Feature correlation analysis
- Principal Component Analysis (PCA)
- Confirmed quiescent baseline construction
- Ledoit-Wolf regularized covariance modeling
- Mahalanobis behavioral-distance analysis
- Context-mismatch experiments
- Isolation Forest comparison
- Confidence Engine development
- False-positive suppression
- Controlled Service Intelligence scenarios
- Targeted evidence-collection strategy
- Vehicle-to-Service Intelligence architecture
- Project limitations and recommended next steps

The notebook progresses from the question:

> **Is this vehicle behavior unusual?**

to the more operationally useful question:

> **Is the evidence strong enough to justify an action — and if not, what
> evidence should the vehicle collect next?**

---

### Notebook Execution Flow

The notebooks are intended to be used in the following order:

```text
Cleaned vehicle / IBS telemetry
              │
              ▼
cs_gen_full-charge-discharge-cycle-metrics.ipynb
              │
              ▼
Cycle-level charge / discharge behavioral features
              │
              ▼
Operating Mode + confidence metadata
              │
              ▼
cs_parked_energy_service_intelligence.ipynb
              │
              ▼
Signal Validity
              │
              ▼
Expected Behavior
              │
              ▼
Behavioral Deviation
              │
              ▼
Confidence Engine
              │
              ▼
Service Intelligence

---

# Reference Use Case

## Parked Energy & Wake Intelligence

Modern vehicles remain computationally active even while parked.

ECUs may wake for legitimate reasons including scheduled activity,
charging, connectivity, diagnostics, software processes, or user
interaction.

The Service Intelligence question is therefore not:

> **Did the vehicle wake?**

It is:

> **Was the observed energy and wake behavior appropriate for the vehicle's
> expected Operating Mode?**

Potential indicators of abnormal parked-energy behavior include:

- **Unexpected wake frequency**  
  The vehicle wakes significantly more frequently than its normal parked
  pattern.

- **Unexpected recharge demand after sleep**  
  Following a comparable parked interval, substantially more LV energy
  must be replenished than expected.

- **Unexpected awake duration**  
  Vehicle networks or ECUs remain active substantially longer than
  expected while parked.

- **Context mismatch**  
  Energy behavior resembles an active or high-load condition even though
  vehicle context indicates quiescent operation.

Parked Energy & Wake Intelligence serves as the first vertical slice for
developing and testing the broader Vehicle Service Intelligence
architecture.

---

## Dataset

The analysis uses charge/discharge-cycle features derived from vehicle
network data collected during the original Vehicle State of Health
project.

### Vehicle

**2024 Fisker Ocean**

### Current Analysis Population

- **75** detected LV recharge cycles
- **24** source vehicle log files
- **51** cycles with complete preceding discharge intervals
- **14** confirmed quiescent cycles
- **26** confirmed intentional-load cycles
- **1** confirmed HV-charging cycle

### Known-Normal Operating Modes

Three Operating Modes are represented:

- `QUIESCENT`
- `INTENTIONAL_LOAD`
- `HV_CHARGING`

All three represent **known or assumed normal vehicle behavior**.

The dataset does **not** contain a confirmed vehicle failure event.

This distinction is important.

The project therefore performs:

> **Normal-behavior characterization and contextual anomaly detection**

rather than supervised vehicle-failure classification.

---

## Operating-Mode Confidence

Operating Mode is retained as contextual metadata rather than assumed to
be perfect ground truth.

Two confidence levels are used:

### `CONFIRMED`

The operating condition is explicitly supported by the source log or test
description.

### `INFERRED`

The condition is believed to be quiescent based on available test context
but was not independently verified.

The exact electrical loads used during the intentional-load tests were
not independently documented. Therefore, intentional-load observations
are treated as one broad known-normal operating class rather than
attempting to infer the specific load from the resulting electrical
behavior.

---

# Vehicle Service Intelligence Architecture

The proposed architecture separates **detection** from **decisioning**.

```text
VEHICLE SIGNALS
      ↓
SIGNAL VALIDITY / OBSERVABILITY
      ↓
OPERATING CONTEXT
      ↓
EXPECTED BEHAVIOR
      ↓
BEHAVIORAL DEVIATION
      ↓
CONFIDENCE ENGINE
      ↓
EVIDENCE COLLECTION
      ↓
SERVICE INTELLIGENCE
      ↓
VERIFIED OUTCOME
      ↓
LEARNING
      ↺
```

The architecture can be grouped into five functional layers.

## 1. Vehicle Intelligence

Functions include:

- Vehicle-network signal acquisition
- Signal freshness / observability
- Operating Mode determination
- Lightweight feature generation
- Behavioral-event detection
- Event-triggered diagnostic capture

## 2. Behavioral Intelligence

Functions include:

- Vehicle-specific behavioral baselines
- Operating-context baselines
- Cohort / fleet context
- Expected-versus-observed behavior
- Multivariate behavioral-deviation scoring
- Persistence and recurrence analysis

## 3. Confidence Engine

The Confidence Engine determines whether detected behavioral deviation is
sufficiently trustworthy and actionable to justify escalation.

Potential evidence includes:

- Behavioral-deviation magnitude
- Signal quality / observability
- Operating Mode and mode confidence
- Persistence
- Recurrence
- Corroborating vehicle signals
- Vehicle-specific history
- Comparable-vehicle / fleet behavior
- Software or configuration changes

## 4. Service Intelligence

Potential system responses include:

1. **Expected** — no action
2. **Observe / Learn Silently**
3. **Monitor**
4. **Gather Evidence**
5. **Remote / Engineering Action**
6. **Service Action**

## 5. Learning

Verified real-world outcomes can improve future decisions:

- Engineering findings
- Remote / OTA resolution
- Technician diagnosis
- No-fault-found outcome
- Repair performed
- Recurrence after intervention
- Post-repair return to expected behavior

The objective is a closed-loop system in which operational outcomes
continuously improve future decision quality.

---

# Signal Observability

One of the most important findings from the case study was that:

> **Signal availability is not equivalent to signal observability.**

The original vehicle logs were downsampled to a regular time interval.

When an ECU or network stops transmitting during vehicle sleep, the
preprocessing pipeline may carry the **last known signal value forward**.

A numerical value may therefore remain present even though no new physical
measurement has been received.

This is especially important for IBS current during deep vehicle sleep.

Naively integrating a stale current value across a long parked interval
can create physically unrealistic apparent discharge.

Vehicle telemetry should therefore be interpreted as:

```text
Signal Value
+ Measurement Timestamp / Freshness
+ Network State
+ Vehicle Operating Context
```

rather than signal value alone.

### Design Principle

> **Value ≠ Observation**

Signal validity must precede health inference.

---

# Physical-Consistency Validation

The relationship between apparent preceding discharge and subsequent LV
recharge provides a useful physical-consistency check.

Confirmed intentional-load cycles generally show plausible charge balance
because IBS current remains observable while the vehicle is active.

Quiescent cycles show mixed behavior.

Some quiescent intervals produce plausible charge balance, while others
show large apparent discharge with little subsequent recharge.

This is consistent with stale forward-filled IBS current during vehicle
sleep.

The result demonstrates that **Operating Mode and signal observability are
different state variables**.

A vehicle may be legitimately quiescent while the underlying telemetry is
either fresh or stale.

---

# Feature Trust

Feature selection for Vehicle Service Intelligence considers more than
statistical correlation.

Candidate features are evaluated according to:

1. Physical meaning
2. Signal observability
3. Independence from other features
4. Consistency across Operating Modes

The initial trusted recharge representation includes:

| Feature | Behavioral Meaning |
|---|---|
| `Ah_in` | Recharge demand |
| `duration_s` | Recharge timing |
| `I_bulk_mean` | Recharge intensity |
| `V_peak_charge` | Voltage response |

Operating Mode and mode confidence remain contextual metadata rather than
being automatically treated as numerical model inputs.

Several discharge features remain useful for exploration but require
additional signal-freshness validation before they should be trusted
during deep vehicle sleep.

---

# Normal-Behavior Modeling

The dataset does not contain labeled failure examples.

The modeling objective is therefore:

> **Characterize known-normal behavior and determine whether a new
> observation is consistent with the expected contextual baseline.**

Methods evaluated include:

- Robust statistical summaries
- Principal Component Analysis (PCA)
- Regularized Mahalanobis distance
- Isolation Forest

## PCA Behavioral Representation

PCA was applied to four standardized trusted recharge features:

- `Ah_in`
- `duration_s`
- `I_bulk_mean`
- `V_peak_charge`

The first two principal components explain:

### **91.1% of total feature variance**

- **PC1 — 48.3%**
- **PC2 — 42.8%**

This demonstrates that the four-dimensional trusted recharge
representation can be represented compactly for exploratory behavioral
analysis.

Operating Mode was **not** used to construct the PCA components. Mode and
confidence labels were applied afterward for interpretation.

---

# Confirmed Quiescent Baseline

The highest-confidence normal-behavior baseline consists of:

### **14 confirmed quiescent cycles**

These observations come from explicitly documented vehicle-sleep
conditions.

The confirmed quiescent population forms a relatively concentrated
multivariate recharge signature and provides the initial reference
baseline for contextual anomaly detection.

Because the dataset is small, covariance regularization is important.

The project therefore uses a **Ledoit-Wolf regularized covariance
estimate** for the primary multivariate behavioral-distance model.

---

# Context-Mismatch Experiment

A key experiment asks:

> **What happens when real known-normal intentional-load behavior is
> evaluated against a quiescent behavioral baseline?**

The same trusted behavioral features are evaluated using the regularized
Mahalanobis model.

### Median Behavioral Distance

| True Operating Mode | Median Distance from Quiescent Baseline |
|---|---:|
| Confirmed `QUIESCENT` | **0.795** |
| Confirmed `INTENTIONAL_LOAD` | **2.377** |

Known-normal intentional-load behavior is therefore approximately:

# **3× farther**

from the quiescent behavioral baseline at the median.

This demonstrates that the selected recharge features contain meaningful
information about whether observed behavior is consistent with an
expected quiescent state.

However, the populations overlap.

Some legitimate intentional-load events resemble quiescent behavior, and
some confirmed quiescent events receive elevated deviation scores.

Therefore:

> **Behavioral deviation is evidence — not a fault diagnosis.**

A high behavioral-deviation score means that behavior is inconsistent
with the selected contextual baseline.

It does **not** mean that the vehicle has failed.

---

# Model Comparison

Isolation Forest was evaluated as an alternative nonlinear anomaly model.

Its anomaly scores broadly agreed with the regularized Mahalanobis
behavioral-distance model.

The correlation between the two approaches was approximately:

### **r = 0.764**

For the current small dataset, Isolation Forest did not demonstrate
sufficient incremental value to replace the more interpretable
regularized Mahalanobis baseline.

This supports an important project principle:

> **More complex ML does not automatically produce a better Service
> Intelligence decision.**

Interpretability and decision usefulness are prioritized over model
complexity.

---

# Confidence Engine & False-Positive Suppression

A sensitive anomaly detector may be useful internally, but customer and
technician attention should have a substantially higher threshold.

Therefore:

# **Anomaly ≠ Notification**

The Confidence Engine sits between behavioral-deviation detection and
Service Intelligence.

Its purpose is to determine whether available evidence is sufficiently
trustworthy, persistent, corroborated, and actionable to justify
escalation.

## Behavioral Confidence vs. Diagnostic Confidence

These concepts are intentionally separated.

The system may have:

### High Behavioral Confidence

> The observed vehicle behavior is inconsistent with the expected
> baseline.

while simultaneously having:

### Low Diagnostic Confidence

> The available evidence is insufficient to identify the responsible
> subsystem or root cause.

In this condition, the appropriate response is **additional evidence
collection**, not a customer notification or premature diagnosis.

### Design Principle

> **The system should have a low threshold for noticing something and a
> much higher threshold for interrupting a customer or technician.**

---

# Controlled Service-Intelligence Scenario

A real measured intentional-load cycle was used to exercise the decision
architecture.

Measured behavior:

| Metric | Value |
|---|---:|
| Charge returned | **9.489 Ah** |
| Recharge duration | **45.0 min** |
| Mean bulk charging current | **12.639 A** |
| Distance from quiescent baseline | **4.885** |

In its actual context, this observation represents **known-normal
`INTENTIONAL_LOAD` behavior**.

For the controlled decision experiment, the same behavioral signature is
evaluated as if:

- Expected Operating Mode = `QUIESCENT`
- Similar deviation persists across three consecutive cycles

Persistence and expected quiescent context are therefore **controlled
scenario assumptions**, not measured failure conditions.

The resulting evidence state is:

```text
Behavioral Deviation     HIGH
Signal Confidence        HIGH
Context Confidence       HIGH
Persistence              HIGH

Corroboration            LIMITED
Root-Cause Confidence    LOW
```

The resulting Service Intelligence decision is:

# **GATHER EVIDENCE**

**Customer notification:** `NO`

**Next-best action:** `CAPTURE_TARGETED_WAKE_TELEMETRY`

This demonstrates the distinction between detection sensitivity and
action sensitivity.

> **High confidence that something is unusual does not imply high
> confidence about what is wrong.**

---

# Targeted Evidence Collection

Rather than continuously uploading high-resolution vehicle telemetry, the
behavioral model can trigger selective diagnostic capture when additional
evidence is required.

For parked-energy behavior, useful targeted evidence includes:

- Wake timestamp
- Sleep timestamp
- Awake duration
- Wake frequency
- Time between wake events
- Fresh IBS voltage/current
- DC/DC activation
- LV recharge start / end
- Recharge demand
- Network activity
- ECU / bus activity
- Vehicle context
- Charging state
- Software / configuration state

The diagnostic sequence becomes:

```text
Normal Monitoring
      ↓
Behavioral Deviation
      ↓
Persistence
      ↓
Gather Evidence
      ↓
Unexpected Wake
      ↓
Targeted Diagnostic Capture
      ↓
Evidence Analysis
      ↓
Update Diagnostic Confidence
      ↓
Next-Best Action
```

### Diagnostic Principle

> **Collect the minimum additional evidence required to make the next
> decision with greater confidence.**

---

# From Vehicle Intelligence to Service Intelligence

The complete architecture is:

```text
                            VEHICLE
                               │
                               ▼
                      RAW VEHICLE SIGNALS
                               │
                               ▼
                    SIGNAL OBSERVABILITY
                  Fresh? Stale? Trustworthy?
                               │
                               ▼
                      OPERATING CONTEXT
              What is the vehicle expected to do?
                               │
                               ▼
                     TRUSTED FEATURE LAYER
                               │
                               ▼
                    EXPECTED BEHAVIOR MODEL
                   Vehicle / Cohort / Fleet
                               │
                               ▼
                     BEHAVIORAL DEVIATION
                               │
                               ▼
                       CONFIDENCE ENGINE
          Signal Quality • Context • Persistence
                 • Corroboration • History
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ▼              ▼              ▼
             EXPECTED        MONITOR        GATHER
                                           EVIDENCE
                                              │
                                              ▼
                                  TARGETED DIAGNOSTIC
                                         CAPTURE
                                              │
                                              ▼
                                      EVIDENCE ANALYSIS
                                              │
                                              ▼
                                   ROOT-CAUSE CONFIDENCE
                                              │
                       ┌──────────────────────┼──────────────────────┐
                       │                      │                      │
                       ▼                      ▼                      ▼
                    SUPPRESS              REMOTE /               SERVICE
                                         ENGINEERING             ACTION
                                              │                      │
                                              └──────────┬───────────┘
                                                         │
                                                         ▼
                                                 VERIFIED OUTCOME
                                                         │
                                                         ▼
                                                      LEARNING
                                                         │
                                                         └──────────────↺
```

The key architectural principle is that **behavioral-deviation detection
does not directly trigger a customer or service action**.

Instead, detected behavior passes through a Confidence Engine that evaluates
whether the available evidence is sufficiently trustworthy, persistent,
contextually appropriate, and corroborated to justify escalation.

When evidence is insufficient, the system can request targeted diagnostic
telemetry rather than prematurely diagnosing a fault.

This creates a progressive decision process:

> **Observe → Validate → Contextualize → Detect → Corroborate → Decide → Learn**

---

# What This Case Study Demonstrates

This project does **not** demonstrate a production-ready predictive
maintenance system or validated vehicle-failure classifier.

It demonstrates the feasibility of an end-to-end reasoning architecture for
transforming vehicle telemetry into progressively more actionable Service
Intelligence.

The case study demonstrates that:

1. **Telemetry must be validated before it is modeled.**  
   Forward-filled IBS values during vehicle sleep can remain numerically
   present while no longer representing fresh physical observations.

2. **Normal behavior is contextual.**  
   Known-normal quiescent and intentional-load conditions can produce
   substantially different energy signatures.

3. **Normal behavior is multivariate.**  
   Individual feature thresholds can misrepresent physically coherent
   vehicle behavior.

4. **Behavioral deviation can be modeled with interpretable methods.**  
   A regularized multivariate distance provides a practical initial
   quiescent behavioral baseline for the current small dataset.

5. **More complex ML does not automatically improve the solution.**  
   Isolation Forest broadly confirms the same behavioral structure but does
   not demonstrate sufficient incremental value in the current dataset to
   replace the simpler primary model.

6. **Behavioral anomaly is not equivalent to vehicle failure.**  
   Known-normal observations can receive elevated deviation scores when
   evaluated against an inappropriate contextual baseline.

7. **Persistence and corroboration help suppress false positives.**  
   The system can remain sensitive internally while maintaining a higher
   threshold for human interruption.

8. **Uncertainty can drive active diagnostics.**  
   Persistent unexplained behavior can trigger targeted evidence collection
   rather than a premature diagnosis or service notification.

9. **Service Intelligence is a decision problem, not only an ML problem.**  
   The useful output is the next-best action, not merely an anomaly score.

---

# Project Scope & Current Limitations

This project is a **proof-of-concept Vehicle Service Intelligence workflow**
using Parked Energy & Wake Intelligence as the reference use case.

Several limitations are important.

## Dataset Size

The current analysis contains:

- **75** detected LV recharge cycles
- **24** source vehicle log files
- **51** cycles with complete preceding discharge intervals
- **14** confirmed quiescent cycles
- **26** confirmed intentional-load cycles
- **1** confirmed HV-charging cycle

This is sufficient for exploratory analysis and architecture development,
but not for production model calibration.

## No Confirmed Failure Events

The dataset contains known or assumed normal Operating Modes.

It does not contain confirmed examples of:

- Abnormal parasitic battery drain
- Fault-induced network wake behavior
- ECU-induced sleep disruption
- Confirmed 12V battery failure caused by parked-energy behavior

Behavioral deviation therefore cannot currently be interpreted as failure
probability.

## Limited Operating-Mode Metadata

The exact electrical loads used during intentional-load testing were not
independently documented.

Some quiescent Operating Modes are inferred rather than confirmed.

These observations remain useful for exploratory analysis but are not used
as ground-truth confirmed quiescent baseline data.

## Signal Observability

The downsampled dataset may carry the last known signal value forward when
a vehicle network or ECU stops transmitting.

As a result, some apparent discharge-current features cannot be trusted
during deep vehicle sleep without explicit signal-freshness information.

## Limited Vehicle / Fleet Context

The current case study uses telemetry collected from a **2024 Fisker Ocean
production vehicle**.

The analysis does not yet establish how expected behavior varies across:

- Individual vehicles
- Battery age and condition
- Vehicle configuration
- Software version
- Ambient temperature
- Customer usage
- Geographic environment
- Fleet population

## No Verified Service Outcomes

The project does not currently contain technician findings, repair records,
no-fault-found outcomes, or post-repair behavioral validation.

Diagnostic and service-decision performance therefore cannot yet be
validated against real service outcomes.

---

# Next Steps

The next development phase should focus primarily on **better evidence and
validation**, rather than simply adding more complex ML.

## Phase 1 — Prove the Vertical Slice

Develop Parked Energy & Wake Intelligence as a complete
vehicle-to-service decision loop.

Priorities include:

- Add explicit signal freshness / observability
- Define relevant Operating Modes
- Expand confirmed normal-behavior baselines
- Engineer wake-frequency and awake-duration features
- Characterize recharge demand by parked interval
- Detect contextual behavioral deviation
- Trigger targeted telemetry when evidence is insufficient

The desired outcome is:

> **A working vehicle → cloud → service decision loop**

## Phase 2 — Close the Service Loop

Connect behavioral intelligence to real operational outcomes.

Potential feedback includes:

- Engineering findings
- Remote / OTA resolution
- Service diagnosis
- No-fault-found cases
- Repair outcome
- Recurrence after intervention
- Post-repair return to expected behavior

These outcomes can feed back into:

```text
Vehicle Baseline
       ↓
Cohort Baseline
       ↓
Fleet Intelligence
```

The objective is to measure **decision quality — not just model accuracy**.

Relevant measures include:

- False-positive rate
- Detection lead time
- Diagnostic resolution time
- No-fault-found service visits
- Remote-resolution rate
- First-time-fix improvement

## Phase 3 — Scale the Decision Platform

Apply the same architecture to additional vehicle-health and service
domains:

- Charging
- Thermal systems
- Electrical / network systems
- Software behavior
- Intermittent faults
- Component degradation

The long-term objective is a reusable decision architecture capable of
turning increasingly rich vehicle telemetry into increasingly useful
service actions.

> **Start with one measurable service problem. Build the intelligence
> architecture around it. Prove decision quality. Then scale.**

---

# Relationship to the Berkeley Capstone

The two projects are intended to be read together.

## Project 1 — Vehicle State of Health

**Repository:**  
[github.com/watermj/lv-battery-soh](https://github.com/watermj/lv-battery-soh)

**Question:**

> **Can vehicle-network telemetry provide richer insight into vehicle health?**

## Project 2 — Vehicle Service Intelligence

**Repository:**  
[github.com/watermj/vehicle-service-intelligence](https://github.com/watermj/vehicle-service-intelligence)

**Question:**

> **When can vehicle-network telemetry be trusted to make a service decision?**

The evolution can be summarized as:

```text
Vehicle Telemetry
       ↓
Vehicle Health Insight
       ↓
Behavioral Intelligence
       ↓
Decision Confidence
       ↓
Service Intelligence
       ↓
Verified Outcome
       ↓
Learning
```

---

# Final Perspective

The original Vehicle State of Health project asked whether production
vehicle telemetry could provide richer insight into vehicle health.

This project asks the harder operational question:

> **How can vehicle intelligence become trustworthy Service Intelligence?**

The Parked Energy & Wake Intelligence case study suggests that the answer
requires more than an increasingly sophisticated prediction model.

It requires a system that understands:

- What was measured
- Whether the measurement can be trusted
- What the vehicle was expected to be doing
- Whether observed behavior is unusual in that context
- Whether the deviation persists
- What additional evidence is required
- Whether there is sufficient confidence to take action
- What can be learned from the eventual outcome

The resulting architecture can be summarized by four principles:

### Detect locally.

Identify potentially meaningful behavioral changes close to the vehicle.

### Understand globally.

Interpret behavior using vehicle history, operating context, and ultimately
fleet intelligence.

### Act intelligently.

Escalate only when the available evidence supports a useful next action.

### Learn from outcomes.

Use verified diagnostic and service outcomes to improve future decisions.

> **The objective is not a smarter warning light.**
>
> **The objective is a vehicle that knows when something deserves
> attention, what evidence is still missing, and what should happen next.**

---

# Repository Structure

```text
vehicle-service-intelligence/
│
├── README.md
│
├── cs_gen_full-charge-discharge-cycle-metrics.ipynb
└── cs_parked_energy_service_intelligence.ipynb
```

| File | Purpose |
|---|---|
| [`README.md`](./README.md) | Project overview, methodology, findings, architecture, limitations, and next steps |
| [`cs_gen_full-charge-discharge-cycle-metrics.ipynb`](./cs_gen_full-charge-discharge-cycle-metrics.ipynb) | Generates cycle-level recharge and preceding-discharge behavioral features from cleaned IBS telemetry |
| [`cs_parked_energy_service_intelligence.ipynb`](./cs_parked_energy_service_intelligence.ipynb) | Main Vehicle Service Intelligence analysis: contextual behavior, anomaly detection, Confidence Engine, targeted evidence collection, and service-decision architecture |

---

## Contact and Further Information

**Jason Waterman**

Email: [jawaterman@gmail.com](mailto:jawaterman@gmail.com)  
LinkedIn: [www.linkedin.com/in/jasonwaterman](https://www.linkedin.com/in/jasonwaterman)

